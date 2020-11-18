---
title: Netty实现Websocket点对点通信
date: 2020-11-17 15:36:14
tags: [Spring Boot, Websocket, Netty]
---

Netty是优秀的异步网络通信框架，采用的是异步事件驱动模型，就是非阻塞式的线程模型，这样可以节省大量的线程资源，今天基于netty搭建一个点对点通信的demo。

### pom.xml
```xml
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.50.Final</version>
        </dependency>
```
### 主服务类
```java
@Slf4j
@Component
public class NettyBootstrapRunner implements ApplicationRunner,
        ApplicationListener<ContextClosedEvent>, ApplicationContextAware {


    @Value("${netty.websocket.port}")
    private int port;

    @Value("${netty.websocket.ip}")
    private String ip;

    @Value("${netty.websocket.path}")
    private String path;

    @Value("${netty.websocket.max-frame-size}")
    private long maxFrameSize;

    private ApplicationContext applicationContext;

    private Channel serverChannel;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    @Override
    public void run(ApplicationArguments args) throws Exception {

        EventLoopGroup bossGroup = new NioEventLoopGroup();
        EventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap serverBootstrap = new ServerBootstrap();
            serverBootstrap.group(bossGroup, workerGroup);
            serverBootstrap.channel(NioServerSocketChannel.class);
            serverBootstrap.localAddress(new InetSocketAddress(this.ip, this.port));
            serverBootstrap.childHandler(new ChannelInitializer<SocketChannel>() {
                @Override
                protected void initChannel(SocketChannel socketChannel) throws Exception {
                    ChannelPipeline pipeline = socketChannel.pipeline();
                    pipeline.addLast(new HttpServerCodec());
                    pipeline.addLast(new ChunkedWriteHandler());
                    pipeline.addLast(new HttpObjectAggregator(65536));
                    pipeline.addLast(new ChannelInboundHandlerAdapter() {
                        @Override
                        public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
                            if(msg instanceof FullHttpRequest) {
                                AttributeKey<Map<String,String>> requestParam = AttributeKey.valueOf("request.params");

                                FullHttpRequest request = (FullHttpRequest) msg;
                                //http request uri: /chat?accesskey=hello
                                String uri    = request.uri();
                                String [] splittedUri = uri.split("\\?");

                                HashMap<String, String> params = new HashMap<String, String>();
                                request.setUri(splittedUri[0]);
                                if(splittedUri.length > 1){
                                    String queryString = splittedUri[1];
                                    for(String param : queryString.split("&")){
                                        String [] keyValue = param.split("=");
                                        if(keyValue.length >= 2){
                                            log.info("uri = {} key = {}, value = {}", splittedUri[0], keyValue[0], keyValue[1]);
                                            params.put(keyValue[0], keyValue[1]);
                                        }
                                    }
                                }
                                ctx.channel().attr(requestParam).set(params);
                            }
                            super.channelRead(ctx, msg);
                        }
                    });
                    pipeline.addLast(new WebSocketServerCompressionHandler());
                    pipeline.addLast(new WebSocketServerProtocolHandler(path));
                    pipeline.addLast(applicationContext.getBean(WebsocketMessageHandler.class));
                }
            });
            Channel channel = serverBootstrap.bind().sync().channel();
            this.serverChannel = channel;
            log.info("websocket 服务启动，ip={},port={}", this.ip, this.port);
            //这样代码会卡住所以不会退出
            channel.closeFuture().sync();
        } finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    @Override
    public void onApplicationEvent(ContextClosedEvent event) {
        if (this.serverChannel != null) {
            this.serverChannel.close();
        }
        log.info("websocket 服务停止");
    }
}
```
里面一个重要的对象是ChannelPipeline，这是请求处理的管线，用到了设计模式里面的职责链模式。

管线上挂了很多个handler,每个handler对请求轮流进行处理，按照类型分为：ChannelInboundHandler,ChannelOutboundHandler.

ChannelInboundHandler负责上报事件。从接口也能看出这一点。
```java
public interface ChannelInboundHandler extends ChannelHandler {
    void channelRegistered(ChannelHandlerContext var1) throws Exception;

    void channelUnregistered(ChannelHandlerContext var1) throws Exception;

    void channelActive(ChannelHandlerContext var1) throws Exception;

    void channelInactive(ChannelHandlerContext var1) throws Exception;

    void channelRead(ChannelHandlerContext var1, Object var2) throws Exception;

    void channelReadComplete(ChannelHandlerContext var1) throws Exception;

    void userEventTriggered(ChannelHandlerContext var1, Object var2) throws Exception;

    void channelWritabilityChanged(ChannelHandlerContext var1) throws Exception;

    void exceptionCaught(ChannelHandlerContext var1, Throwable var2) throws Exception;
}
```

