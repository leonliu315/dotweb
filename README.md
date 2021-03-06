# DotWeb
简约大方的go Web微型框架

## 1. Install

```
go get -u github.com/devfeel/dotweb
```

## 2. Getting Started
```go
func StartServer() error {
	//初始化DotApp
	app := dotweb.New()
	//设置dotapp日志目录
	app.SetLogPath("/home/logs/wwwroot/")
	//设置路由
	app.HttpServer.Router().GET("/index", func(ctx *dotweb.HttpContext) {
		ctx.WriteString("welcome to my first web!")
	})
	//开始服务
	err := app.StartServer(80)
	return err
}

```
#### 详细示例 - https://github.com/devfeel/dotweb-example

## 3. Features
* 支持静态路由、参数路由
* 路由支持文件/目录服务，支持设置是否允许目录浏览
* 中间件支持(Middleware\HttpModule双重支持)
* Feature支持，可绑定HttpServer全局启用
* 支持STRING/JSON/JSONP/HTML格式输出
* 统一的HTTP错误处理
* 统一的日志处理
* 支持Hijack与websocket
* 内建Cache支持
* 支持接入第三方模板引擎（需实现dotweb.Renderer接口）
* 可配置化，80%模块可通过配置维护

#### Config Example
dotweb.conf
```xml
<?xml version="1.0" encoding="UTF-8"?>
<config>
<app logpath="d:/gotmp/" enabledlog="true" runmode="development"/>
<offline offline="false" offlinetext="server is offline!" offlineurl="" />
<server isrun="true" port="8080" enabledgzip="false" enabledlistdir="false" enabledautohead="true"/>
<session enabled="true" mode="runtime" timeout="20"/>
<middlewares>
    <middleware name="applog" isuse="true" />
</middlewares>
<routers>
    <router method="GET" path="/index" handler="Index" isuse="true">
         <middleware name="urllog" isuse="true" />
    </router>
    <router method="GET" path="/redirect" handler="Redirect" isuse="true"></router>
    <router method="GET" path="/error" handler="DefaultError" isuse="true"></router>
</routers>
<groups>
    <group path="/admin" isuse="true">
        <middleware name="grouplog" isuse="true" />
        <middleware name="simpleauth" isuse="true" />
        <router method="GET" path="/login" handler="Login" isuse="true"></router>
        <router method="GET" path="/logout" handler="Logout" isuse="true"></router>
    </group>
</groups>
</config>　
```
dotweb.json.conf
```json
{
    "app": {
        "logpath": "d:/",
        "enabledlog": false,
        "runmode": "development",
        "pprofport": 8081,
        "enabledpprof": true
    },
    "offline": {
        "offline": false,
        "offlinetext": "",
        "offlineurl": ""
    },
    "server": {
        "enabledlistdir": false,
        "enabledgzip": false,
        "enabledautohead": true,
        "enabledautocors": false,
        "port": 8080
    },
    "session": {
        "enabled": true,
        "mode": "runtime",
        "timeout": 20,
        "serverip": ""
    },
    "routers": [
        {
            "method": "get",
            "path": "/index",
            "HandlerName": "Index",
            "isuse": true
        },
        {
            "method": "get",
            "path": "/redirect",
            "HandlerName": "Redirect",
            "isuse": true
        },
        {
            "method": "get",
            "path": "/error",
            "HandlerName": "DefaultError",
            "isuse": true
        }
    ],
    "Groups": [
        {
            "Path": "/admin",
            "Routers": [
                {
                    "Method": "GET",
                    "Path": "/login",
                    "HandlerName": "Login",
                    "IsUse": true
                },
                {
                    "Method": "GET",
                    "Path": "/logout",
                    "HandlerName": "Logout",
                    "IsUse": true
                }
            ],
            "IsUse": true
        }
    ]
}
```

## 4. Router
#### 1) 常规路由
* 支持GET\POST\HEAD\OPTIONS\PUT\PATCH\DELETE 这几类请求方法
* 支持HiJack\WebSocket\ServerFile三类特殊应用
* 支持Any注册方式，默认兼容GET\POST\HEAD\OPTIONS\PUT\PATCH\DELETE方式
* 支持通过配置开启默认添加HEAD方式
* 支持注册Handler，以启用配置化
* 支持检查请求与指定路由是否匹配
```go
1、Router.GET(path string, handle HttpHandle)
2、Router.POST(path string, handle HttpHandle)
3、Router.HEAD(path string, handle HttpHandle)
4、Router.OPTIONS(path string, handle HttpHandle)
5、Router.PUT(path string, handle HttpHandle)
6、Router.PATCH(path string, handle HttpHandle)
7、Router.DELETE(path string, handle HttpHandle)
8、Router.HiJack(path string, handle HttpHandle)
9、Router.WebSocket(path string, handle HttpHandle)
10、Router.Any(path string, handle HttpHandle)
11、Router.RegisterRoute(routeMethod string, path string, handle HttpHandle)
12、Router.RegisterHandler(name string, handler HttpHandle)
13、Router.GetHandler(name string) (HttpHandle, bool)
14、Router.MatchPath(ctx *HttpContext, routePath string) bool
```
接受两个参数，一个是URI路径，另一个是 HttpHandle 类型，设定匹配到该路径时执行的方法；
#### 2) 静态路由
静态路由语法就是没有任何参数变量，pattern是一个固定的字符串。
```go
package main

import (
    "github.com/devfeel/dotweb"
)

func main() {
    dotapp := dotweb.New()
    dotapp.HttpServer.Router().GET("/hello", func(ctx *dotweb.HttpContext) {
        ctx.WriteString("hello world!")
    })
    dotapp.StartServer(80)
}
```
测试：
curl http://127.0.0.1/hello
#### 3) 参数路由
参数路由以冒号 : 后面跟一个字符串作为参数名称，可以通过 HttpContext的 GetRouterName 方法获取路由参数的值。
```go
package main

import (
    "github.com/devfeel/dotweb"
)

func main() {
    dotapp := dotweb.New()
    dotapp.HttpServer.Router().GET("/hello/:name", func(ctx *dotweb.HttpContext) {
        ctx.WriteString("hello " + ctx.GetRouterName("name"))
    })
    dotapp.HttpServer.Router().GET("/news/:category/:newsid", func(ctx *dotweb.HttpContext) {
    	category := ctx.GetRouterName("category")
	    newsid := ctx.GetRouterName("newsid")
        ctx.WriteString("news info: category=" + category + " newsid=" + newsid)
    })
    dotapp.StartServer(80)
}
```
测试：
<br>curl http://127.0.0.1/hello/devfeel
<br>curl http://127.0.0.1/hello/category1/1


