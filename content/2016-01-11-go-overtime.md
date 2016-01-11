Title: Go 超时的应用
Date: 2016-01-11 10:20
Modified: 2016-01-11 19:30
Category: 技术
Tags: go 
Slug: my-super-post1
Authors: vantigege
Summary: 介绍Go中的超时处理

## 单个超时
```
package main
import (
)
```


下载标准安装包

```
http://golang.org/mkdidl/
```
<br />

```
解压到/usr/local目录sudo tar -xzvf go1.5.2.linux-amd64.tar.gz /usr/local
```
<br />

```
在$HOME目录下创建文件夹gopath
```
<br />

在~/.bashrc添加如下内容，并source .bashrc

```
export GOPATH=$HOME/gopath
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
```
<br />

# 源码安装nsq 
依次执行如下步骤

```
go get github.com/tools/godep
go get github.com/bmizerany/assert
godep get github.com/bitly/nsq/...
```
<br />

godep执行之后，如果报错：

```
godep: outdated Godeps missing source code
This dependency list was created with an old version of godep.
To work around this, you have two options:
1. Run ‘godep restore‘, and try again.
2. Ask the maintainer to switch to a newer version of godep,
then try again with the updated package.
```

解决办法

```
cd src/github.com/bitly/nsq
rm Godeps
godep save ./...
godep get github.com/bitly/nsq/...
```
<br />

# 部署测试
依次在不同终端执行如下命令
<br />

启动nsqlookupd

```
nsqlookupd
```
<br />

启动nsqd

```
nsqd --lookupd-tcp-address=127.0.0.1:4160
```
<br />


启动nsqadmin

```
nsqadmin --lookupd-http-address=127.0.0.1:4161
```
<br />

向nsqd发送消息

```
curl -d "hello world 1" "http://127.0.0.1:4151/put?topic=test"
```
<br />

启动nsq_to_file

```
nsq_to_file --topic=test --output-dir=/home/root --lookupd-http-address=127.0.0.1:4161
```
<br />

向nsqd发送消息

```
curl -d "hello world 2" "http://127.0.0.1:4151/put?topic=test"
```
<br />

向nsqd发送消息

```
curl -d "hello world 3" "http://127.0.0.1:4151/put?topic=test"
```
<br />

nsqlookupd是枢纽。nsqd是接受和转发消息的服务。nsqadmin是用于查看转态的web服务。nsq_to_file是消息的消费者。消息可以由nsqd经过nsqlookup来转发，也可以nsqd之间之间转发。
curl -d是以http的post方式向nsqd发出消息。
<br />
<br />


# nsq编程测试
以python代码示例
<br />

安装pynsq

```
pip install pynsq
```
<br />

启动nsq服务

```
nsqlookupd
nsqd --lookupd-tcp-address=127.0.0.1:4160
```
<br />

生产者：

```
import nsq
import tornado.ioloop
import time

def pub_message():
    writer.pub('test', time.strftime('%H:%M:%S'), finish_pub)

def finish_pub(conn, data):
    print "It is ", data

reconnect_interval = 10.0
writer = nsq.Writer(nsqd_tcp_addresses=['127.0.0.1:4150'],
                            reconnect_interval=reconnect_interval,
                            name='test')

tornado.ioloop.PeriodicCallback(pub_message, 3000).start()
nsq.run()
```

生产者端将输出如下信息

```
It is  OK
It is  OK
It is  OK
It is  OK
...
```
<br />

消费者

```
import nsq
from pprint import pprint

def handler(message):
    print message.body
    return True

r = nsq.Reader(message_handler=handler,
        lookupd_http_addresses=['127.0.0.1:4161'],
        topic='test', channel='t1', lookupd_poll_interval=15)
nsq.run()
```
<br />

消费者端将输出如下信息

```
16:49:59
16:50:02
16:50:05
16:50:08
...
```


