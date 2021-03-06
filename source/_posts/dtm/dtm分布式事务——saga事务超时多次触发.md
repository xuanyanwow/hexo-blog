---
title: dtm分布式事务——saga事务超时多次触发
tags:
  - 分布式事务
categories:
  - 开发层
date: 2022-3-1 21:53:50
---

# 问题背景

在使用saga分布式事务的时候，接口执行时间过长，导致 `dtm-server多次重发` 请求到节点

在dtm文档中，关于超时的说明如下

> saga属于长事务，因此持续的时间跨度很大，可能是100ms到1天，因此saga没有默认的超时时间。
> 
> dtm支持saga事务单独指定超时时间，到了超时时间，全局事务就会回滚。
> 
> `saga.TimeoutToFail = 1800`


这里的Timeout指的是 整个事务的超时时间，而我们这里出现的问题是`单个节点(子事务)`的超时时间导致重发

# 排查

dtm-server的报错日志

```
{"level":"info","ts":"2022-03-01T21:37:15.276+0800","caller":"dtmsvr/trans_status.go:27","msg":"TouchCronTime for: {\"ID\":0,\"create_time\":\"2022-03-01T21:30:53.3232344+08:00\",\"update_time\":\"2022-03-01T21:37:15.2743237+08:00\",\"gid\":\"ghXUgV63AnsBwtxa6LtY5o\",\"trans_type\":\"saga\",\"steps\":[{\"action\":\"http://127.0.0.1:8888/order_create\",\"compensate\":\"http://127.0.0.1:8888/order_create_compensate\"},{\"action\":\"http://127.0.0.1:8889/goods_inventory\",\"compensate\":\"http://127.0.0.1:8889/goods_inventory_compensate\"}],\"payloads\":[\"{\\\"goods_id\\\":1,\\\"number\\\":3}\",\"{\\\"goods_id\\\":1,\\\"number\\\":3}\"],\"status\":\"submitted\",\"protocol\":\"http\",\"next_cron_interval\":20,\"next_cron_time\":\"2022-03-01T21:37:35.2743237+08:00\",\"wait_result\":true}"}
{"level":"error","ts":"2022-03-01T21:37:15.277+0800","caller":"dtmsvr/trans_type_saga.go:136","msg":"exec branch error: http/grpc result should be specified as in:\nhttps://dtm.pub/summary/arch.html#http\nunkown result will be retried: Post \"http://127.0.0.1:8889/goods_inventory?branch_id=02&gid=ghXUgV63AnsBwtxa6LtY5o&op=action&trans_type=saga\": dial tcp 127.0.0.1:8889: connectex: No connection could be made because the target machine actively refused it.","stacktrace":"github.com/dtm-labs/dtm/dtmsvr.(*transSagaProcessor).ProcessOnce.func3.1\n\t/github/workspace/dtmsvr/trans_type_saga.go:136\ngithub.com/dtm-labs/dtm/dtmsvr.(*transSagaProcessor).ProcessOnce.func3\n\t/github/workspace/dtmsvr/trans_type_saga.go:140"}
```

通过报错和查看源码得知 还有一个`RequestTimeout`需要配置

# 解决

RequestTimeout默认时间是3s，我们可以根据需要调大一些，也可以不调整这里  在dtm-server下次重发的时候通过`接口幂等控制`返回处理完成的结果，完成整个事务的调用
