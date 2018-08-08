# HEADERS帧

HEADERS帧(type=0x1)用于打开一个流，此外还携带一个header块片段。HEADERS帧可以在状态为”idle”，”reserved (local)”，”open”，或”half-closed (remote)”的流上发送。

## 帧格式

```
 +---------------+
 |Pad Length? (8)|
 +-+-------------+-----------------------------------------------+
 |E|                 Stream Dependency? (31)                     |
 +-+-------------+-----------------------------------------------+
 |  Weight? (8)  |
 +-+-------------+-----------------------------------------------+
 |                   Header Block Fragment (*)                 ...
 +---------------------------------------------------------------+
 |                           Padding (*)                       ...
 +---------------------------------------------------------------+
```

HEADERS帧具有如下的字段：

- Pad Length/填充长度

  8位的字段，包含以字节为单位的帧的填充长度。只有在PADDED标记设置时这个字段才出现。

- E

  单bit的标记，指示流依赖是独占的。这个字段只有在PRIORITY标记设置时才会出现。

- Stream Dependency/流依赖：31位的流标识符，标识这个流依赖的流。这个字段只有在PRIORITY标记设置时才会出现。

- Weight/权重

  无符号8位整型值，表示流的优先级权重。值的范围为1到256。这个字段只有在PRIORITY标记设置时才会出现。

- header块片段/Header Block Fragment

  一个header块片段。

- 填充/Padding

  填充字节。 

## 标记

HEADERS帧定义了如下的标记：

- END_STREAM (0x1)

  当设置时，位0表示这个header块是终端将是被标识的流发送的最后一个块。

  HEADERS帧可以携带 END_STREAM 标记，标明流的结束。然而，在相同的流上，一个设置了 END_STREAM 标记的HEADERS帧后面可以跟着 CONTINUATION 帧。逻辑上来说，CONTINUATION 帧是 HEADERS 帧的一部分。

- END_HEADERS (0x4)