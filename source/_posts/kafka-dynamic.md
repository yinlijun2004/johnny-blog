---
title: Kafka动态操作初探
date: 2020-11-15 10:43:06
tags: [Spring Boot, Kafka]
---
我的需求是创建一个款基于点对点实时游戏的服务器，后期连接服务器可能会做成集群，用于连接的可能是不是同一台服务器，因此需要一个消息中间件来做转发。

消息不需要持久化，连接实时创建，实时销毁。

现在用消息中间件的原因，单纯的是没用过，想尝鲜，后期可能会改成netty来实现。

Kafka是一款优秀的给予发布订阅的消息中间件，利用注解可以很方便完成发布订阅消息功能。
今天利用Kafka完成动态创建topic，动态修改订阅topic列表的功能。

### Kafka安装

Kafaka依赖zookeeper，为了避免直接在主机上安装，可以用docker-compose创建一个Kafka镜像，并运行。
```yml
version: '2'
services:
  zookeeper:
    image: wurstmeister/zookeeper
    volumes:
      - ./data:/data
    ports:
      - "2181:2181"
       
  kafka:
    image: wurstmeister/kafka
    ports:
      - "9092:9092"
    environment:
      KAFKA_ADVERTISED_HOST_NAME: 127.0.0.1 
      KAFKA_MESSAGE_MAX_BYTES: 2000
      KAFKA_AUTO_CREATE_TOPICS_ENABLE: 'false'
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    volumes:
      - ./kafka-logs:/kafka
      - /var/run/docker.sock:/var/run/docker.sock
 
  kafka-manager:
    image: sheepkiller/kafka-manager
    ports:
      - 9020:9000
    environment:
      ZK_HOSTS: zookeeper:2181

```
KAFKA_AUTO_CREATE_TOPICS_ENABLE，这个设置为false，不自动创建topic，避免topic数量爆炸。
KAFKA_ADVERTISED_HOST_NAME，这个如果想要多broker，就不能设为localhost。

执行命令启动
```
docker-compose up -d
```
此时可能会报错
```bash
found character '\t' that cannot start any token
  in "./docker-compose.yml", line 17, column 1
```
那是因为yml中有无效的字符（比如tab被空格代替），如果是vim，可以打开如下开关，然后再打开yml文件查看字符。
```
set list
```

### 修改项目pom.xml
```xml
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
```

### application.yml配置
```
  kafka:
    # 指定kafka server的地址，集群配多个，中间，逗号隔开
    bootstrap-servers: 127.0.0.1:9092
    # 生产者
    producer:
      # 写入失败时，重试次数。当leader节点失效，一个repli节点会替代成为leader节点，此时可能出现写入失败，
      # 当retris为0时，produce不会重复。retirs重发，此时repli节点完全成为leader节点，不会产生消息丢失。
      retries: 0
      # 每次批量发送消息的数量,produce积累到一定数据，一次发送
      batch-size: 16384
      # produce积累数据一次发送，缓存大小达到buffer.memory就发送数据
      buffer-memory: 33554432
      # 指定消息key和消息体的编解码方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: com.sevenblock.xxxx.ProtobufSerializer
      properties:
        linger.ms: 1
    # 消费者
    consumer:
      enable-auto-commit: false
      auto-commit-interval: 100ms
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.apache.kafka.common.serialization.ByteBufferDeserializer
      properties:
        session.timeout.ms: 15000
      group-id: group
```
consumer的enable-auto-commit，设置为false，我们记录位置自动提交。
consumer的value-deserializer为ByteBufferDeserializer，因为我们要传输的protobuf数据，可以改成你想要的数据。
同理producer的串行化工具也要针对protobuf订制。

