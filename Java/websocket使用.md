# `Websocket`使用

## 方式1：原生注解

+ 监听路径：`@ServerEndpoint`
+ 建立连接：`@OnOpen`
+ 接收消息：`@OnMessage`
+ 关闭连接：`@OnClose`
+ 连接错误：`@OnError`

### 1. 引入`POM`依赖

````xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
````

### 2. 配置文件

````java
@Configuration
@EnableWebSocket
public class WebSocketConfig {
 @Bean
 public ServerEndpointExporter serverEndpointExporter() {
  return new ServerEndpointExporter();
 }
}
````

### 3. 连接处理

````java
@Slf4j
@ServerEndpoint("/ws-demo")
@Component
public class TestWSEndPoint {
 private static ConcurrentHashMap<String, Session> map = new ConcurrentHashMap<>();

 @OnOpen
 public void onOpen(Session session) {
  log.info("接受连接: " + session);
  map.put(session.getId(), session);
 }

 @OnMessage
 public void onMessage(Session session, String message) throws IOException {
  session.getBasicRemote().sendText("返回响应：" + message);
 }

 @OnClose
 public void onClose(Session session) {
  log.info("关闭连接: " + session);
  map.remove(session.getId());
 }

 @OnError
 public void onError(Session session, Throwable throwable) throws Exception {
  log.error("错误连接: " + session + "throwable : " + throwable.getMessage());
 }

}
````

## 方式2：使用`Spring`封装

### 1. 引入`POM`依赖

````xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-websocket</artifactId>
</dependency>
````

### 2. 编写处理器`WebsocketHandler`

````java
@Component
@Slf4j
public class WebsocketCustomHandler implements WebSocketHandler {
 @Override
 public void afterConnectionEstablished(WebSocketSession session) throws Exception {
  log.info("接收到新的连接" + session.getId());
 }

 @Override
 public void handleMessage(WebSocketSession session, WebSocketMessage<?> message) throws Exception {
  String text = "接收到消息：" + message.toString() + "session = " + session.getId();
  log.info(text);
  session.sendMessage(new TextMessage(text));
  // session.sendMessage(new
  // BinaryMessage(text.getBytes(StandardCharsets.UTF_8)));
  // session.sendMessage(new PingMessage());
  // session.sendMessage(new PongMessage());
 }

 @Override
 public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
  log.info("连接错误" + exception + "session =" + session.getId());
 }

 @Override
 public void afterConnectionClosed(WebSocketSession session, CloseStatus closeStatus) throws Exception {
  log.info("关闭连接 = " + closeStatus.toString() + "session =" + session.getId());
 }

 @Override
 public boolean supportsPartialMessages() {
  // 支持分片
  return false;
 }
}
````

### 3. 注册`Handler`

````java
@Configuration
@EnableWebSocket
public class SpringWebsocketCofig implements WebSocketConfigurer {

 @Autowired
 WebsocketCustomHandler websocketCustomHandler;

 @Override
 public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
  registry.addHandler(websocketCustomHandler, "/spring-ws").setAllowedOrigins("*");
 }
}
````

## 方式3：使用`STOMP`

## 方式4：使用`Netty` 等其他方式
