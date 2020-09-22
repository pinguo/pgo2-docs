# 错误包
自定义错误
## 函数列表
```go

New(status int, msg ...interface{}) *Error 
// 初始化一个err类型的错误对象, 如果panic这个对象，pgo2框架会自动捕获，json输出status,msg,并写error日志
NewWarn(status int, msg ...interface{}) *Error 
// 初始化一个warn类型的错误对象, 如果panic这个对象，pgo2框架会自动捕获，json输出status,msg,并写warn日志
NewIgnore(status int, msg ...interface{}) *Error
// 初始化一个warn类型的错误对象, 如果panic这个对象，pgo2框架会自动捕获，json输出status,msg,不会写日志
```

## 使用说明

```go
type IndexController struct {
     pgo2.Controller
 }

 func (t *IndexController) BeforeAction(action string){
 	// curl -v http://127.0.0.1:8000/index/test1
 	// 写warn日志
 	// output : { "data": {}, "message": "panic 400", "status": 400 }
 	// 可用于一些参数判断和过滤后终止程序等
    if action == "test1"{
        panic(perror.NewWarn(400,"panic 400"))
    }
    
    // curl -v http://127.0.0.1:8000/index/test2
    // 写error日志
    // output : { "data": {}, "message": "panic 500", "status": 500 }
    if action == "test2"{
        panic(perror.New(500,"panic 500"))
    }
    
    // curl -v http://127.0.0.1:8000/index/test3
    // 不写日志
    // output : { "data": {}, "message": "panic 200", "status": 200 }
    if action == "test3"{
        panic(perror.NewIgnore(200,"panic 200"))
    }
 }

 // curl -v http://127.0.0.1:8000/index/index 
 func (m *IndexController) ActionIndex() {
 	
 }   
```