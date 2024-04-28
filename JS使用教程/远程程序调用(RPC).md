# [Remote procedure call (RPC)](https://www.rabbitmq.com/tutorials/tutorial-six-javascript)

## 远程程序调用 (RPC)

### （使用 amqp.node 客户端）

在[第二个教程](./工作队列.md)中，我们学习了如何使用工作队列在多个工人之间分配耗时的任务。

但如果我们需要在远程计算机上运行一个函数并等待结果呢？那就另当别论了。这种模式通常被称为远程过程调用或 RPC。

在本教程中，我们将使用 RabbitMQ 构建一个 RPC 系统：一个客户端和一个可扩展的 RPC 服务器。由于我们没有任何值得分发的耗时任务，因此我们将创建一个返回斐波那契数字的虚拟 RPC 服务。

> **关于 RPC 的一点说明**
> 虽然 RPC 是计算机中一种相当常见的模式，但它经常受到批评。当程序员不知道函数调用是本地调用还是慢速 RPC(slow RPC) 时，问题就出现了。
>
> 考虑到这一点，请考虑以下建议:
>
> -   确保哪个函数调用是本地调用，哪个是远程调用，一目了然。
> -   为你的系统写一些说明。明确组件之间的依赖关系。
> -   处理错误情况。当 RPC 服务器长时间停机时，客户端应如何反应？
>     如有疑问，请避免使用 RPC。如果可以，应该使用异步流水线--将结果异步推送到下一个计算阶段，而不是类似 RPC 的阻塞。

### 回调队列

一般来说，通过 RabbitMQ 进行 RPC 非常简单。客户端发送请求消息，服务器回复响应消息。为了接收响应，我们需要随请求发送一个 "回调(callback)"队列地址。我们可以使用默认的交换(exchange)。让我们试试看：

```javascript
channel.assertQueue("", {
    exclusive: true,
});

channel.sendToQueue("rpc_queue", Buffer.from("10"), {
    replyTo: queue_name,
});
```

> **消息属性**
> AMQP 0-9-1 协议预定义了一组 14 个属性，与消息一起使用。除以下属性外，大多数属性都很少使用：
>
> -   `persistent:` 将消息标记为持久（值为 true）或暂存（false）。您可能还记得[第二个教程](./工作队列.md)中的这个属性。
> -   `content_type:`: 用于描述编码的 mime 类型。例如，对于常用的 JSON 编码，最好将此属性设置为：application/json。
> -   `reply_to:` 常用于命名回调队列。
> -   `correlation_id:`: 用于将 RPC 响应与请求关联起来。
