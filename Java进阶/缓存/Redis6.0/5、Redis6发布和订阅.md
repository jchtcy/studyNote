# 一、概念
````
Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

Redis 客户端可以订阅任意数量的频道。
````
# 二、实现
````
subscribe channe1
客户端订阅 channe1

publish channe1 hello
另一个客户端发送消息 hello

客户端接收到消息
````