ChannelOutboundHandler负责请求IO操作。从接口也能看出这一点：
```java
public interface ChannelOutboundHandler extends ChannelHandler {
    void bind(ChannelHandlerContext var1, SocketAddress var2, ChannelPromise var3) throws Exception;

    void connect(ChannelHandlerContext var1, SocketAddress var2, SocketAddress var3, ChannelPromise var4) throws Exception;

    void disconnect(ChannelHandlerContext var1, ChannelPromise var2) throws Exception;

    void close(ChannelHandlerContext var1, ChannelPromise var2) throws Exception;

    void deregister(ChannelHandlerContext var1, ChannelPromise var2) throws Exception;

    void read(ChannelHandlerContext var1) throws Exception;

    void write(ChannelHandlerContext var1, Object var2, ChannelPromise var3) throws Exception;

    void flush(ChannelHandlerContext var1) throws Exception;
}
```
有点像Android RIL层里面的指令，一种是Request（下发请求），比如说拨打电话，一种是Unsolicited（主动上报），比如说信号强度变化。

#### ChannelOutboundHandler里的read方法，ChannelInboundHandler里的channelRead方法，区别是啥？

查看ChannelOutboundHandler里的read注释，read方法发起读请求，读到的数据，从一个ChannelInboundHandler开始传递channelRead时间，这个是符合设计理念的。inbound负责事件传递，outbound负责io请求。
```
/**
    * Request to Read data from the {@link Channel} into the first inbound buffer, triggers an
    * {@link ChannelInboundHandler#channelRead(ChannelHandlerContext, Object)} event if data was
    * read, and triggers a
    * {@link ChannelInboundHandler#channelReadComplete(ChannelHandlerContext) channelReadComplete} event so the
    * handler can decide to continue reading.  If there's a pending read operation already, this method does nothing.
    * <p>
    * This will result in having the
    * {@link ChannelOutboundHandler#read(ChannelHandlerContext)}
    * method called of the next {@link ChannelOutboundHandler} contained in the {@link ChannelPipeline} of the
    * {@link Channel}.
    */
```

管线的模型如下：
```
  
                                                  I/O Request
                                             via {@link Channel} or
                                         {@link ChannelHandlerContext}
                                                       |
   +---------------------------------------------------+---------------+
   |                           ChannelPipeline         |               |
   |                                                  \|/              |
   |    +---------------------+            +-----------+----------+    |
   |    | Inbound Handler  N  |            | Outbound Handler  1  |    |
   |    +----------+----------+            +-----------+----------+    |
   |              /|\                                  |               |
   |               |                                  \|/              |
   |    +----------+----------+            +-----------+----------+    |
   |    | Inbound Handler N-1 |            | Outbound Handler  2  |    |
   |    +----------+----------+            +-----------+----------+    |
   |              /|\                                  .               |
   |               .                                   .               |
   | ChannelHandlerContext.fireIN_EVT() ChannelHandlerContext.OUT_EVT()|
   |        [ method call]                       [method call]         |
   |               .                                   .               |
   |               .                                  \|/              |
   |    +----------+----------+            +-----------+----------+    |
   |    | Inbound Handler  2  |            | Outbound Handler M-1 |    |
   |    +----------+----------+            +-----------+----------+    |
   |              /|\                                  |               |
   |               |                                  \|/              |
   |    +----------+----------+            +-----------+----------+    |
   |    | Inbound Handler  1  |            | Outbound Handler  M  |    |
   |    +----------+----------+            +-----------+----------+    |
   |              /|\                                  |               |
   +---------------+-----------------------------------+---------------+
                   |                                  \|/
   +---------------+-----------------------------------+---------------+
   |               |                                   |               |
   |       [ Socket.read() ]                    [ Socket.write() ]     |
   |                                                                   |
   |  Netty Internal I/O Threads (Transport Implementation)            |
   +-------------------------------------------------------------------+
 ```
初看这个图，有点像koa里面的洋葱模型，又有点像spring里面的过滤器。

