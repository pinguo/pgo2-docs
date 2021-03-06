# RabbitMq

RabbitMq组件提供了RabbitMq客户端的功能.支持Channel连接池，支持数据的gob序列化，支持故障自动转移,恢复,探测.
一个server地址对应一个tcp长链接，一个tcp链接对应一个Channel连接池
## 配置文件
app.yaml
```yaml
components:
    # 组件ID，默认为"rabbitMq"
    rabbitMq:
        # 用户
        user: ""
        # 密码
        pass: "" 
        # tls相关,PEM encoded certificates file.默认为空,可不配置
        tlsRootCAs: ""
        # tls相关,public key file，默认为空,可不配置
        tlsCert: ""
        # tls相关,private key file默认为空,可不配置
        tlsCertKey: ""
        # 交换机名,默认为direct_pgo_dft,可不配置
        exchangeName: "direct_pgo2_dft"
        # 交换机类型,默认为：direct,可不配置
        exchangeType: "direct"
        # 每个地址tcp channel最大连接池大小，默认为2000,可不配置
        maxChannelNum: 2000
        # 每个地址tcp channel最大空闲限制接池大小，默认为200，不能大于maxChannelNum,可不配置
        maxIdleChannel: 200
        # 连接空闲时间(超过这个时间将回收)，默认1分钟,可不配置
        maxIdleChannelTime: "60s"
        # Rabbit channel连接池等待时间 默认为200ms,可不配置
        maxWaitTime: "200ms"
        # 服务器探活tcp链接间隔(自动剔除和添加server)，默认关闭,可不配置
        probInterval: "0s"
        # 当前系统名字
        serviceName: "pgo2"
        # 服务器地址，如果server有权重，请自行按比例添加
        servers:
            - "127.0.0.1:5672"
            - "127.0.0.1:5672"
            - "127.0.0.1:5673"
```

## 功能列表

```go
// 从对象池获取对象
rabbitMq:=this.GetObjBox(adapter.RabbitMqClass).(adapter.IRabbitMq) // (v0.1.131+)
// 普通方法
NewRabbitMq(componentId ...string) // 对象 rabbitMq:=this.GetObj(adapter.NewRabbitMq()).(adapter.IRabbitMq)/(*adapter.RabbitMq)
rabbitMq.SetPanicRecover(v bool) // 是否Recover
rabbitMq.ExchangeDeclare(dftExchange ...*rabbitmq.ExchangeData)               // 定义交换机
rabbitMq.Publish(opCode, data, dftOpUid ...string)  bool           // 发布数据
rabbitMq.PublishExchange(serviceName, exchangeName, exchangeType, opCode string, data interface{}, dftOpUid ...string) bool   // 发布数据到指定的交换机
rabbitMq.GetConsumeChannelBox()          // 获取消费者RabbitMq channal对象
rabbitMq.Consume(queueName string, opCodes []string, limit int, autoAck, noWait, exclusive bool) <-chan amqp.Delivery              // 获取指定队列消息，返回go channel
rabbitMq.ConsumeExchange(exchangeName, exchangeType, queueName string, opCodes []string, limit int, autoAck, noWait, exclusive bool) <-chan amqp.Delivery  // 获取指定交换机的指定队列消息，返回go channel
rabbitMq.DecodeBody(d,ret)              // 解析内容,通过指针引用方式
rabbitMq.DecodeHeaders(d)               // 直接返回header


```

## 使用示例

```go
// curl -v http://127.0.0.1:8000/rabbit/publish
func (r *RabbitController) ActionPublish() {
    type pubData struct {
        Id int
        Name string
    }
    //获取rabbitMq上下文适配对象
    rabbit :=t.GetObj(adapter.NewRabbitMq()).(*adapter.RabbitMq)
    // 定义交换机,这里可以不用定义，那consumme需要先执行
    rabbit.ExchangeDeclare()
    // 发布消息
    ret := rabbit.Publish("test",&pubData{Id:1,Name:"name1"})
    if ret == true {
        fmt.Println("成功")
    } else {
         fmt.Println("失败")
    }
}

// curl -v http://127.0.0.1:8000/rabbit/consummer
func (r *RedisController) ActionConsummer() {
    type pubData struct {
            Id int
            Name string
        }
    //获取rabbitMq上下文适配对象
    rabbit :=t.GetObjBox(adapter.RabbitMqClass).(*adapter.RabbitMq)
    opCodes := []string{"test"}
    // 此程序为常驻程序，会一直监听消息,最好为脚本运行
    rets :=rabbit.Consume("testQueue",opCodes,1,false,false,false)
    for d:=range rets{
        // 获取header 
        headers := rabbit.DecodeHeaders(d)
        fmt.Println("headers",Util.ToString(headers))
        var ret *pubData
        rabbit.DecodeBody(d,&ret)
        fmt.Println(Util.ToString(ret))
        // 回复收到
        d.Ack(false)
    }
    
}
```

