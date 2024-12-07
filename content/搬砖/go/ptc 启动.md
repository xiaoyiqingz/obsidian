---
title: ptc 启动
draft: true
tags:
  - ptc
---
总结：
```go
main() {
	BootXxx()
}

BootXxx() {
	//1. 通过foundation.init() 初始化 facades.Config，并使用viper.Viper加载配置文件config
	app := foundation.Application{} 

	//3. 通过app.registerConfiguredServiceProviders()与app.bootConfiguredServiceProviders() 初始化 facades.Config['app']['providers'] 到 facades.*, 不使用 app.Boot() 是因为 其中有 app.bootPtc() 只支持默认命令
	app.Load()

	// 2. Boot() 本身是空的，通过 config.init() 生成 facades.Config['app']['providers'] 
	config.Boot() 

	// 4. runPtc or runQueue or runWorker 执行业务逻辑
}
```

### 一、 ptc.go (main.go)
1. main  -> bootstrap.Boot()
2.  bootstrap.Boot() 分别 调用 app ：= foundation.Application{} 与 config.Boot() 导致 foundation.init() 与 config.init() 调用

### 二、foundation.init()
```go
func init() {
	//Create a new application instance.
	app := Application{}

	app.registerBaseServiceProviders() // 初始化 facades.Config
	app.bootBaseServiceProviders() // 暂时没用
}
```
1.  `app.registerBaseServiceProviders()`  目前 base providers 只有 `config.ServiceProvider`  register 时 :
```go
facades.Config = Application struct {
	vip *viper.Viper
}
并使用 viper.Viper 初始化的config文件，读取 config数据
```
2.  app.bootBaseServiceProviders() `config.ServiceProvider.Boot()` 什么也没做

### 三、config.init()
```go
config := facades.Config
	
config.Add("app", map[string]interface{}{
	"name": config.Env("Name", "ptc"),

	"env": config.Env("Mode", "production"),

	"debug": config.Env("Debug", false),

	"providers": []contracts.ServiceProvider{
		&log.ServiceProvider{},
		&console.ServiceProvider{},
		&foundation.PtcServiceProvider{},
		&queue.ServiceProvider{},
		&providers2.AppServiceProvider{},
		&providers2.ConsoleServiceProvider{},
		&providers2.QueueServiceProvider{},
		&cache.ServiceProvider{},
	},
})
```

获取到 上一步中初始化的 `facades.Config`， 向其中增加了一些  `app` 的值，其中 `app.providers` 会在 后面 `app.registerConfiguredServiceProviders()` 与 `app.bootConfiguredServiceProviders()` 中初始化各个providers

### 四、bootstrap.Boot() 执行

```go
func (app *Application) Boot() {
	app.registerConfiguredServiceProviders()
	app.bootConfiguredServiceProviders()

	app.bootPtc()
}
```
将上一步增加到`facades.Config['app']['providers']` 的providers 初始化 到各个 facades.* 中

	 facades.* 是全局变量，挂了各个prvoders的实例
### 五、app.bootPtc()
`app.bootPtc()` 执行命令
