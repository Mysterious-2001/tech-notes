简单来说，Go 语言中的context（上下文）主要用来解决 “如何优雅地控制那些已经派发出去的协程（Goroutine）”。 在 Go 的并发模型中，你很容易启动成百上千个 Goroutine。如果没有一种机制来通知它们停止，这些协程可能会在后台死循环，导致内存泄漏。
context就是那根连通所有协程的“信号线”。

context的三大核心作用 
1. 退出通知（Cancellation） 这是最常用的功能。当一个请求因为超时或客户端主动关闭而被取消时，你需要通知所有相关的子协程立即停止工作，释放资源。场景：用户请求一个搜索接口，由于数据库响应太慢，用户直接关掉了网页。此时，搜索接口启动的数据库查询、缓存读取等协程都应该立即停掉。
2. 超时控制（Timeout / Deadline） 你可以给一个任务设定“保质期”。如果规定时间内没跑完，context会自动发出取消信号。场景：调用第三方支付接口，规定 3 秒内必须返回。超过 3 秒，context自动触发，你的程序就不再傻等了。 
3. 传递元数据（Value） 它可以在同一个请求的多个协程之间传递一些“全局信息”。场景：链路追踪 ID（Trace ID）、用户登录的 Token 信息等。这些信息不需要作为函数的显式参数一个个传递，而是挂在context里顺着调用链路往下走。context的树状结构context的派生是有层级关系的。它就像一棵树：
	• 根节点：通常是
context.Background()
	• 子节点：通过WithCancel
、WithTimeout等方法创建。 关键特性：如果父节点被取消，所有由它派生出来的子节点都会被取消。 常见的 API 及其用法函数用途适用场景
context.Background()
创建一个空的根 Context程序的入口、main 函数、测试代码
context.WithCancel()
返回一个可手动取消的 Context当任务完成或出错需要手动停止时
context.WithTimeout()

返回一个带超时时间的 Context限制 API 调用时间、数据库查询限制

context.WithValue()

携带 K-V 键值对传递 Request ID、用户身份标识 避坑指南（最佳实践） 1. 不要把 Context 存放在结构体里：应该把它作为函数的第一个参数，通常命名为

ctx

。 2. 不要传递可选参数：

WithValue

应该只用来传递“请求范围”的元数据，而不是用来偷懒传业务参数。 3. 必须调用 Cancel：如果你用了

WithCancel

或

WithTimeout

，一定要记得执行

defer cancel()

，否则可能导致协程或定时器资源泄露。