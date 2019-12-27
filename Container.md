# 容器组件(Container)
Container用于类的注册创建，用于解决GO语言下不能通过字符串实例化一个类的问题。

通常组件对象都是全局单例，而一个请求周期内的对象会随着请求结束而销毁。容器默认会对当前上下文通过`GetObjPool`获取的对象进行缓存，在请求结束时回收缓存的对象，由于增加对象池后每次获取对象都要重置对象的属性，从压测数据看QPS会降低1%左右，容器的对象池可以通过`app.container.enablePool=off`关闭。

除全局对象外，一个请求周期内的对象通常会继承pgo2.Object以加入请求上下文件支持。

## 配置文件
配置位于(app.yaml)中

```yaml
container:
    # 是否启用对象池，默认为on
    enablePool: "off"
```

## 使用示例
```go
var PeopleClass string
// 注册类
func init() {
    container := pgo2.App().GetContainer() // 获取容器对象
    PeopleClass = container.Bind(&People{}) // 注册类模板对象
}
// 方法定义遵循 iface.IObjPoolFunc  接口
func NewPeoplePool(iObj iface.IObject, params ...interface{}) iface.IObject{
    obj:=iObj.(*People)
    obj.name = "unknown"
    obj.age  = 0
    obj.sex  = "unknown"
        
    return p
}

type People struct {
    // 继承自pgo2.Object可增加上下文支持，
    // 由于组件是全局对象，没有请求上下文，
    // 所以组件是不能继承自pgo2.Object的。
    pgo2.Object

    name    string
    age     int
    sex     string
}

// 可选设置函数，根据配置自动调用
func (p *People) SetName(name string) {
    p.name = name
}

func (p *People) SetAge(age int) {
    p.age = age
}

func (p *People) SetSex(sex string) {
    p.sex = sex
}

// 从对象池获取People的对象
func (t *TestController) ActionTest() {
    // 方法1: 初始化函数
    p1 := t.GetObjPool(PeopleClass, NewPeoplePool).(*People)

    // 方法2: 如果初始化函数定义有参数，可以传递参数
    p2 := t.GetObjPool(PeopleClass, NewPeoplePool, arg1, arg2).(*People)
    
    // 方法3：不需要初始化函数
    p3 := t.GetObjPool(PeopleClass, nil).(*People)



}
```
