---
date: 2018-09-29T07:07:00+08:00
title: 流的状态
weight: 401
menu:
  main:
    parent: "stream"
---

## 生命周期

流的生命周期如下所示：

```
                             +--------+
                     send PP |        | recv PP
                    ,--------|  idle  |--------.
                   /         |        |         \
                  v          +--------+          v
           +----------+          |           +----------+
           |          |          | send H /  |          |
    ,------| reserved |          | recv H    | reserved |------.
    |      | (local)  |          |           | (remote) |      |
    |      +----------+          v           +----------+      |
    |          |             +--------+             |          |
    |          |     recv ES |        | send ES     |          |
    |   send H |     ,-------|  open  |-------.     | recv H   |
    |          |    /        |        |        \    |          |
    |          v   v         +--------+         v   v          |
    |      +----------+          |           +----------+      |
    |      |   half   |          |           |   half   |      |
    |      |  closed  |          | send R /  |  closed  |      |
    |      | (remote) |          | recv R    | (local)  |      |
    |      +----------+          |           +----------+      |
    |           |                |                 |           |
    |           | send ES /      |       recv ES / |           |
    |           | send R /       v        send R / |           |
    |           | recv R     +--------+   recv R   |           |
    | send R /  `----------->|        |<-----------'  send R / |
    | recv R                 | closed |               recv R   |
    `----------------------->|        |<----------------------'
                             +--------+

       send:   endpoint sends this frame
       recv:   endpoint receives this frame

       H:  HEADERS frame (with implied CONTINUATIONs)
       PP: PUSH_PROMISE frame (with implied CONTINUATIONs)
       ES: END_STREAM flag
       R:  RST_STREAM frame
```

请注意，此图显示了流状态转换以及会影响这些转换的帧和标志。在这方面，`CONTINUATION`帧不会导致状态转换; 它们实际上是它们跟随的`HEADERS`或`PUSH_PROMISE`的一部分。出于状态转换的目的，`END_STREAM`标志作为单独的事件处理到承载它的帧; 具有`END_STREAM`标志设置的`HEADERS`帧可以导致两个状态转换。 

两个端点都有流状态的主观视图，当帧在传输过程中可能会有所不同。端点不协调创建流; 它们是由任一端点单方面创建的。在发送`RST_STREAM`之后，状态不匹配的负面影响仅限于“closed”状态，在关闭后帧可能会被接收一段时间。 

## 流的状态

数据流具有以下状态：

### idle

所有的流都以 idle状态开始。

以下转换在此状态下有效：

- 发送或接收`HEADERS`帧会导致流成为“open”。 同样 `HEADERS`帧也可以使流立即变成“half -closed”。

- 在另一个流上发送`PUSH_PROMISE`帧将保留标识的空闲流为以后使用。保留流的流状态转换为“reserved (local)”。

- 在另一个流上接收`PUSH_PROMISE`帧将保留标识的空闲流为以后使用。保留流的流状态转换为“reserved (remote)”。

- 请注意，`PUSH_PROMISE`帧不在空闲流上发送，而是在`Promised Stream ID`字段中引用新保留的流。 

在idle状态下，在流上接收除`HEADERS`或`PRIORITY`之外的任何帧必须被视为类型为`PROTOCOL_ERROR`的连接错误。

### reserved (local)

处于“reserved (local)”状态的流是通过发送`PUSH_PROMISE`帧来承诺的流。 `PUSH_PROMISE`帧通过将流与远程对等端发起的开放流关联来保留idle流。

在这种状态下，只有以下转换是可能的：

- 端点可以发送`HEADERS`帧。 这会导致流开启为“half-closed(remote)”状态。
- 任一端点都可以发送`RST_STREAM`帧，以使流成为“ closed”。 这将释放流预留。

在此状态下，端点不得发送除`HEADERS，RST_STREAM 或 PRIORITY` 之外的任何类型的帧。

可以在此状态下接收 PRIORITY 或`WINDOW_UPDATE`帧。在此状态的流上接收除`RST_STREAM，PRIORITY 或  WINDOW_UPDATE` 以外的任何类型的帧必须视为`PROTOCOL_ERROR`类型的连接错误。

### reserved (remote)

处于“reserved (remote)”状态的流已被远程对端保留。

在这种状态下，只有以下转换是可能的：

- 接收`HEADERS`帧会导致数据流转换为“half-closed（local）”。

- 任一端点都可以发送`RST_STREAM`帧，以使流成为“closed”。 这将释放流预留。

