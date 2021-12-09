# Buffer

## 概念

> 对于node端需要处理网络协议，操作数据库，处理图片，接收上传文件等，需要处理大量二进制数据，字符串不能胜任，buffer就出现了

- Buffer是一个类似Array的对象,可访问length、下标访问元素、修改下标对应的值等

```js
// 会将数值控制在0~255区间中，通过+- 256，小数下取整
const buff = new Buffer(100);
buff[1] = 100;
buff[2] = -100;
buff[3] = 300;
buff[4] = 5.334;
buff[1] // 100
buff[2] // -100 + 256 = 156
buff[3] // 300 - 256 = 44
buff[4] // 5.334 | 0 = 5
```

- 真正的内存是在node的c++层面提供，js层面使用它（Buffer、SlowBuffer）



## buffer内存分配

> buffer对象内存分配不是在V8堆内存中，而是在node的c++层面实现内存申请的，要是每需要一点内存，就向操作系统申请，会造成大量的内存申请消耗。

对于node中，以8KB为界限区分Buffer是大对象还是小对象，将会使用不同的方式分配，都是以slab动态内存管理机制。

- 小对象分配 <= 8KB

  - 将pool作为中间处理对象，当调用buffer构造函数时，会去检查是否存在pool对象，要是不存在将开辟一个空的8KB的pool，用来存储buffer对象

  - 当slab单元剩余空间不足时，将生成新的slab

    例如：第一次生成buffer是1字节，第二次生成是8kb，计算时发现第二次不能和第一份buffer合在一个slab单元中，将会生成新的一个slab单元

- 大对象分配 > 8KB

  - 直接分配SlowBuffer对象作为slab单元，SlowBuffer是C++中定义的

## buffer会存在的问题

1、buffer不支持中文编码GBK/GB2312转换

2、buffer拼接

​		对于英文字符来说，字符串和buffer可以自由转换因此直接使用 + 进行组合拼接，但对于中文来说，是占3个字节，当3个字节正好处于buffer夹缝处就会出现混乱打印出❓。

- 解决方法

使用Buffer.concat将多个buffer和为一个，在进行转化即可



## buffer性能

> 除了网络I/O,文件IO,对于网络传输，都是使用buffer

因此直接传递字符串的话会经过两步骤

1、转换成buffer类型

2、发送buffer

因此此时只需先转换成buffer后在传递，会加快一倍的QPS(每秒查询次数)
