---
title: MQTT
date: '2025-11-25 19:46:29'
updated: '2025-11-26 17:16:46'
---
<font style="background-color:rgba(255, 255, 255, 0);">在湾区杯第一次遇见 MQTT 协议，这个是物联网常见的协议</font>

<font style="background-color:rgba(255, 255, 255, 0);">对比看了一下，这个题目好像是 Pubsubclient 项目改了个名而已</font>

# <font style="background-color:rgba(255, 255, 255, 0);">协议相关</font>
<font style="background-color:rgba(255, 255, 255, 0);">客户端(订阅者)：通过</font>`<font style="background-color:rgba(255, 255, 255, 0);">broker</font>`<font style="background-color:rgba(255, 255, 255, 0);">接收到自己订阅的频道的消息，也可以向自己订阅的频道向</font>`<font style="background-color:rgba(255, 255, 255, 0);">broker</font>`<font style="background-color:rgba(255, 255, 255, 0);">发送消息</font>

<font style="background-color:rgba(255, 255, 255, 0);">服务端(发布者)：将对应频道的消息转发给订阅了该通道的订阅者</font>

![](/images/e73f1d53b11af00e37c96788a4b1fb81.webp)

# <font style="background-color:rgba(255, 255, 255, 0);">订阅/发布脚本相关</font>
```bash
import paho.mqtt.client as mqtt
import time
from pwn import *

broker = '127.0.0.1'
client_id = "orzzz"

# 用于接收订阅主题的消息
def on_message(client, userdata, msg):
    print(f"{msg.topic}: {msg.payload}")


client = mqtt.Client(client_id)		# client_id类似于一个识别id，随意
client.on_message = on_message		# 消息到达回调函数。当订阅到的主题收到消息时触发
client.connect(broker, 9999, 60)	# broker为中心代理的ip，第二个参数为端口，第三个参数为心跳时间60s
payload = b''
payload = payload.ljust(0x128, b'\x00')

time.sleep(0.5)
client.loop_start()					# 启动循环处理mqtt报文
client.subscribe('imi/cmd')			# 订阅imi/cmd主题
client.subscribe('imi/response')	# 订阅imi/response主题
client.publish('imi/cmd', payload) 	# 向imi/cmd主题发送payload，broker向所有订阅了该主题的进行广播

time.sleep(1)

client.loop_stop()					# 结束循环
client.disconnect()					# 关闭连接
```

<font style="background-color:rgba(255, 255, 255, 0);">网上找到了现成的用 paho 库实现的 mqtt 订阅和发布脚本，直接可以用的（吗？</font>

```bash
# python3.6
 
import random
 
from paho.mqtt import client as mqtt_client
 
 
broker = 'broker.emqx.io'
port = 1883
topic = "/python/mqtt"
# generate client ID with pub prefix randomly
client_id = f'python-mqtt-{random.randint(0, 100)}'
 
 
def connect_mqtt() -> mqtt_client:
    def on_connect(client, userdata, flags, rc):
        if rc == 0:
            print("Connected to MQTT Broker!")
        else:
            print("Failed to connect, return code %d\n", rc)
 
    client = mqtt_client.Client(client_id)
    client.on_connect = on_connect
    client.connect(broker, port)
    return client
 
 
def subscribe(client: mqtt_client):
    def on_message(client, userdata, msg):
        print(f"Received `{msg.payload.decode()}` from `{msg.topic}` topic")
 
    client.subscribe(topic)
    client.on_message = on_message
 
 
def run():
    client = connect_mqtt()
    subscribe(client)
    client.loop_forever()
 
 
if __name__ == '__main__':
    run()
```

```bash
# python 3.6
 
import random
import time
 
from paho.mqtt import client as mqtt_client
 
 
broker = 'broker.emqx.io'
port = 1883
topic = "/python/mqtt"
# generate client ID with pub prefix randomly
client_id = f'python-mqtt-{random.randint(0, 1000)}'
 
 
def connect_mqtt():
    def on_connect(client, userdata, flags, rc):
        if rc == 0:
            print("Connected to MQTT Broker!")
        else:
            print("Failed to connect, return code %d\n", rc)
 
    client = mqtt_client.Client(client_id)
    client.on_connect = on_connect
    client.connect(broker, port)
    return client
 
 
def publish(client):
    msg_count = 0
    while True:
        time.sleep(1)
        msg = f"messages: {msg_count}"
        result = client.publish(topic, msg)
        # result: [0, 1]
        status = result[0]
        if status == 0:
            print(f"Send `{msg}` to topic `{topic}`")
        else:
            print(f"Failed to send message to topic {topic}")
        msg_count += 1
 
 
def run():
    client = connect_mqtt()
    client.loop_start()
    publish(client)
 
 
if __name__ == '__main__':
    run()
```

# <font style="background-color:rgba(255, 255, 255, 0);">处理信息函数</font>
![](/images/4ac8589cab2fd796cc69d7d87280879d.png)

![](/images/c8c87329b40d9eae649b1725d68bf09b.png)

<font style="background-color:rgba(255, 255, 255, 0);">正常来说这些服务端的处理信息的程序都是通过子线程来处理接收到的信息的，之前没仔细了解过 pthread_create 函数，先来看看这个</font>