端点可以在这种状态下发送`PRIORITY`帧来重新设置保留流的优先级。在此状态下，端点不得发送除`RST_STREAM，WINDOW_UPDATE或PRIORITY`以外的任何类型的帧。

在这种状态下，在流上接收除`HEADERS，RST_STREAM或PRIORITY`以外的任何类型的帧必须被视为类型为`PROTOCOL_ERROR`的连接错误。

### open

处于“open”状态的流可以被两个对端用来发送任何类型的帧。 在这种状态下，发送对等方遵守流级别的流量控制限制。

在这种状态下，任何端点都可以发送带有`END_STREAM`标志的帧，这会导致流转换到“half-closed”状态之一。 发送`END_STREAM`标志的端点导致流状态变为“half-close(local)”; 接收`END_STREAM`标志的端点将导致流状态变为“half-close(remote)”。每个端点都可以从此状态发送`RST_STREAM`帧，使其立即转换为“closed”。

### half-closed (local)

处于“half-closed(local)”状态的流不能用于发送`WINDOW_UPDATE，PRIORITY 和 RST_STREAM` 以外的帧。 

当接收到包含 END_STREAM 标志的帧或任一对端发送 RST_STREAM 帧时，流将从此状态转换为“closed”状态。

端点可以在这种状态下接收任何类型的帧。 使用 WINDOW_UPDATE 帧提供流量控制信用是继续接收流量控制帧所必需的。 在这种状态下，接收方可以忽略 WINDOW_UPDATE 帧，因为带有 END_STREAM 标志的帧可能会在短时间内到达。

在此状态下收到的优先级帧用于重新确定依赖于已识别流的流的优先级。

### half-closed (remote)

“half-closed(remote)”的流不再被对端用来发送帧。 在这种状态下，端点不再需要维护接收器流量控制窗口。 

如果端点接收到除`WINDOW_UPDATE，PRIORITY或RST_STREAM`之外的其他帧，则对于处于此状态的流，它必须响应类型为STREAM_CLOSED的流错误。

 “half-closed(remote)”的流可以被端点用来发送任何类型的帧。在这种状态下，端点会继续观察流级别的流量控制限制。

 通过发送一个包含 END_STREAM 标志的帧或任一对端发送 RST_STREAM 帧，流可以从此状态转换为“closed”状态。

### closed

“closed”状态是终结状态。

**端点不得在封闭流上发送PRIORITY以外的帧**。在接收到RST_STREAM后接收除PRIORITY以外的任何帧的端点必须将其视为类型`STREAM_CLOSED`的流错误。类似地，在接收到设置了 END_STREAM 标志的帧后接收任何帧的端点必须将其视为`STREAM_CLOSED`类型的连接错误，除非该帧是允许的，如下所述。

`WINDOW_UPDATE 或 RST_STREAM` 帧可以在包含 END_STREAM 标志的 DATA 或 HEADERS 帧发送后的短时间内在此状态下接收。在远程节点接收并处理 RST_STREAM 或带有 END_STREAM 标志的帧之前，它可能会发送这些类型的帧。终端必须忽略在这种状态下接收到的 WINDOW_UPDATE 或 RST_STREAM 帧，当然终端也可以选择将在发送 END_STREAM  相当长时间之后到达的帧作为类型为 PROTOCOL_ERROR 的连接错误。

PRIORITY帧可以在closed流上发送，以设置依赖于这个closed流的其他流的优先级。端点应该处理 PRIORITY 帧，当然如果流已经从依赖关系树中移除，则它们可以被忽略。

如果由于发送 RST_STREAM 帧而达到此状态，那么接收 RST_STREAM 的对端可能已经在流上发送 - 或已排队发送 - 帧而无法取消。端点必须在发送 RST_STREAM 帧后忽略它在closed流上接收到的帧。端点可以选择限制它忽略帧的时间段，并将在此时间后到达的帧视为错误。

在发送 RST_STREAM 之后接收的适用于流量控制的帧（即DATA）被计数到连接流量控制窗口。即使这些帧可能被忽略，因为它们是在发送者收到 RST_STREAM 之前发送的，发送者会认为这些帧是针对流量控制窗口进行计数的。

端点在发送 RST_STREAM 后可能会收到一个 PUSH_PROMISE 帧。即使关联的流已重置，PUSH_PROMISE也会使流成为“reserved”。因此，**需要 RST_STREAM 来关闭不需要的承诺流。**

### 补充

这份文档中其它地方没有更多特别说明的，实现应该将描述中没有明确表示允许的帧的状态的接收作为一个类型为PROTOCOL_ERROR的连接错误。注意，任何状态下的流都可以发送和接收PRIORITY。类型未知的帧被忽略。











