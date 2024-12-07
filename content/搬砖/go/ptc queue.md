---
title: ptc queue
draft: true
tags:
  - ptc
---
1. queue.Service  只是 给 facades.Queue 初始化了queue application
2. &providers2.QueueServiceProvider{} 注册了 []queue.Step{} 在消费者绑定消费 signature 与 消费函数时使用
3. 生产者往队列中 push 消息 是通过 `facades.Queue.Step(step, []queue.Arg{}).DispatchASync()`， `facades.Queue.Step()` 生成 `&support.Task{}` 进行 push （<font color="#f79646">问题 每次 dispatch 都要 new 一次 server</font>）
4. 消费者 `facades.Queue.Worker(queue.Args{Connection: "redis"}).Run()`, ``facades.Queue.Worker()` 生成 `&support.Worker{}` run 时 通过 `service.NewServiceGroup` 生成多个 server 进行消费 ( <font color="#f79646">问题 1.生产者与消费者 的 queue name 必须一致，否则无法消费， 2.是否可以生成一个server，通过不同worker name 指定queue name 来消费不同queue 任务</font> )
5.  Machinery 在 newServer 的 conf  中有一个 DefaultQueue，指定默认队列，即 server.NewWorker 和 server.SendTask 如果不置顶queue都会默认放在这个队列处理。woker与 task 分别单独制定queue 的方式
```go
server.NewCustomQueueWorker() 第三个参数指定特定处理的queue
server.SendTask(&tasks.Signature{  RoutingKey 来指定这个 task 需要放在那个队列
  RoutingKey: "queue",
  ...
})
```
6.  Machinery 有几个概念 `server`  `task` `worker` `broker` `Backend`  其中 server 是总的架构 
```
server.SendTask  将 task 放到 broker 等待消费
server.NewWorker 从 broker 中取出 task 处理，并将处理结果，状态放在Backend

其中 broker 与 Backend 在 NewServer 已经配置，支持的 broker Backend 有redis 等
```