```rust
#include <pthread.h>

int pthread_create(
    pthread_t *thread, 
    const pthread_attr_t *attr,
    void *(*start_routine) (void *), 
    void *arg
);
```

`<font style="background-color:rgba(255, 255, 255, 0);">pthread_create</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 接收四个参数，它们共同定义了新线程的行为和身份</font>

#### <font style="background-color:rgba(255, 255, 255, 0);">1. </font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_t *thread</font>`<font style="background-color:rgba(255, 255, 255, 0);"> (线程 ID 输出)</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">类型：</font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_t</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 类型的指针</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">作用： 这是一个输出参数。函数成功返回后，它会存储新创建线程的唯一 ID（Identifier）</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">用途：</font><font style="background-color:rgba(255, 255, 255, 0);"> 线程 ID 是一个不透明（opaque）的数据类型，操作系统用于标识该线程。主线程可以使用这个 ID 来执行后续操作，例如：</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">等待线程结束：</font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_join()</font>`
    - <font style="background-color:rgba(255, 255, 255, 0);">分离线程：</font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_detach()</font>`
    - <font style="background-color:rgba(255, 255, 255, 0);">发送信号：</font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_kill()</font>`

#### <font style="background-color:rgba(255, 255, 255, 0);">2. </font>`<font style="background-color:rgba(255, 255, 255, 0);">const pthread_attr_t *attr</font>`<font style="background-color:rgba(255, 255, 255, 0);"> (线程属性)</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">类型：</font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_attr_t</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 类型的常量指针</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">作用： 用于指定新线程的各种属性</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">常见用法：通常设置为 </font>`<font style="background-color:rgba(255, 255, 255, 0);">NULL</font>`<font style="background-color:rgba(255, 255, 255, 0);">，表示使用系统默认属性</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">高级用法：</font><font style="background-color:rgba(255, 255, 255, 0);"> 如果需要自定义线程行为，必须先使用 </font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_attr_init()</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 初始化一个 </font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_attr_t</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 结构体，然后设置特定的属性，例如：</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">分离状态 (Detach State)：</font>
        * `<font style="background-color:rgba(255, 255, 255, 0);">PTHREAD_CREATE_JOINABLE</font>`<font style="background-color:rgba(255, 255, 255, 0);"> (默认)：线程终止后，其资源不会立即释放，必须等待另一个线程调用 </font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_join()</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 进行回收</font>
        * `<font style="background-color:rgba(255, 255, 255, 0);">PTHREAD_CREATE_DETACHED</font>`<font style="background-color:rgba(255, 255, 255, 0);">：线程终止后，系统会自动回收其所有资源（“即生即灭”），其他线程无法对其使用 </font>`<font style="background-color:rgba(255, 255, 255, 0);">pthread_join()</font>`<font style="background-color:rgba(255, 255, 255, 0);"></font>
    - <font style="background-color:rgba(255, 255, 255, 0);">堆栈大小 (Stack Size)： 设置线程的私有堆栈大小</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">调度策略/优先级： 设置线程的调度参数</font>

#### <font style="background-color:rgba(255, 255, 255, 0);">3. </font>`<font style="background-color:rgba(255, 255, 255, 0);">void *(*start_routine) (void *)</font>`<font style="background-color:rgba(255, 255, 255, 0);"> (线程入口函数)</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">类型： 函数指针</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">作用： 这是新线程开始执行的入口函数。当线程被创建成功后，它会立即调用这个函数</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">签名要求：</font><font style="background-color:rgba(255, 255, 255, 0);"> 线程入口函数的签名是固定的：</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">它必须接受一个 </font>`<font style="background-color:rgba(255, 255, 255, 0);">void *</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 类型的参数</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">它必须返回一个 </font>`<font style="background-color:rgba(255, 255, 255, 0);">void *</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 类型的值</font>

#### <font style="background-color:rgba(255, 255, 255, 0);">4. </font>`<font style="background-color:rgba(255, 255, 255, 0);">void *arg</font>`<font style="background-color:rgba(255, 255, 255, 0);"> (传递给入口函数的参数)</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">类型：</font>`<font style="background-color:rgba(255, 255, 255, 0);">void *</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 指针</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">作用： 用于将数据传递给 </font>`<font style="background-color:rgba(255, 255, 255, 0);">start_routine</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 函数</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">机制：</font><font style="background-color:rgba(255, 255, 255, 0);"> 由于 </font>`<font style="background-color:rgba(255, 255, 255, 0);">start_routine</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 只能接受一个 </font>`<font style="background-color:rgba(255, 255, 255, 0);">void *</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 参数，这是传递任何类型数据的通用方法。</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">单个值： 可以直接传递一个整数或指针</font>
    - <font style="background-color:rgba(255, 255, 255, 0);">多个值： 如果需要传递多个参数或复杂数据结构，必须将这些数据打包到一个 </font>`<font style="background-color:rgba(255, 255, 255, 0);">struct</font>`<font style="background-color:rgba(255, 255, 255, 0);"> 中，然后传递该结构体的指针</font>

