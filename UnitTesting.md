# 单元测试(UnitTesting)
go test 原生命令, 如果用到config 可能需要配置--base=/xxx/xxx/appPath
```go
go test -run '' ./... --args --base=/xxx/xxx/appPath
```

## 注入pgo2.Context
```go 
obj := &pgo2.Object{}
context := &pgo2.Context{}
context.Start(nil) // 被测试对象有写日志,初始化日志组件
obj.SetContext(context)
adObj := obj.GetObjPool(XxxClass, nil).(*Xxx)
```
## mock日志组件
```go
func (m *mockTarget) Process(item *logs.LogItem) {
	// fmt.Println("Process")
}

// Flush flush log to stdout
func (m *mockTarget) Flush(final bool) {
	//fmt.Println("Flush")
}


func setTarget() {
	pgo2.App().Log().SetTarget("info", &mockTarget{})
	pgo2.App().Log().SetTarget("error", &mockTarget{})
	pgo2.App().Log().SetTarget("console", &mockTarget{})
}
func init(){
	setTarget()
}
obj := &pgo2.Object{}
context := &pgo2.Context{}
context.Start(nil) // 被测试对象有写日志,初始化日志组件
obj.SetContext(context)
adObj := obj.GetObjPool(XxxClass, nil).(*Xxx)
```

