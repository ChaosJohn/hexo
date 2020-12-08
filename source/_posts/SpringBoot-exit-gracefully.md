---
title: SpringBoot 优雅退出
date: 2020-12-08 23:11:54
tags: [jvm, java, SpringBoot, kotlin]
thumbnail: https://image.blog.chaosjohn.com/SpringBoot-exit-gracefully/banner.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/SpringBoot-exit-gracefully.html)

## 背景
公司某项目的后端技术栈采用的是 `SpringBoot` + `Kotlin`，具体细节本文不作展开。

业务中，用户在我们平台上购买产品，我们通过实时请求供应商的交易API，在商户余额（即我们在供应商那边的储值）里扣除对应的金额后，才会将产品的“源文件”返回，进而交付给用户。

项目迭代中我们需要不断的部署新的版本上线，最初的做法很暴力，构建新的 `flatjar` 运行起来，然后直接结束掉 `旧的 java 进程`。

然后就发现问题了：某次购买行为中，交易API的请求发送出去了，还没等待请求返回，进程就被杀死了，造成储值余额被扣了，但是“源文件”并没有拿到，而供应商的API又存在延迟，即交易API处理成功，但是通过订单查询API却查不到。

所以，如何才能在 java 进程被杀死的时候，做完 `善后工作` 再退出呢？

## 解决
一般我们杀死进程，都是给进程发送信号 `Signal`：
- `SIGINT`
- `SIGTERM`

这里我们用到两个类：
- `sun.misc.Signal`，代表信号
- `sun.misc.SignalHandler`，用来处理进程接收到的信号

同时，我们设计以下全局变量：
- `var killSignalReceived = false` // 用来表示是否接收到终止信号
- `val jobSet = mutableSetOf<String>()` // 表示需要在结束之前等待完成的任务

所以交易API的请求处理，我们将改成：
```
if (!killSignalReceived) { // 只有为 `false`，才进行交易处理
  synchronized(jobSet) {
      jobSet.add(jobTitle) // 处理之前，将当前处理任务存入 `jobSet`
  }
  // Todo: 具体实现请求交易API的处理
  synchronized(jobSet) {
      jobSet.remove(jobTitle) // 处理结束，将当前处理任务从 `jobSet` 中移除
  }
}
```

在程序主函数中，新增：
```
val killHandler = SignalHandler {
    logger.error("intercept signal of ${it.toJson()}")
    killSignalReceived = true
    while (jobSet.isNotEmpty()) {
        logger.error("Background Jobs: \n\t\t" + jobSet.joinToString("\n\t\t") + "\n")
        Thread.sleep(1000)
    }
    exitProcess(0)
}
Signal.handle(Signal("INT"), killHandler)
Signal.handle(Signal("TERM"), killHandler)
```
当 java 进程接收到 `SIGINT` 和 `SIGTERM` 信号后：
- 将 `killSignalReceived` 置为 `true`
- 循环检查 `jobSet` 是否为空
  - 不为空，打印当前未结束的任务列表，等待1s后再次检查
  - 为空，程序退出

如果需要忽略 `善后工作` 强行退出，给进程发送 `SIGKILL` 即可：
- `kill -KILL pid`
- `kill -9 pid`

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)