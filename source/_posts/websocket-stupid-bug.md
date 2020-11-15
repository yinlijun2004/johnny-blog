---
title: 记我的一个关于websocket的很傻X的bug。
date: 2020-11-15 22:46:58
tags: [websocket, spring boot]
---

用Spring Boot集成过WebSocket的基本都会碰到一个Bug，报错如下：
```
The remote endpoint was in state [TEXT_FULL_WRITING] which is an invalid state for called method
```
TEXT_FULL_WRITIN表示文本发送状态错误，如果发送的是二进制流，那报错的是BINARY_FULL_WRITING。

这是由于多个线程同时对一个session发送导致的，解决的方法也很简单，将发送方法用synchronized保护起来。

其实我也是这样做的，但是还是有这样的问题，我一度怀疑是广大网友们错了，但是今天我偶然发现，是我傻X了， 想广大网友道歉。

```java
 private void sendContentLock(ByteBuffer content) {
        synchronized (locker) {
            if (session.isOpen()) {
                try {
                    //奏是这货，我写成了session.getAsyncRemote()，o(╥﹏╥)o
                    session.getBasicRemote().sendBinary(content);
                } catch (Exception e) {
                    String message = e.getMessage();
                    if(!(message.contains("[TEXT_FULL_WRITING]") || message.contains("[BINARY_FULL_WRITING]"))) {
                        log.error("发送消息异常，即将关闭连接：{}", e.getMessage());
                        try {
                            session.close();
                        } catch (Exception ex) {
                            log.error("关闭连接异常：{}", ex.getMessage());
                        }
                    }

                }
            }
        }
    }
```

可不是吗，getAsyncRemote是异步发送，可不就是又起了一个线程发吗，这样synchronized又不起作用了。不过这API设计的也真实的，既然不能同时发，为啥要给我提供这么个接口，不知道干啥用。就是听了某些别有用心的网友的话，才用getAsyncRemote的，当时也没有理解getAsyncRemote的意思。

getAsyncRemote的[官方解释](https://docs.oracle.com/javaee/7/api/javax/websocket/Session.html#getAsyncRemote--)是
```
Return a reference a RemoteEndpoint object representing the peer of this conversation that is able to send messages asynchronously to the peer.
```
说人话，就是异步发送，但是限制就是不能发太快？那我要发很快呢？