每个inbound的事件都是从下往上传播（有点像web里面的事件冒泡），IO的写请求从上往下传播，这个顺序就是往管线里面注册handler的顺序。

以一个channelActive，writeAndFlush为例，浏览管线（DefaultChannelPipeline）中的源码，

可以看到channelActive事件，是从头部传播的。
```java
@Override
public final ChannelPipeline fireChannelActive() {
    AbstractChannelHandlerContext.invokeChannelActive(head);
    return this;
}
```

而writeAndFlush，是从尾部开始传播的。
```java
@Override
public final ChannelFuture writeAndFlush(Object msg) {
    return tail.writeAndFlush(msg);
}
```

查看我们的demo，依次加入了如下handler.
```
HttpServerCodec  //处理http请求响应编解码的
ChunkedWriteHandler //处理批量写入的
HttpObjectAggregator  //Http请求聚合器
ChannelInboundHandlerAdapter //处理query参数
WebSocketServerCompressionHandler //
WebSocketServerProtocolHandler //处理websocket消息
WebsocketMessageHandler //处理主业务
```
#### handler、 channel、 pipeline(管线) 的关系

channel关联一个socket链接，或者一个组件，这个组件有I/O操作的能力。Channel类的注释：
```java
 A nexus to a network socket or a component which is capable of I/O
 operations such as read, write, connect, and bind.
```

pippline是channel用来传递事件、执行操作的。Channel类的注释：
```java
the {@link ChannelPipeline} which handles all I/O events and requests associated with the channel.</li> 
```
handler是处理IO事件，执行IO操作。ChannelHandler的注释：
```java
Handles an I/O event or intercepts an I/O operation, and forwards it to its next handler in its {@link ChannelPipeline}.
```

handler是pipeline用来具体执行上述事件传递，执行操作的，即pipeline只是组织者，handler是执行者。

举个不恰当的例子：channel相当于食客，pipeline相当于服务员，handler相当于帮厨、主厨、传菜员。
channel进店，点个鱼香肉丝。pipeline组织帮厨切料，主厨炒菜，传菜员上菜（真流水线），handler就是干活的。handler说鱼香肉丝没有鱼惹，pipeline告诉channel，channel说好吧，我不吃了，于是走了，就断开连接了。


### 主业务Handler类
```java
@Slf4j
@ChannelHandler.Sharable
@Component
public class WebsocketMessageHandler extends SimpleChannelInboundHandler<BinaryWebSocketFrame> {
    @Autowired
    private GameMessageService gameMessageService;

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, BinaryWebSocketFrame msg) throws Exception {
            this.gameMessageService.onGameMessage(ctx, msg);
    }

    @Override
    public void handlerAdded(ChannelHandlerContext ctx) throws Exception {
        System.out.println(ctx.channel().remoteAddress()+"建立连接!");
    }

    @Override
    public void handlerRemoved(ChannelHandlerContext ctx) throws Exception {
        log.info(ctx.channel().remoteAddress()+"断开连接");

        AttributeKey<Map<String,String>> requestParam = AttributeKey.valueOf("request.params");
        Map<String,String> param = ctx.channel().attr(requestParam).get();
        String channelId = param.get("channelId");
        String user = param.get("user");
        gameMessageService.removeChannel(user, channelId);
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
    }
}
```
收到一帧数据后，转交到服务类中处理。
```java
@Slf4j
@Service
public class GameMessageServiceImpl implements GameMessageService {
    private static Map<String, Channel> appChannel = new ConcurrentHashMap<>();
    private static Map<String, Channel> webChannel = new ConcurrentHashMap<>();

    @Override
    public void onGameMessage(ChannelHandlerContext ctx, BinaryWebSocketFrame frame) {
        byte[] bytes = null;
        if(frame.content().hasArray()) {
            bytes = frame.content().array();
        } else {
            bytes = new byte[frame.content().readableBytes()];
            frame.content().getBytes(0, bytes);
        }
        ByteBuf buf = Unpooled.buffer(bytes.length);
        buf.writeBytes(bytes);

        AttributeKey<Map<String,String>> requestParam = AttributeKey.valueOf("request.params");
        Map<String,String> param = ctx.channel().attr(requestParam).get();
        String channelId = param.get("channelId");
        String user = param.get("user");

        switch (user) {
            case "app":
                appChannel.putIfAbsent(channelId, ctx.channel());
                Channel web = webChannel.getOrDefault(channelId,null);
                if(web != null) {
                    web.writeAndFlush(new BinaryWebSocketFrame(buf));
                }
                break;
            case "web":
                webChannel.putIfAbsent(channelId, ctx.channel());
                Channel app = appChannel.getOrDefault(channelId,null);
                if(app != null) {
                    app.writeAndFlush(new BinaryWebSocketFrame(buf));
                }
                break;
        }
    }

    @Override
    public void removeChannel(String user, String channelId) {
        switch (user) {
            case "app":
                appChannel.remove(channelId);
                break;
            case "web":
                webChannel.remove(channelId);
                break;
        }
    }
}
```
根据channelId，找到对方，然后转发消息，链接断开的时候，也根据参数移除响应的channel.

