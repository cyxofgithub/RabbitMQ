[# Topic 交换器(exchange)](https://www.rabbitmq.com/tutorials/tutorial-five-javascript)

## Topic 交换器

在上一教程中，我们改进了日志系统。我们不再使用只能进行虚假广播的`fanout`exchange，而是使用了`direct`exchange，并获得了选择性接收日志的能力。

虽然使用`direct`交换改进了我们的系统，但它仍有局限性--不能根据多个标准进行路由选择。

在日志系统中，我们可能不仅要根据日志的重要性订阅日志，还要根据日志的来源订阅日志。您可能从 syslog unix 工具中了解到这一概念，它根据重要性（info/warn/crit......）和设施（auth/cron/kern......）筛选日志。

这将为我们提供很大的灵活性--我们可能只想监听来自 "cron "的关键错误，但也想监听来自 "kern "的所有日志。

要在日志系统中实现这一点，我们需要了解更复杂的`topic`交换器。
