# Es

Es组件是对Elasticsearch封装,支持批量和异步写入

## 配置文件

```yaml
app.yaml
# 组件ID，es组件可以配置多个，只要分配唯一的组件ID, 通过
# GetObj(adapter.NewEs(), <id>)来获取非默认es组件
components:
    es:
      # host
      esHost: "http://127.0.0.1:9200/"
      # 默认http 组件
      # httpId:    "http" 
      # 默认0重试1次
      # retryNum: 0
      # 队列数量 默认500 
      # batchChanLen: 500 
      # 批量操作刷新间隔时间 默认50秒
      # batchFlushInterval: "50s" 
      # 批量请求es服务器超时时间 默认30秒
      # batchEsTimeout:"30s" 
      # 单个请求es服务器超时时间 默认500毫秒
      # singleEsTimeout: "500ms"
      # 缓冲最条数,默认600
      # maxBufferLine: 600 
      # 写缓冲大小 单位bite 默认1M
      # maxBufferByte: 1048576 
```

## 功能列表

```go
// 从对象池获取对象 
this.GetObjBox(adapter.EsClass).(adapter.IEs)/(*adapter.Es) 
// 普通方法列表
NewEs(componentId ...string) // 对象  this.GetObj(adapter.NewEs()).(adapter.IEs)/(*adapter.Es)
Single(method, uri string, body []byte, timeout time.Duration) ([]byte, error) // 单个请求

Batch(action, head, body string) error // 批量异步执行
```

## 使用示例

```go

type EsController struct {
    pgo2.Controller
}

// curl -v http://127.0.0.1:8000/es/exec
func (m *MysqlController) ActionExec() {
    esClient := t.GetObjBox(adapter.EsClass).(adapter.IEs)
   indexName := "test"
   indexName1 := "/test/_doc"

   type tData struct {
       Name string `json:"name"`
       Age int `json:"age"`
   }

   data := &tData{Name:"name1",Age:2}

   esClient.Batch("index", `{"_index":"`+indexName+`","_type":"_doc"}`, util.ToString(data))
   esClient.Single("POST",indexName1,[]byte(util.ToString(data)),1*time.Second)
}

```



