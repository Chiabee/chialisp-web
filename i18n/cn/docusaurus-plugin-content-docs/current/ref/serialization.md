---
id: serialization
title: CLVM 参考手册
sidebar_label: 序列化
---

> CLVM Reference Manual

## 序列化

CLVM 序列化格式严格遵循程序树的内存表示。

反过来，这与完全编译的 CLVM 程序的文本格式非常相似。

在最底层，CLVM 中只有两种类型的 Object。

* 对（也称为 Cons Pair/Cons Box/Cons Cell）—— 一对两个值。 通常，第一个值是一个原子并包含信息，而第二个值通常是指向另一对的指针，或一个 `nil` 原子。 列表是从这些对构建的。
* 原子 —— 保存数据的值，表示为具有静态长度的字节数组。 空的零长度原子用于表示 `nil` 值。

每个 CLVM 对象由一系列一个或多个字节表示。 每个字节都属于恰好一个 CLVM 对象的表示。 也就是说，一个字节中的任何位都不会被多个 CLVM 对象共享。

### 值

CLVM 中的每个值都是一个无类型的字节序列。 在运行的虚拟机中，值有一个属性长度，包含字节数。 序列化格式中保留了相同的概念。 但是，必须将值与字节流中的 cons 框区分开来，因此使用了转义方案。 这种转义意味着使用超过 7 位的序列化值与内存中的表示具有不同的表示。

Nil 有一个不与任何用户定义值共享的预定义值。

<details>
<summary>原文参考</summary>

- ## Serialization

The CLVM serialization format closely follows the in-memory representation of the program tree.

This in turn, closely resembles the text format of a fully compiled CLVM program.

At the lowest level, there are only two types of Object in the CLVM.

* Pair (also called Cons Pair/Cons Box/Cons Cell) - A pair of two values. Usually, the first value is an atom and contains information, whereas the second value is usually either a pointer to another pair, or a `nil` atom. Lists are built from these pairs.
* Atom - A value that holds data, which is represented as a byte array with the static length. Empty, zero-length atoms are used to represent `nil` value.

Each CLVM Object is represented by a series of one or more bytes. Each byte belongs to the representation of exactly one CLVM Object. That is, no bits in a byte are shared by multiple CLVM objects.


- ### Values

Each value in the CLVM is an untyped sequence of bytes. In the running virtual machine, values have a property length, containing the number of bytes. The same concept is preserved in the serialization format. However, values must be distinguished from cons boxes in the byte stream, so an escaping scheme is used. This escaping means that serialized values using more than 7 bits have a different representation than the in memory representation.

Nil has a single predefined value which is not shared with any user-defined value.

</details>

### 编码

可以用 7 位表示的值被编码为具有该值的单个字节。 较大的序列化值表示为字节序列，这些字节对大小进行编码，然后是值字节。

size 前缀的编码方案如下：

第一个序列化字节的值决定了 size 字节的数量（1 到 6，包括第一个字节）。 然后大小决定了表示值的字节数（0 到 0x400000000-1 字节长）

**size** 是包含该值的字节数组的长度，为整数。在下表中，我们使用 **s** 来描述构成编码大小值的字节，s[0] 是 **size** 的最低有效字节。 大小字节首先编码为 MSB。

```
object is cons box:   0xFF
value == nil:         0x80
value <= 0x7F:        value

size < 0x40:          0x80 | s[0], value
size < 0x2000:        0xC0 | s[1], s[0], value
size < 0x100000:      0xE0 | s[2], s[1], s[0], value
size < 0x8000000:     0xF0 | s[3], s[2], s[1], s[0], value
size < 0x400000000:   0xF8 | s[4], s[3], s[2], s[1], s[0], value
```

在下表中，标记为 x 的位包含值数组的长度，以字节为单位。

### 编码大小字节布局

