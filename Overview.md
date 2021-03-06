# 项目背景
pgo2应用框架即"PinGuo GO application framework 2.0"，是Camera360服务端团队基于[pgo](https://github.com/pinguo/pgo)研发的一款简单、高性能、组件化的GO应用框架。
受益于GO语言高性能与原生协程，使用pgo2可快速地开发出高性能的web应用程序。

# 基准测试
主要测试PGO2框架与PGO的性能差异。

说明:
- 测试机为4核8G阿里云主机
- go版本为1.13.5, GOMAXPROCS=4
- 输出均为字符串{"code": 200, "message": "success","data": "hello world"}
- 命令: ab -n 1000000 -c 100 -k 'http://target-ip:8000/welcome/index'

分类 | QPS | 平均响应时间(ms) |CPU
---- | ---- | ---- | -----
pgo1 | 59917.26 | 1.669 | 63.36%
pgo2 | 62442.44 | 1.601 | 41.52%


pgo与其他比较(https://github.com/pinguo/pgo-docs)

# 环境要求
- GO 1.13+
- Make 3.8+
- Linux/MacOS/Cygwin
- GoLand 2019 (建议)

# 项目目录
规范：
1.基于go module
2. 按go标准规范，项目源码文件与目录使用小写形式

```
<project>
├── bin/                # 编译程序目录
├── configs/            # 配置文件目录
│   ├── production/     # 环境配置目录
│   │   ├── app.yaml
│   │   └── params.yaml
│   ├── testing/
│   ├── app.yaml        # 项目配置文件
│   └── params.yaml     # 自定义配置文件
├── makefile            # 编译打包
├── runtime/            # 运行时目录 主要用于存储日志
├── assets/             # 非web相关资源
├── web/                # 静态资源目录
│   ├── static          # 静态资源
│   ├── template        # 视图模板目录
├── cmd/                # 项目入口目录
└── pkg/                # 项目源码目录
    ├── command/        # 命令行控制器目录
    ├── controller/     # HTTP控制器目录
    ├── lib/            # 项目基础库目录
    ├── model/          # 模型目录(数据交互)
    ├── service/        # 服务目录(业务逻辑)
    ├── structs/         # 结构目录(数据定义)
```

# 依赖管理
go modules

```
用 go help module-get 和 go help gopath-get分别去了解 Go modules 启用和未启用两种状态下的 go get 的行为
用 go get 拉取新的依赖
拉取最新的版本(优先择取 tag)：go get golang.org/x/text@latest
拉取 master 分支的最新 commit：go get golang.org/x/text@master
拉取 tag 为 v0.3.2 的 commit：go get golang.org/x/text@v0.3.2
拉取 hash 为 342b231 的 commit，最终会被转换为 v0.3.2：go get golang.org/x/text@342b2e
用 go get -u 更新现有的依赖
用 go mod download 下载 go.mod 文件中指明的所有依赖
用 go mod tidy 整理现有的依赖
用 go mod graph 查看现有的依赖结构
用 go mod init 生成 go.mod 文件 (Go 1.13 中唯一一个可以生成 go.mod 文件的子命令)
用 go mod edit 编辑 go.mod 文件
用 go mod vendor 导出现有的所有依赖 (事实上 Go modules 正在淡化 Vendor 的概念)
用 go mod verify 校验一个模块是否被篡改过           # 更新依赖包
```

# 快速开始

1. 拷贝makefile
    
    非IDE环境(命令行)下，推荐使用make做为编译打包的控制工具，从[pgo2](https://github.com/pinguo/pgo2)或[pgo2-demo](https://github.com/pinguo/pgo2-demo)将makefile复制到项目目录下。
    ```sh
    make start      # 编译并运行当前工程
    make stop       # 停止当前工程的进程
    make build      # 仅编译当前工程
    make update     # go get
    make install    # go mod download
    make pgo2       # 安装pgo2框架到当前工程
    make init       # 初始化工程目录
    make help       # 输出帮助信息
    ```

2. 创建项目目录(以下三种方法均可)
    - 执行`make init`创建目录
    - 参见《项目目录》手动创建
    - 从[pgo2-demo](https://github.com/pinguo/pgo2-demo)克隆目录结构

3. 修改配置文件(conf/app.yaml)
    ```yaml
    name: "pgo2-demo"
    GOMAXPROCS: 2
    runtimePath: "@app/runtime"
    publicPath: "@app/web/static"
    viewPath: "@app/web/template"
    server:
        httpAddr: "0.0.0.0:8000"
        readTimeout: "30s"
        writeTimeout: "30s"
    components:
        log:
            levels: "ALL"
            targets:
                info:
                    levels: "DEBUG,INFO,NOTICE"
                    filePath: "@runtime/info.log"
                error:
                    levels: "WARN,ERROR,FATAL"
                    filePath: "@runtime/error.log"
                console: 
                    levels: "ALL"
    ```

4. 安装pgo2(以下两种方法均可)
    - 在项目根目录执行`make pgo2`安装pgo2
    - 在项目根目录执行`go get -u github.com/pinguo/pgo2`
5. 创建service(pkg/service/Welcome.go)
    ```go
    package service

    import (
        "fmt"

        "github.com/pinguo/pgo2"
    )

    func NewWelcome() *Welcome{
       return &Welcome{}
    }

    type Welcome struct {
        pgo2.Object
    }
    
    func (w *Welcome) SayHello(name string, age int, sex string) {
        fmt.Printf("call in  service/Welcome.SayHello, name:%s age:%d sex:%s\n", name, age, sex)
    }
   
    ```
6. 创建控制器(pkg/controller/welcome.go)
需要提前注册controller : https://github.com/pinguo/pgo2-demo/blob/master/pkg/controller/init.go
```go
package controller

import (
    "fmt"
    "net/http"

    "pgo2-demo/pkg/service"

    "github.com/pinguo/pgo2"
)

type Welcome struct {
    pgo2.Controller
}

// 预处理函数，控制器初始化的时候可以执行
func (w *Welcome) Prepare(){

}

// curl -v http://127.0.0.1:8000/welcome/index
// 默认动作 curl -v http://127.0.0.1:8000/welcome
func (w *Welcome) ActionIndex() {
    w.Json("hello world ActionIndex", http.StatusOK)
}

// curl -v http://127.0.0.1:8000/welcome/view
// 模板渲染
func (w *Welcome) ActionView() {
    // 获取并验证参数
    name := w.Context().ValidateParam("name", "hitzheng").Do()
    age := w.Context().ValidateParam("age", "100").Int().Do()

    data := map[string]interface{}{
        "name": name,
        "age":  age,
    }

    // 渲染html模板
    w.View("welcome.html", data)
}

// curl -v http://127.0.0.1:8000/welcome/say-hello?name=sssss&age=1
// URL路由控制器，根据url自动映射控制器及方法，不需要配置.
// url的最后一段为动作名称，不存在则为index,
// url的其余部分为控制器名称，不存在则为index,
// 例如：/welcome/say-hello，控制器类名为
// controller/Welcome 动作方法名为ActionSayHello
func (w *Welcome) ActionSayHello() {
    ctx := w.Context() // 获取PGO2请求上下文件

    // 验证参数，提供参数名和默认值，当不提供默认值时，表明该参数为必选参数。
    // 详细验证方法参见Validate.go
    name := ctx.ValidateParam("name").Min(5).Max(50).Do()          // 验证GET/POST参数(string)，为空或验证失败时panic
    age := ctx.ValidateQuery("age", 20).Int().Min(1).Max(100).Do() // 只验证GET参数(int)，为空或失败时返回20
    ip := ctx.ValidatePost("ip", "").IPv4().Do()                   // 只验证POST参数(string), 为空或失败时返回空字符串

    // 打印日志
    ctx.Info("request from welcome, name:%s, age:%d, ip:%s", name, age, ip)
    ctx.PushLog("clientIp", ctx.ClientIp()) // 生成clientIp=xxxxx在pushlog中

    // 调用业务逻辑，一个请求生命周期内的对象都要通过GetObj()获取，
    // 这样可自动查找注册的类，并注入请求上下文(Context)到对象中。
    svc := w.GetObj(service.NewWelcome()).(*service.Welcome)

    // 添加耗时到profile日志中
    ctx.ProfileStart("Welcome.SayHello")
    svc.SayHello(name, age, ip)
    ctx.ProfileStop("Welcome.SayHello")

    // 调用业务逻辑，一个请求生命周期内的对象通过GetObjBox()从对象池获取对象，
    // 这样可自动查找注册的类，并注入请求上下文(Context)到对象中。
    // 从对象池获取对象，并初始化Prepare方法,第二个或者后面的参数为Prepare定义参数
    svcPool1 := w.GetObjBox(service.WelcomeClass, "1123").(*service.Welcome)
    svcPool1.ShowId()

    data := map[string]interface{}{
        "name": name,
        "age":  age,
        "ip":   ip,
    }

    // 输出json数据
    w.Json(data, http.StatusOK)
}

// 正则路由控制器，需要配置Router组件(components.router.rules)
// 规则中捕获的参数通过动作函数参数传递，没有则为空字符串.
// eg. "^/reg/eg/(\\w+)/(\\w+)$ => /welcome/regexp-example"
func (w *Welcome) ActionRegexpExample(p1, p2 string) {
    data := map[string]interface{}{"p1": p1, "p2": p2}
    w.Json(data, http.StatusOK)
}

// RESTful动作，url中没有指定动作名，使用请求方法作为动作的名称(需要大写)
// 例如：GET方法请求GET(), POST方法请求POST()
func (w *Welcome) GET() {
   fmt.Println("call in Controller/Welcome.GET")
}

func (w *Welcome) POST() {

}

    
```
7. 创建程序入口(cmd/projectName/main.go)
```go
    package main

    import (
        _ "pgo2-demo/pkg/controller" // 导入控制器

        "github.com/pinguo/pgo2"
    )

    func main() {
        pgo2.Run() // 运行程序
    }
```
8. 编译运行
    ```sh
    make update
    make start
    curl http://127.0.0.1:8000/welcome
    ```
