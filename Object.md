# 获取对象(Object)

获取对象并注入Context上下文，还可以实现对象池，单例

## 方法列表
```go
    # 设置上下文
	SetContext(ctx IContext) IObject
    # 获取上下文
	Context() IContext
    # 获取对象并注入上下文
	GetObj(obj IObject) IObject
    # 从对象池获取对象并注入上下文,如果有定义Prepare方法，会自动调用,funcName!=nil,params会传入funcName，也会传入Prepare
    # (推荐用GetObjBox替换)
	GetObjPool(className string, funcName IObjPoolFunc, params ...interface{}) IObject
    # 从对象池获取对象并注入上下文,如果有定义Prepare方法，会自动调用，并把params传入
	GetObjBox(className string, params ...interface{}) IObject
    # 获取单例对象并注入上下文
	GetObjSingle(name string, funcName IObjSingleFunc, params ...interface{}) IObject
    # 获取对象并注入自定义上下文
    GetObjCtx(ctx IContext, obj IObject) IObject
    # 从对象池获取对象并注入自定义上下文
    # (推荐用GetObjBoxCtx替换)
	GetObjPoolCtx(ctr IContext, className string, funcName IObjPoolFunc, params ...interface{}) IObject
    # 从对象池获取对象并注入自定义上下文
	GetObjBoxCtx(ctr IContext, className string, params ...interface{}) IObject
    # 获取单例对象并注入自定义上下文
	GetObjSingleCtx(ctx IContext, name string, funcName IObjSingleFunc, params ...interface{}) IObject
```

## 使用示例

```go

func NewLogicData() *LogicData{
	return &LogicData{}
}

type LogicData struct {
	pgo2.Object
	data string
}

func (l *LogicData) Show(){
	fmt.Println("LogicData show data:" + l.data)
}



type LogicData1 struct {
	pgo2.Object
	data string
}

func (l *LogicData1) Show(){
	fmt.Println("LogicData1 show data:" + l.data)
}

// 定义对象池初始化函数需要遵循iface.IObjPoolFunc 接口
func NewLogicDataPool(iObj iface.IObject, params ...interface{}) iface.IObject{
	obj := iObj.(*LogicData2)
	if len(params)>0 {
		obj.data = params[0].(string)
	}

	return obj
}

type LogicData2 struct {
	pgo2.Object
	data string
}

func (l *LogicData2) Show(){
	fmt.Println("LogicData2 show data:" + l.data)
}

func NewLogicData3Single(params ...interface{}) iface.IObject{
	data := ""
	if len(params) > 0{
		data = params[0].(string)
	}
	return &LogicData3{data:data}
}

type LogicData3 struct {
	pgo2.Object
	data string
}

func (l *LogicData3) Show(){
	fmt.Println("LogicData3 show data:" + l.data)
}

type LogicData4 struct {
	pgo2.Object
	data string
}

func (l *LogicData4) Prepare(a string,b int){
	fmt.Println("LogicData4 Prepare a:", a,"b",b)
}

func (l *LogicData4) Show(){
	fmt.Println("LogicData3 show data:" + l.data)
}

type LogicData5 struct {
	pgo2.Object
	data string
}

func (l *LogicData5) Prepare(params ...string){
	fmt.Println("LogicData5 Prepare params:", params)
}

func (l *LogicData5) Show(){
	fmt.Println("LogicData5 show data:" + l.data)
}

type Demo struct {
	pgo2.Object
}

var LogicData1Class string
var LogicData2Class string
var LogicData3Class string
var LogicData4Class string
var LogicData5Class string
// 如果需要用到对象池，需要先绑定对象
func init (){
	container := pgo2.App().Container()
	LogicData1Class = container.Bind(&LogicData1{})
	LogicData2Class = container.Bind(&LogicData2{})
	LogicData3Class = container.Bind(&LogicData3{})
	LogicData4Class = container.Bind(&LogicData4{})
	LogicData5Class = container.Bind(&LogicData5{})
}

func (d *Demo) Index(){
	// 获取对象并注入上下文
	l:=d.GetObj(NewLogicData()).(*LogicData)
	l.Show()
	// 获取对象并注入自定义上下文
	l0:=d.GetObjCtx(d.Context().Copy(), NewLogicData()).(*LogicData)
	l0.Show()

	// 从对象池获取对象并注入上下文(推荐用GetObjBox替换)
	l1 := d.GetObjPool(LogicData1Class, nil).(*LogicData1)
	l1.Show()
	// 从对象池获取对象并注入上下文,并初始化函数
	l2 := d.GetObjPool(LogicData2Class, NewLogicDataPool).(*LogicData2)
	l2.Show()
	// 从对象池获取对象并注入上下文,并初始化函数
	l3 := d.GetObjPoolCtx(d.Context().Copy(), LogicData2Class, NewLogicDataPool).(*LogicData2)
	l3.Show()

	// 获取单例对象并注入上下文
	l4:=d.GetObjSingle("logicData3",NewLogicData3Single,"dataStr").(*LogicData3)
	l4.Show()

	// 获取单例对象并注入自定义上下文
	l5:=d.GetObjSingleCtx(d.Context().Copy(), "logicData3-1",NewLogicData3Single,"dataStr").(*LogicData3)
	l5.Show()
	
	// 从对象池获取对象并注入上下文,并初始化函数
    l6 := d.GetObjBox(LogicData4Class, "aa",22).(*LogicData4)
    l6.Show()
    
    // 从对象池获取对象并注入上下文,并初始化函数
    l7 := d.GetObjBoxCtx(d.Context().Copy(), LogicData5Class, "aa","bb").(*LogicData5)
    l7.Show()
}

func NewDemo() *Demo{
	return &Demo{}
}

```



