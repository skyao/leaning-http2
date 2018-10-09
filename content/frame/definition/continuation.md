---
date: 2018-09-29T07:07:00+08:00
title: CONTINUATION帧
weight: 359
menu:
  main:
    parent: "frame-definition"
keywords:
- CONTINUATION帧
description : "详细介绍HTTP/2的CONTINUATION帧"
---


## 帧格式

CONTINUATION帧 (type=0x9) 被用于继续发送header块片段的序列。只要相同流上的前导帧是没有设置END_HEADERS标记的HEADERS，PUSH_PROMISE，或CONTINUATION帧，就可以发送任意数量的CONTINUATION帧。

```
 +---------------------------------------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
```

CONTINUATION帧有效载荷包含一个header块片段。

## 标志

CONTINUATION帧定义了以下标志：

- END_HEADERS（0x4）：置位时，位2表示该帧结束header块。
  - 如果未设置END_HEADERS位，则该帧必须紧跟着另一个CONTINUATION帧。接收方必须将接收到的任何其他类型的帧或不同流上的帧视为PROTOCOL_ERROR类型的连接错误。

## 说明

CONTINUATION帧必须与流相关联。如果收到其流标识符字段为0x0的CONTINUATION帧，接收方必须响应PROTOCOL_ERROR类型的连接错误。

CONTINUATION帧必须在前面加上HEADERS，PUSH_PROMISE或CONTINUATION帧，而不要设置END_HEADERS标志。观察到违反此规则的接受者必须响应PROTOCOL_ERROR类型的连接错误。