### 测试程序
```javascript
let WebSocket = require( 'ws' );
let GameMessage =  require("./GameMsg");

console.log(process.argv);
let type = process.argv[2];
let channelId = process.argv[3];
var ws;
let isClient = false;
if(type === 'c') {
  	isClient = true;
    ws = new WebSocket(`ws://localhost:1024/channel?user=web&channelId=${channelId}`);
} else {
    ws = new WebSocket(`ws://localhost:1024/channel?user=app&channelId=${channelId}`);
}

function sendMessage() {
  let id = isClient ? 2 : 3;
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
ws.onclose = function(e) {
		console.error("链接closed",e);

}
ws.onmessage = function(evt){
  	let message = evt.data
    let msg = GameMessage.game_msg.Msg.decode(message);
		console.log("收到消息", msg.cmd, " ", i++);
    switch(msg.cmd) {
        case GameMessage.game_msg.CMD.EMOJI:
            console.log("receive emoji", msg.emoji.emojiId);
            break;
    }
}
```


### More

为什么netty一个服务实例，不能支持多websocket多端点（Multiple EndPoints）。

我有两个业务模块，就想逻辑分开，想用两个端点，每个端点处理各自的业务逻辑，所以想在管线里面添加两个Handler，让两个Handler的连接都能得到处理的。

```java
pipeline.addLast(new WebSocketServerProtocolHandler("/app_ws"));
pipeline.addLast(new WebSocketServerProtocolHandler("/web_ws"));
```

但是跑起来发现，”/web_ws”是不能处理的，tcp可以建立连接，但是websocket的handeshake一直没有返回成功。也就是说，只有先注册的app_ws是生效的。通过跟踪源码，发现在WebSocketServerProtocolHandshakeHandler类里面，处理握手时，会判断当前端点是否已经注册的端点。
```java
public void channelRead(final ChannelHandlerContext ctx, Object msg) throws Exception {
    final FullHttpRequest req = (FullHttpRequest)msg;
    if (this.isNotWebSocketPath(req)) {
        ctx.fireChannelRead(msg);
    } else {
      …
	}
	…
}
```
也就是说，后面的端点handler没有注册进来，又跟踪注册的代码，发现在WebSocketServerProtocolHandler里面有判断。
```java
public void handlerAdded(ChannelHandlerContext ctx) {
    ChannelPipeline cp = ctx.pipeline();
    if (cp.get(WebSocketServerProtocolHandshakeHandler.class) == null) {
        cp.addBefore(ctx.name(), WebSocketServerProtocolHandshakeHandler.class.getName(), new WebSocketServerProtocolHandshakeHandler(this.serverConfig));
    }

    if (this.serverConfig.decoderConfig().withUTF8Validator() && cp.get(Utf8FrameValidator.class) == null) {
        cp.addBefore(ctx.name(), Utf8FrameValidator.class.getName(), new Utf8FrameValidator());
    }

}
```
也就是说在管线(pipeline)里面，只能有一个WebSocketServerProtocolHandshakeHandler的类实例，我尝试绕过这个现实，想继承WebSocketServerProtocolHandshakeHandler实现自己的handlerAdded方法，发现没有成功，原因：
- WebSocketServerProtocolHandshakeHandler只能包内访问
- serverConfig属于类私有，子类无法访问。

虽然魔改不行，还是有办法的，就是多端口，nginx反向代理，根据不同的端点来访问，这样实际内部是两个netty服务来处理。


### 参考

https://netty.io/wiki/user-guide-for-4.x.html

https://www.cnblogs.com/crazymakercircle/p/9853586.html

