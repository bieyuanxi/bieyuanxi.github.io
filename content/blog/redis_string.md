+++
title = "Redis String"
date = "2026-01-28"
description = "Redis string can be anything!"
draft = false

[taxonomies]
tags = ["redis", "valkey", "string"]
+++

# 参考资料
- [valkey commands: string](https://valkey.io/commands/#string)


# 可用命令
String Operations on the String data type
- APPEND Appends a string to the value of a key. Creates the key if it doesn't exist.
- DECR Decrements the integer value of a key by one. Uses 0 as initial value if the key doesn't exist.
- DECRBY Decrements a number from the integer value of a key. Uses 0 as initial value if the key doesn't exist.
- DELIFEQ Delete key if value matches string.
- GET Returns the string value of a key.
- GETDEL Returns the string value of a key after deleting the key.
- GETEX Returns the string value of a key after setting its expiration time.
- GETRANGE Returns a substring of the string stored at a key.
- GETSET Returns the previous string value of a key after setting it to a new value.
- INCR Increments the integer value of a key by one. Uses 0 as initial value if the key doesn't exist.
- INCRBY Increments the integer value of a key by a number. Uses 0 as initial value if the key doesn't exist.
- INCRBYFLOAT Increment the floating point value of a key by a number. Uses 0 as initial value if the key doesn't exist.
- LCS Finds the longest common substring.
- MGET Atomically returns the string values of one or more keys.
- MSET Atomically creates or modifies the string values of one or more keys.
- MSETNX Atomically modifies the string values of one or more keys only when all keys don't exist.
- PSETEX Sets both string value and expiration time in milliseconds of a key. The key is created if it doesn't exist.
- SET Sets the string value of a key, ignoring its type. The key is created if it doesn't exist.
- SETEX Sets the string value and expiration time of a key. Creates the key if it doesn't exist.
- SETNX Set the string value of a key only when the key doesn't exist.
- SETRANGE Overwrites a part of a string value with another by an offset. Creates the key if it doesn't exist.
- STRLEN Returns the length of a string value.
- SUBSTR Returns a substring from a string value.



## SET
将键 `key` 的值设为指定字符串 `value`。若该键已存有值，则无论原数据类型如何，都会被直接覆盖。默认情况下，当 `SET` 命令执行成功时，该键原本关联的生存时间（`TTL`）也会被清除。

命令变种：`SETEX`，`SETNX`，`PSETEX`，`GETSET`。

### 语法
```sh
SET key value [ NX | XX | IFEQ comparison-value ] [ GET ] [ EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | KEEPTTL ] 
```

SET 命令支持一组可修改其行为的选项，具体如下：
- EX seconds —— 设置键的过期时间，单位为秒（需传入正整数）。
- PX milliseconds —— 设置键的过期时间，单位为毫秒（需传入正整数）。
- EXAT timestamp-seconds —— 设置键的过期 Unix 时间戳，单位为秒（需传入正整数）。
- PXAT timestamp-milliseconds —— 设置键的过期 Unix 时间戳，单位为毫秒（需传入正整数）。
- NX —— 仅当键不存在时，才执行设置操作。
- XX —— 仅当键已存在时，才执行设置操作。
- IFEQ comparison-value —— 仅当键的现有值与指定的比对值完全匹配时，才执行设置操作。若键存储的值并非字符串类型，则返回错误并终止 SET 命令的执行。
- KEEPTTL —— 保留该键原本关联的生存时间（TTL），不会因设置新值而清空过期时间。
- GET —— 返回键原本存储的旧字符串值；若键不存在，则返回 nil。若键存储的值并非字符串(String)类型（例如存的是列表、哈希），则返回错误并终止 SET 命令的执行。


### 用例
```sh
127.0.0.1:6379> SET mykey "Hello"
OK
127.0.0.1:6379> GET mykey
"Hello"

# Set a value and an expiry time.
127.0.0.1:6379> SET anotherkey "will expire in a minute" EX 60
OK

# Conditionally set a value.
127.0.0.1:6379> SET foo "Initial Value"
OK
127.0.0.1:6379> GET foo
"Initial Value"
127.0.0.1:6379> SET foo "New Value" IFEQ "Initial Value"
OK
127.0.0.1:6379> GET foo
"New Value"

# 分布式锁，key是资源名称，value是锁持有者随机生成的token，释放锁时客户端应该检查确认是自己生成的锁再释放
localhost:6379> SET res_lock a-random-token NX
OK

```

## SETEX
```sh
SETEX key seconds value 
等价于
SET key value EX seconds
```

## SETNX
**SET** if **N**ot e**X**ists.
```sh
SETNX key value 
```

### 用例
```sh
127.0.0.1:6379> SETNX mykey "Hello"
(integer) 1
127.0.0.1:6379> SETNX mykey "World"
(integer) 0
127.0.0.1:6379> GET mykey
"Hello"
```

## SETRANGE
覆盖键key中存储的字符串的指定部分：从指定偏移量开始，以待写入值value的完整长度为覆盖范围完成写入。若指定的偏移量大于键对应字符串的当前长度，会自动用空字节（0 字节） 填充字符串至该偏移量的位置，以匹配写入起始位置；不存在的键会被视作空字符串，因此本命令会自动扩展该键对应的字符串长度，确保能在指定偏移量处写入value。

注意：可设置的最大偏移量为 2^29 - 1（即 536870911），这是因为 Valkey 中字符串类型的大小上限为 512MB。若需要写入的数据超出该长度限制，可通过多个键分片存储来实现。

> Patterns: 借助 SETRANGE 及其同类型的 GETRANGE 命令，可以将 Valkey 字符串当作支持 O (1) 随机访问的线性数组来使用。在诸多实际业务场景中，这都是一种极快且高效的存储方式。

```sh
SETRANGE key offset value 
```

### 用例
```sh
127.0.0.1:6379> SET key1 "Hello World"
OK
127.0.0.1:6379> SETRANGE key1 6 "Valkey"
(integer) 12
127.0.0.1:6379> GET key1
"Hello Valkey"

# zero padding
127.0.0.1:6379> SETRANGE key2 6 "Valkey"
(integer) 12
127.0.0.1:6379> GET key2
"\x00\x00\x00\x00\x00\x00Valkey"
```

## MSET
将指定的多个键分别设为对应的取值。MSET 会像普通的 SET 命令一样，用新值覆盖键的原有值；若你不想覆盖已有值，可使用 MSETNX 命令。
MSET 是**原子操作**，因此所有指定的键会被一次性完成赋值。客户端不会出现「部分键已更新、其余键仍保持原值」的中间可见状态。
```sh
MSET key value [ key value ... ] 
```


### 用例
```sh
127.0.0.1:6379> MSET key1 "Hello" key2 "World"
OK
127.0.0.1:6379> GET key1
"Hello"
127.0.0.1:6379> GET key2
"World"
```

## GET
获取key对应的值。如果键不存在，返回`nil`。如果值不是string类型，返回错误。GET 是纯**读命令**，不会修改键的任何元数据（包括过期时间）。

### 用例
```sh
127.0.0.1:6379> GET nonexisting
(nil)
127.0.0.1:6379> SET mykey "Hello"
OK
127.0.0.1:6379> GET mykey
"Hello"
```


## GETEX
获取键 key 对应的取值，并可按需为该键设置过期时间。GETEX 命令与 GET 命令功能相近，区别在于它属于**写命令**，即使仅使用 GETEX key（不带任何选项），也会触发 Redis 对键的访问时间更新（影响 LRU 淘汰策略），且会同步到主从节点 / AOF 日志。

### 语法
```sh
GETEX key [ EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | PERSIST ] 
```

GETEX 命令支持一组可修改其行为的选项，具体如下：
- PERSIST —— 移除该键关联的生存时间（TTL），使其变为永久有效。

### 用例
```sh
127.0.0.1:6379> SET mykey "Hello"
OK
127.0.0.1:6379> GETEX mykey
"Hello"
127.0.0.1:6379> TTL mykey
(integer) -1
127.0.0.1:6379> GETEX mykey EX 60
"Hello"
127.0.0.1:6379> TTL mykey
(integer) 60
```

## GETDEL
获取键key的值并删除该键。此命令与GET命令功能相近，区别在于：执行成功时会同时删除该键（当且仅当该键的值类型为字符串类型时，才会执行删除操作）。
### 语法
```sh
GETDEL key 
```
### 用例
```sh
127.0.0.1:6379> SET mykey "Hello"
OK
127.0.0.1:6379> GETDEL mykey
"Hello"
127.0.0.1:6379> GET mykey
(nil)
```


## GETRANGE
返回存储在指定键`key`中的字符串值的子串，子串范围由偏移量 `start` 和 `end` 决定（两个偏移量对应的字符均包含在内）。可以使用负数偏移量，表示从字符串的末尾开始计算位置。例如，`-1` 代表最后一个字符，`-2` 代表倒数第二个字符，以此类推。该函数会通过将结果范围限制在字符串的实际长度内，来处理超出范围的请求。

### 语法
```sh
GETRANGE key start end 
```

### 时间复杂度
`O(N)`，`N`是返回子串的大小，子串较小时可以近似认为是`O(1)`。

### 用例
```sh
127.0.0.1:6379> SET mykey "This is a string"
OK
127.0.0.1:6379> GETRANGE mykey 0 3
"This"
127.0.0.1:6379> GETRANGE mykey -3 -1
"ing"
127.0.0.1:6379> GETRANGE mykey 0 -1
"This is a string"
127.0.0.1:6379> GETRANGE mykey 10 100
"string"
# 右边界在左边界左边，返回空子串
localhost:6379> GETRANGE mykey 5 0
""
```



## GETSET
以**原子操作**的方式将键 key 的值设为 value，并返回该键原本存储的值。若键已存在但存储的并非字符串类型，则返回错误。当 SET 命令执行成功时，该键原本关联的所有生存时间（TTL）都会被清除。
### 语法
```sh
GETSET key value 
# 等价于
SET key value GET
```

### 用例
GETSET 可与 INCR 配合使用，实现**原子化重置**的计数功能。例如：某进程可在每次有事件发生时，对键 mycounter 执行 INCR 命令（累加计数）；但我们需要不定期地原子性地获取计数器当前值，并将其重置为 0，这一操作可通过执行 GETSET mycounter "0" 实现。
```sh
127.0.0.1:6379> INCR mycounter
(integer) 1
127.0.0.1:6379> GETSET mycounter "0"
"1"
127.0.0.1:6379> GET mycounter
"0"
```



## MGET
返回所有指定键对应的值。对于非字符串类型或不存在的键，均返回特殊值 `nil`。正因如此，该操作永远不会执行失败。
```sh
MGET key [ key ... ] 
```


### 用例
```sh
127.0.0.1:6379> SET key1 "Hello"
OK
127.0.0.1:6379> SET key2 "World"
OK
127.0.0.1:6379> MGET key1 key2 nonexisting
1) "Hello"
2) "World"
3) (nil)
```



## DELIFEQ
> Since: 9.0.0
```sh
DELIFEQ key value 
```
仅当键的值与指定字符串完全匹配时，才删除该键。
此命令通常用于分布式系统中安全释放锁，确保只有锁的持有者（通过值来标识）能够删除该锁。
若键存在，且存储的字符串值与传入值完全一致，则删除该键并返回1；否则，键保持不变并返回0。

### 用例
```sh
127.0.0.1:6379> SET mykey abc123
OK
127.0.0.1:6379> DELIFEQ mykey abc123
(integer) 1
127.0.0.1:6379> DELIFEQ mykey abc123
(integer) 0
```



## INCR
将存储在键 key 中的数字值自增 1。若该键不存在，则会先将其值设为 0，再执行自增操作。若键中存储的值类型错误（非字符串），或存储的字符串无法解析为整数，则返回错误。此操作仅支持**64 位有符号整数**。


注意：这是一个字符串操作，因为 Valkey 并不具备专用的整数类型。执行此操作时，键中存储的字符串会被解析为10 进制的 64 位有符号整数。
Valkey 会以整数的原生存储格式来保存整数值，因此对于那些实际存储整数的字符串值来说，并不会产生存储整数字符串表示形式的额外开销。


### 用例
```sh
127.0.0.1:6379> SET mykey "10"
OK
127.0.0.1:6379> INCR mykey
(integer) 11
127.0.0.1:6379> GET mykey
"11"
```


## INCRBY
```sh
INCRBY key increment 
```

## DECR
```sh
DECR key 
```

## DECRBY
```sh
DECRBY key decrement 
```