### 增加configure。
要动态的创建topic,需要一个AdminClient实例，以及一个consumer实例。
```java
@Configuration
public class KafkaConfig {
    @Value("${spring.kafka.bootstrap-servers}")
    private List<String> kafkaStringURL;

    @Autowired
    private ConsumerFactory consumerFactory;

    @Bean
    public AdminClient createAdminClient() {
        String clientID = "vibrator-" + RandomStringUtils.randomAlphanumeric(20);
        Properties properties = new Properties();
        properties.put("bootstrap.servers", kafkaStringURL);
        properties.put("client.id", clientID);
        return AdminClient.create(properties);
    }

    @Bean
    public KafkaConsumer<String, Object> createConsumer() {
        return (KafkaConsumer<String, Object>)consumerFactory.createConsumer();
    }
}
```
### 定义KafkaService接口
```java
public interface KafkaService {
    Result createTopic(String topic);

    Result deleteKafkaTopic(String topic);

    void sendAsync(String topic, Object data);

    void subscribe(String topic);

    void unsubscribe(String topic);
}
```
### 实现类
```java
@Slf4j
@Service(value = "topicService")
public class KafkaServiceImpl implements KafkaService, ApplicationRunner {
    @Autowired
    private KafkaTemplate kafkaTemplate;

    @Autowired
    private WSUserManager userManager;

    @Autowired
    private KafkaConsumer kafkaConsumer;

    @Autowired
    private AdminClient adminClient;

    private AtomicBoolean topicChanges = new AtomicBoolean(false);

    private List<String> topics = Collections.synchronizedList(new ArrayList<>());

    @Override
    public Result createTopic(String topicName) {
        // Must be a valid topic name and valid URL
        Result result = new Result();

        CreateTopicsResult topicResults = null;
        boolean createdTopic = true;

        try {
            List<NewTopic> topicsToAdd = new ArrayList<>();
            NewTopic newTopic = null;
            newTopic = new NewTopic(topicName, 1, (short) 1);
            topicsToAdd.add(newTopic);
            topicResults = adminClient.createTopics(topicsToAdd);
            //topicResults.all().get(30, TimeUnit.SECONDS);
            Void test = topicResults.values().get(topicName).get();
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            e.printStackTrace();
            result.setKafkaErrorMessage(e.getLocalizedMessage());
            createdTopic = false;
        } catch (TopicExistsException e) {
            createdTopic = false;
        } catch (Exception e) {
            e.printStackTrace();
            result.setKafkaErrorMessage(e.getLocalizedMessage());
            createdTopic = false;
        }
        if (createdTopic) {
            result.setKafkaTopicMessage("Created topic.");
        } else {
            result.setKafkaTopicMessage("Could not create topic.");
        }
        return result;
    }

    @Override
    public Result deleteKafkaTopic(String kafkaTopic) {
        Result result = new Result();
        unsubscribe(kafkaTopic);

        try {
            Collection<String> topics = new ArrayList<>();
            topics.add(kafkaTopic);
            final DeleteTopicsResult deleteTopicsResult = adminClient.deleteTopics(topics);

            try {
                deleteTopicsResult.all().get();
                result.setTopicName(kafkaTopic);
            } catch (InterruptedException | ExecutionException e) {
                e.printStackTrace();
                result.setKafkaErrorMessage(e.getLocalizedMessage());
            }
        } catch (Exception e) {
            e.printStackTrace();
            result.setKafkaErrorMessage(e.getLocalizedMessage());
        }
        return result;
    }

    @Override
    public void sendAsync(final String topic,final Object message) {
        ListenableFuture<SendResult<String, GameProtoMsg.Msg>> future = kafkaTemplate.send(topic,message);
        future.addCallback(new ListenableFutureCallback<SendResult<String, GameProtoMsg.Msg>>() {
            @Override
            public void onSuccess(SendResult<String, GameProtoMsg.Msg> result) {
                //System.out.println("发送消息成功：" + result);
            }

            @Override
            public void onFailure(Throwable ex) {
                System.out.println("发送消息失败："+ ex.getMessage());
            }
        });
        kafkaTemplate.flush();
    }

    @Override
    public void unsubscribe(String topic) {
        topics.remove(topic);
        topicChanges.set(true);
    }

    @Override
    public void subscribe(String topic) {
        topics.add(topic);
        topicChanges.set(true);
    }

    private void reSubscribe() {
        kafkaConsumer.subscribe(topics);
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {
        while (true) {
            if(topics.size() == 0) {
                ThreadUtil.sleep(50);
                continue;
            }
            if(topicChanges.get()) {
                reSubscribe();
                topicChanges.set(false);
            }
            ConsumerRecords<String, ByteBuffer> records = kafkaConsumer.poll(Duration.ofMillis(100));
            long timeStamp = System.currentTimeMillis();
            for (ConsumerRecord<String, ByteBuffer> record : records) {
                //消息处理
            }
            kafkaConsumer.commitAsync();
            long duration = System.currentTimeMillis() - timeStamp;
            if(records.count() > 0) {
                log.error(" process records {} duration:{}ms",
                        records.count(), duration);
            }
        }
    }
}

```
topicChanges的引入的作用时，kafkaConsumer不能再多个线程里操作，否则报如下错误：
```
KafkaConsumer is not safe for multi-threaded access
```

### nodejs测试程序
忽略发送时的protobuf打包过程。
```javascript
let WebSocket = require( 'ws' );
let GameMessage =  require("./GameMsg");

let type = process.argv[2];
var ws;
if(type === 'c') {
    ws = new WebSocket('ws://localhost:8080/web_ws?channelId=123');
} else {
    ws = new WebSocket('ws://localhost:8080/app_ws?channelId=123');
}

function sendMessage() {
  let id = Math.random() * 3 + 1;
  let emoji = GameMessage.game_msg.Emoji.create({emojiId:id});
  let msg = GameMessage.game_msg.Msg.create({cmd:GameMessage.game_msg.CMD.EMOJI, emoji:emoji});
  ws.send(GameMessage.game_msg.Msg.encode(msg).finish());
}

ws.onopen = function(){
    console.log("connect success");
    setInterval(sendMessage, 10);
}
ws.onerror = function(e) {
	console.error("链接出错",e);
}
ws.onclose = function() {
	console.error("链接closed");
}
let i = 0;
ws.onmessage = function(evt){
  	let message = evt.data
    let msg = GameMessage.game_msg.Msg.decode(message);
    switch(msg.cmd) {
        case GameMessage.game_msg.CMD.EMOJI:
            console.log("receive emoji", msg.emoji.emojiId);
            break;
        case GameMessage.game_msg.CMD.LOCATION:
            console.log("receive location", msg.location.latitude, msg.location.longitude);
            break;  
    }
}
```
### 总结

这是一个demo，还有TopicPartitioin没有处理，最不能接受的是，实时性不强，收消息时偶尔会一卡一卡，不知道是不是BUG，但是现在没有精力查了……所以本文所示代码仅供参考，谨慎用于生产环境。下一章节用netty来实现此功能，

### 参考资料

https://blog.csdn.net/goose_flesh/article/details/86502096

https://blog.csdn.net/hitzhang/article/details/5298049

https://blog.csdn.net/Crystalqy/article/details/94006936

https://hub.docker.com/r/wurstmeister/kafka

https://www.cnblogs.com/rickiyang/p/11074193.html

https://github.com/tspannhw/kafkaadmin-processor.git