Size 字节 | 最小值长度 | 最大值长度 | 字节 1 | 字节 2 | 字节 3 | 字节 4 | 字节 5
---|---|---|---|---|---|---|---
1|0x00|0x3F|1xxxxxxx
2|0x40|0x1FFF|11xxxxxx|xxxxxxxx
3|0x2000|0xFFFFF|111xxxxx|xxxxxxxx|xxxxxxxx
4|0x100000|0x7FFFFFF|1111xxxx|xxxxxxxx|xxxxxxxx|xxxxxxxx
5|0x8000000|0x3FFFFFFFF|11111xxx|xxxxxxxx|xxxxxxxx|xxxxxxxx|xxxxxxxx

<details>
<summary>原文参考</summary>

- ### Encoding

Values which can be represented in 7 bits are encoded as a single byte with that value. Larger serialized values are represented as a sequence of bytes that encode the size, and then the value bytes.

The encoding scheme for the size prefix is as follows:

The value of the first serialized byte determines the number of size bytes (1 to 6, including the first byte). The size then determines the number of bytes denoting the value (0 to 0x400000000-1 bytes long)

**size** is the length of the byte array containing the value, as an integer.
In the table below, we use **s** to describe the bytes that make up the encoded size value, s[0] being the least significant byte of **size**. The size bytes are encoded MSB first.

```
object is cons box:   0xFF
value == nil:         0x80
value <= 0x7F:        value

size < 0x40:          0x80 | s[0], value
size < 0x2000:        0xC0 | s[1], s[0], value
size < 0x100000:      0xE0 | s[2], s[1], s[0], value
size < 0x8000000:     0xF0 | s[3], s[2], s[1], s[0], value
size < 0x400000000:   0xF8 | s[4], s[3], s[2], s[1], s[0], value
```

In the table below, the bits marked x contain the length of the value array, in bytes.

- ### Encoded Size bytes layout

Size bytes | Min Value Len | Max Value Length | Byte 1 | Byte 2 | Byte 3 | Byte 4 | Byte 5
---|---|---|---|---|---|---|---
1|0x00|0x3F|1xxxxxxx
2|0x40|0x1FFF|11xxxxxx|xxxxxxxx
3|0x2000|0xFFFFF|111xxxxx|xxxxxxxx|xxxxxxxx
4|0x100000|0x7FFFFFF|1111xxxx|xxxxxxxx|xxxxxxxx|xxxxxxxx
5|0x8000000|0x3FFFFFFFF|11111xxx|xxxxxxxx|xxxxxxxx|xxxxxxxx|xxxxxxxx

</details>

### 解码

解码方案如下：

c[0] 是序列化 CLVM 对象的第一个字节。

**size** 是值字节数组中包含的字节数
**s** 是包含二进制补码整数大小的字节的字节数组，即值数组中的字节数。
**值** 是描述 CLVM 对象本身的字节范围

c[0] 可以包含整个值，也可以是大小标头的一部分。低于 0x80 的值没有大小头字节。

0x00-0x7f：一字节的文字值。 c[0] 包含值。 `size = 1; value = c[0]`

0x80-0xbf：值从字节 c[1] 开始，大小在c[0]的低 6 位`size = (c[0] & 0x3F); value = c[1] .. c[size]`

0xc0-0xdf：值从 c[2] 开始； c[0] 的低 5 位是大小 `size = (c[0] & 0x1F) .. c[1]; value = c[2] .. c[size+1]`

0xe0-0xef：值从 c[3] 开始； c[0] 的低 4 位是大小 `size = (c[0] & 0x0F) .. c[2]; value = c[3] .. c[size+2]`

0xf0-0xf7：值从 c[4] 开始； c[0] 的低 3 位是大小 `size = (c[0] & 0x07) .. c[3]; value = c[4] .. c[size+3]`

0xf7-0xfb：值从 c[5] 开始； c[0] 的低 2 位是大小 `size = (c[0] & 0x03) .. c[4]; value = c[5] .. c[size+4]`

序列化格式中不允许大小为 0x400000000 或更大的原子。

举个例子：

```
c = [ 0x84 0x33 0x22 0x11 0x00 ]
c[0] = 0x84
size = (0x84 & 0x3F) = 4
value = [ 33 22 11 00 ]
```

在上面的例子中，值的长度是 **4**，我们只需要 c[0] 字节的后3位来编码长度，所以大小完全由第一个序列化字节来描述。 编码原子的总长度为 5 个字节。

