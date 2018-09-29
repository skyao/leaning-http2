---
date: 2018-09-29T07:07:00+08:00
title: PUSH_PROMISE帧
weight: 356
menu:
  main:
    parent: "frame-definition"
---


## 帧格式

`PUSH_PROMISE`帧（type = 0x5）**用于在发送者打算初始化流之前通知对端**。 PUSH_PROMISE帧包括端点计划创建的流的无符号31位标识符以及为流提供附加上下文的一组header。 

```
 +---------------+
 |Pad Length? (8)|
 +-+-------------+-----------------------------------------------+
 |R|                  Promised Stream ID (31)                    |
 +-+-----------------------------+-------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```

有如下字段：

- Pad Length（填充长度）：8位字段，以八位位组为单位，包含帧填充的长度。 该字段仅在PADDED标志置位时才存在。
- R：一个保留位。
- Promised Stream ID：一个无符号的31位整数，用于标识由PUSH_PROMISE保留的流。 承诺流标识符必须是发送方发送的下一个流的有效选择。
- Header Block Fragment（header块片段）：包含请求头部字段的header块片段。
- Padding：填充字节。

## 标志

`PUSH_PROMISE`帧有如下标志：

- END_HEADERS（0x4）：置位时，位2指示该帧包含整个header块，并且没有任何CONTINUATION帧。 没有设置END_HEADERS标志的PUSH_PROMISE帧必须跟着同一个流的CONTINUATION帧。 接收方必须将接收到的任何其他类型的帧或不同流上的帧视为类型为PROTOCOL_ERROR的连接错误。
- PADDED（0x8）：置位时，位3指示Pad Length字段及其描述的填充符存在。

## 说明

PUSH_PROMISE帧必须只能在处于“open”或“half-closed (remote)”状态的已对端初始化的流上发送。PUSH_PROMISE帧的流标识符指示与其关联的流。如果流标识符字段指定值0x0，则接收方必须响应类型为PROTOCOL_ERROR的连接错误。

承诺的流不需要按照承诺的顺序使用。 PUSH_PROMISE仅保留流标识符供以后使用。

如果对端端点的SETTINGS_ENABLE_PUSH设置被设置为0，则不应发送PUSH_PROMISE。已设置此设置并已收到确认的端点务必将PUSH_PROMISE帧的接收视为类型为PROTOCOL_ERROR的连接错误。

PUSH_PROMISE帧的接受者可以选择拒绝承诺的流，通过发送带有承诺的流标识符的RST_STREAM返回给PUSH_PROMISE的发送者来。

