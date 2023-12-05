---
title: 使用js实现MQTT
date: 2019-12-17 10:33:48
tags: MQTT
---
MQTT是一个长连接的通讯应用层协议，最大的特点是数据精简、消息可靠、Publish-Subscribe模式灵活易用。MQTT已经成为IoT传输的标准协议，应用非常广泛。

MQTT被广泛使用的一个重要的原因是MQTT的生态非常完善，同时也支持JavaScript。因此下图所示的所有链路和模块，都可以通过JavaScript实现。
![avatar](/images/mqtt.png)

### JavaScript在MQTT架构中常用的框架
mosca（https://github.com/mcollina/mosca） 
mosca是一个用JavaScript实现的MQTT Broker。不仅如此，mosca还增加了对数据库，如Redis、MongoDB的支持，用来实现消息数据的存储。

MQTT.js（https://github.com/mqttjs/MQTT.js） 
MQTT.js是官网推荐的JavaScript实现的Client端。

KOA和Express 
这两者都是非常主流的Node版本的Server，简单易用。

### Broker端的实现
如果在公司的局域网开发，则可以忽略这一步
- 1.安装mosca。

```javascript
nmp install mosca --save
```
- 2.启动mosca。这里需要注意，如果本地没有配置MongoDB，则需要把ascoltatore中的内容全部注释掉。

```javascript
var mosca = require('mosca');

var ascoltatore = {
  //using ascoltatore
  // type: 'mongo',
  // url: 'mongodb://localhost:27017/mqtt',
  // pubsubCollection: 'ascoltatori',
  // mongo: {}
};

var settings = {
  port: 1883,
  backend: ascoltatore
};

var server = new mosca.Server(settings);

server.on('clientConnected', function(client) {
    console.log('client connected', client.id);
});

// fired when a message is received
server.on('published', function(packet, client) {
  console.log('Published', packet.payload); //{"clientId":"mqttjs_02fea7b4","topic":"/tips"}
  // console.log('>>>packet', packet); //{"clientId":"mqttjs_02fea7b4","topic":"/tips"}
});

server.on('ready', setup);

// fired when the mqtt server is ready
function setup() {
  console.log('Mosca server is up and running');
}
```
代码完成后，启动文件，本地的一个Broker就跑在localhost的1883端口上了。

### Client端发布实现
Client使用MQTT.js实现
当我们发布一个topic时，要用到publish方法，该方法第一个参数为字符串格式的topic
第二个参数可有可无，为字符串格式的消息内容
- 1.安装

```javascript
npm install mqtt --save
```
- 2.启动

```javascript
var mqtt = require('mqtt');
//创建一个链接
var client  = mqtt.connect('mqtt://localhost:1883');
//链接成功时
client.on('connect', function () {
   console.log('>>> connected')
   // client.subscribe('/tips')
   setInterval(
       //定时发布消息
        ()=>{client.publish('/temperature', '30');},
        3000
    );
})
//链接错误时
client.on('error', function(){
    console.log('链接错误')
})
//链接关闭时
client.on('close', function(){
    console.log('已关闭链接')
})
```

### Client端订阅实现
订阅者要订阅一个topic，可以按条件筛选topic
注意，message是按照buffer格式传递的，所以要解析出数据
```javascript
var mqtt = require('mqtt');
//创建一个链接
var client  = mqtt.connect('mqtt://localhost:1883');
//链接成功时
client.on('connect', function () {
   console.log('>>> connected')
   //订阅一个topic
   client.subscribe('/temperature')
})
//链接错误时
client.on('error', function(){
    console.log('链接错误')
})
//链接关闭时
client.on('close', function(){
    console.log('已关闭链接')
})
//当有消息时
client.on('message', function (topic, message) {
  console.log('topic') // /temperature
  // message is Buffer
  console.log(message.toString()) // 30
  //消息传递完成后可以取消订阅
  client.unsubscribe('/temperature')
})
```
该案例是一个比较简单的通信案例：
A发布一个消息，B来订阅这个消息
同样的，A和B也可以既发布消息，也可以订阅消息

[戳此处👀别人家的博客](https://blog.csdn.net/tangxiaoyin/article/details/73743166)