请注意，对于大于 0x7F 的值，表示长度的序列化值的字节与实际值字节不相交。

让我们考虑一些特殊情况。

```
value(0x80) = 81 80
```

c[0] 是 `0x81`。由于 c[0] 介于 `0x7F` 和 `0xC0` 之间，我们知道只有一个大小字节 c[0]，其值包含在后面的字节中，从 在 c[1]。 值数组的总大小为 `size` = `c[0] & 0x3F` = `0x1`。 因此，完整值包含在单个后续字节中。

```
value(0x81) = 81 81
value(0x82) = 81 82
value(0xFF) = 81 ff
```

请注意，在表示值的字节中允许使用特殊字节 0xFF。当它是解码 CLVM 对象的第一个字节时，0xFF 表示一个 cons 框，但它也可能出现在值的序列化字节中。

一个 2 字节的值

```
value(0x01FF) = 82 01 ff
```

<details>
<summary>原文参考</summary>

- ### Decoding

The decoding scheme is as follows:

c[0] is the first byte of the serialized CLVM object.

**size** is the number of bytes contained in the value byte array
**s** is the byte array containing the bytes of the two's complement integer size, the number of bytes in the value array.
**value** is the span of bytes describing the CLVM Object itself

c[0] can contain the entire value, or it can be part of the size header.
Values below 0x80 do not have size header bytes.

0x00-0x7f: A literal one byte value. c[0] contains the value.
           `size = 1; value = c[0]`

0x80-0xbf: The value starts at the byte c[1], and size is in the lower 6 bits of c[0]
           `size = (c[0] & 0x3F); value = c[1] .. c[size]`

0xc0-0xdf: The value starts at c[2]; the lower 5 bits of c[0] are the high bits of size
           `size = (c[0] & 0x1F) .. c[1]; value = c[2] .. c[size+1]`

0xe0-0xef: The value starts at c[3]; the lower 4 bits of c[0] are the high bits of size
           `size = (c[0] & 0x0F) .. c[2]; value = c[3] .. c[size+2]`

0xf0-0xf7: The value starts at c[4]; the lower 3 bits of c[0] are the high bits of size
           `size = (c[0] & 0x07) .. c[3]; value = c[4] .. c[size+3]`

0xf7-0xfb: The value starts at c[5]; the lower 2 bits of c[0] are the high bits of size
           `size = (c[0] & 0x03) .. c[4]; value = c[5] .. c[size+4]`

Atoms of size 0x400000000 or greater are disallowed in the serialization format.

As an example:
```
c = [ 0x84 0x33 0x22 0x11 0x00 ]
c[0] = 0x84
size = (0x84 & 0x3F) = 4
value = [ 33 22 11 00 ]
```

In the above example, the length of the value is **4**, and we only needed the bottom 3 bits of the c[0] byte to encode the length, so the size is completely described by the first serialized byte. The total length of the encoded atom is 5 bytes.

Note that for values greater than 0x7F, the bytes of the serialized value representing the length are disjoint with the actual value bytes.

Let us consider some special cases.

```
value(0x80) = 81 80
```
c[0] is `0x81`.Since c[0] is between `0x7F` and `0xC0`, we know that there is only one size byte, c[0], and the value is contained in the following bytes, starting at c[1]. The total size of the value array is
`size` = `c[0] & 0x3F` = `0x1`. So, the full value is contained in the single following byte.

```
value(0x81) = 81 81
value(0x82) = 81 82
value(0xFF) = 81 ff
```

Note that the special byte 0xFF is allowed within the bytes representing a value.
0xFF denotes a cons box when it is the first byte of a decoded CLVM object, but it may also occur within the serialized bytes of a value.

A 2 byte value
```
value(0x01FF) = 82 01 ff
```

</details>

### 列表

在大多数 LISP 实现中，列表是主要的高级数据结构。传统上，LISP 从 cons 框构建列表，这是一种双单元格数据结构，可以将其视为具有左右两个字段的结构，每个字段包含一个值或一个（指向）另一个 cons 单元格的（指针）。

因为 cons 单元是构建列表的低级数据结构，所以 LISP 列表只是列表，因为列表可以从树中实现。从二叉树中的 cons 单元构建的 lisp 列表。

