---
date: 2018-09-29T07:07:00+08:00
title: RST_STREAM帧
weight: 354
menu:
  main:
    parent: "frame-definition"
---


## 帧格式

`RST_STREAM`帧（类型= 0x3）允许立即终止一个流。 发送RST_STREAM以请求取消流或指示发生了错误情况。

```
 +---------------------------------------------------------------+
 |                        Error Code (32)                        |
 +---------------------------------------------------------------+
```

RST_STREAM帧包含标识错误代码的单个无符号的32位整数。错误代码指示流被终止的原因。 

## 标志

RST_STREAM帧不定义任何标志。

## 说明

RST_STREAM帧完全终止流并使其进入“close”状态。在流上接收到RST_STREAM之后，接收者不得为该流发送额外的帧，但PRIORITY除外。但是，在发送RST_STREAM之后，发送端点务必准备好接收和处理在RST_STREAM到达之前可能已经由对端在流上发送的帧。

RST_STREAM帧必须与一个流相关联。如果接收到RST_STREAM帧且流标识符为0x0，则接收方必须将其视为`PROTOCOL_ERROR`类型的连接错误。

RST_STREAM帧不得在“idle”状态下发送流。如果接收到标识空闲流的RST_STREAM帧，接收者必须将其视为类型为`PROTOCOL_ERROR`的连接错误。

长度不是4个字节的RST_STREAM帧必须被视为`FRAME_SIZE_ERROR`类型的连接错误。