## 5. Binder
* HttpContext.Bind(interface{})
* 支持json、xml、Form数据
* 集成echo的bind实现模块
```go
type UserInfo struct {
		UserName string `form:"user"`
		Sex      int    `form:"sex"`
}

func(ctx *dotweb.HttpContext) TestBind{
        user := new(UserInfo)
        if err := ctx.Bind(user); err != nil {
        	 ctx.WriteString("err => " + err.Error())
        }else{
             ctx.WriteString("TestBind " + fmt.Sprint(user))
        }
}

```

## 6. Middleware
#### RegisterModule - 拦截器
* 支持OnBeginRequest、OnEndRequest两类中间件
* 通过实现HttpModule.OnBeginRequest、HttpModule.OnEndRequest接口实现自定义中间件
* 通过设置HttpContext.End()提前终止请求

#### Middleware - 中间件
* 支持粒度：App、Group、RouterNode
* DotWeb.Use(m ...Middleware)
* Group.Use(m ...Middleware)
* RouterNode.Use(m ...Middleware)
* 启用顺序：App -> Group -> RouterNode，同级别下按Use的引入顺序执行
```go
app.Use(NewAccessFmtLog("app"))

func InitRoute(server *dotweb.HttpServer) {
	server.Router().GET("/", Index)
	server.Router().GET("/use", Index).Use(NewAccessFmtLog("Router-use"))

	g := server.Group("/group").Use(NewAccessFmtLog("group"))
	g.GET("/", Index)
	g.GET("/use", Index).Use(NewAccessFmtLog("group-use"))
}

type AccessFmtLog struct {
	dotweb.BaseMiddlware
	Index string
}

func (m *AccessFmtLog) Handle(ctx *dotweb.HttpContext) error {
	fmt.Println(time.Now(), "[AccessFmtLog ", m.Index, "] begin request -> ", ctx.Request.RequestURI)
	err := m.Next(ctx)
	fmt.Println(time.Now(), "[AccessFmtLog ", m.Index, "] finish request ", err, " -> ", ctx.Request.RequestURI)
	return err
}

func NewAccessFmtLog(index string) *AccessFmtLog {
	return &AccessFmtLog{Index: index}
}
```

## 7. Server Config
#### HttpServer：
* HttpServer.EnabledSession 设置是否开启Session支持，目前支持runtime、redis两种模式，默认不开启
* HttpServer.EnabledGzip 设置是否开启Gzip支持，默认不开启
* HttpServer.EnabledListDir 设置是否启用目录浏览，仅对Router.ServerFile有效，若设置该项，则可以浏览目录文件，默认不开启
* HttpServer.EnabledAutoHEAD 设置是否自动启用Head路由，若设置该项，则会为除Websocket\HEAD外所有路由方式默认添加HEAD路由，默认不开启

#### Run Mode
* 新增development、production模式
* 默认development，通过DotWeb.SetDevelopmentMode\DotWeb.SetProductionMode开启相关模式
* 若设置development模式，未处理异常会输出异常详细信息
* 未来会拓展更多运行模式的配置


## 8. Exception
#### 500错误
* 默认设置: 当发生未处理异常时，会根据RunMode向页面输出默认错误信息或者具体异常信息，并返回 500 错误头
* 自定义: 通过DotServer.SetExceptionHandle(handler *ExceptionHandle)实现自定义异常处理逻辑
```go
type ExceptionHandle func(*HttpContext, interface{})
```

## 9. Session
#### 支持runtime、redis两种
* 默认不开启Session支持
* runtime:基于内存存储实现session模块
* redis:基于Redis存储实现session模块,其中redis key以dotweb:session:xxxxxxxxxxxx组成
```go
//设置session支持
dotapp.HttpServer.SetEnabledSession(true)
//使用runtime模式
dotapp.HttpServer.SetSessionConfig(session.NewDefaultRuntimeConfig())
//使用redis模式
dotapp.HttpServer.SetSessionConfig(session.NewDefaultRedisConfig("127.0.0.1:6379"))
//HttpContext使用
ctx.Session().Set(key, value)
```

## 外部依赖
websocket - golang.org/x/net/websocket
<br>
redis - github.com/garyburd/redigo/redis


## 相关项目
#### <a href="https://github.com/devfeel/tokenserver" target="_blank">TokenServer</a>
项目简介：token服务，提供token一致性服务以及相关的全局ID生成服务等

## 贡献名单
目前已经有几位朋友在为框架一起做努力，我们将在合适的时间向大家展现，谢谢他们的支持！

## 如何联系
QQ群：193409346