序列化的 cons 单元由字节 0xFF 表示，然后是其单元中的对象，从左到右。值由可变长度字节对齐编码方案表示，如上所述。Nil 被选为零长度对象，其中由字节 0x80 表示。

由于列表表示为一系列 cons 框，因此字节 0xff 在序列化格式中经常出现。

在 FF 介绍人字节之后，接下来的两个值分别描述了左右 cons 盒子中的内容。

为清楚起见，第一个示例使用单字节值。

列表 (1 2 3) 将被编码为：

```
ff 01 ff 02 ff 03 80
```

这可以理解为：

`(a cons box containing 01` and `a cons box (containing 02` and `a cons box (containing 03 and nil)))`

或者，可以将其视为如下所示的二叉树：

```
      [ ]
     /   \
    1    [ ]
         / \
        2  [ ]
           / \
          3  nil
```

或者，作为一个看起来像这样的记忆细胞链：

```
(a)[ 1, ->b ]   (b)[ 2, ->c ]   (c)[ 3, nil ]
```

在哪里
   * (a) 表示“位置 a 和 a+1 处的存储单元的内容（cons 盒子 a）”
   * ->b 表示“指向 cons 盒子 (b) 的指针”
   * `(a)`, `(b)`, `(c)` 是 cons 单元的任意标签，在 CLVM 中不存在

因为上面的列表只包含一层嵌套，单个`0x80`字节就足以终止列表。 请注意下面示例中的两个 `0x80` 字节。

```
opc '(1 (2 3))'
ff 01 ff ff 02 ff 03 80 80
```

CLVM 程序中可以有很多 cons 单元，因此 0xFF 在序列化程序中很常见。每个正确终止的列表将有一个序列化的 nil (0x80)。 Nil 也可能出现在程序的其他地方。cons 盒子通常比列表多，因此 0xFF 比 0x80 出现的频率更高。

<details>
<summary>原文参考</summary>

- ### Lists

Lists are the primary high-level data structure in most LISP implementations. Traditionally, a LISP builds lists from cons boxes, a two-celled data structure that can be thought of as a struct with two fields, left and right, each of which contains either a value, or a (pointer to) another cons cell.

Because the cons cell is the low level data structure that lists are built from, LISP lists are only lists by way of the fact that lists can be implemented from trees. A lisp list built from cons cells in a binary tree.

Serialized cons cells are represented by the byte 0xFF, followed by the objects in its cells, left then right.
Values are represented by a variable length byte-aligned encoding scheme, described above.
Nil is chosen to be the zero-length object, which is represented by the byte 0x80.

Because lists are represented as a sequence of cons boxes, the byte 0xff occurs frequently in the serialization format.

After the FF introducer byte, the next two values describe what is in the left and right cons boxes, respectively.

The first examples use single byte values for clarity.

The list (1 2 3) will be encoded as:
```
ff 01 ff 02 ff 03 80
```
This can be read as:

`(a cons box containing 01` and `a cons box (containing 02` and `a cons box (containing 03 and nil)))`

Alternatively, it could be viewed as a binary tree that looks like this:

```
      [ ]
     /   \
    1    [ ]
         / \
        2  [ ]
           / \
          3  nil
```

Or, as a chain of memory cells that look like this:
```
(a)[ 1, ->b ]   (b)[ 2, ->c ]   (c)[ 3, nil ]
```

Where
  * (a) means "The contents of the memory cells at position a and a+1 (cons box a)"
  * ->b means "a pointer to cons box (b)"
  * `(a)`, `(b)`, `(c)` are arbitrary labels for the cons cells, and do not exist in the CLVM

Because the above list contains only one level of nesting, a single `0x80` byte is sufficient to terminate the list. Note the two `0x80` bytes in the example below.

```
opc '(1 (2 3))'
ff 01 ff ff 02 ff 03 80 80
```

There can be many cons cells in a CLVM program, so 0xFF will be common in the serialized program. There will be one serialized nil (0x80) per properly terminated list. Nil may also occur at other places in the program. There are usually more cons boxes than lists, so 0xFF occurs more frequently than 0x80.

</details>
