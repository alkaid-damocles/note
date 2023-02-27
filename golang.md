### 1.Golang内存管理

- 内存分配：

  - 是指内存在程序执行过程中分配或者回收存储空间的分配内存的方法

- 内存管理包含的三个组件:

  - 应用程序
  - 内存分配器
  - 垃圾收集器（GC）

  #### 内存分配器：

  最重要指标： 速度， 空间利用率

- 常见内存分配方式

  - 线性分配器
    - 内存中维护一个指针， 分配内存时移动指针
    - 优点：简单高效
    - 缺点：之前分配的内存无法被回收，需要配合相应的回收算法， 如标记压缩，复制回收， 分代回收
  - 空闲链表分配器： 
    - golang内存分配基于此

- **TCMalloc**

  - Thread-cache malloc 线程缓存 malloc
  -  TCMalloc 是 Google 对 C 的 [malloc](https://so.csdn.net/so/search?q=malloc&spm=1001.2101.3001.7020)() 和 C++ 的 operator new 的自定义实现，用于在我们的 C 和 C++ 代码中进行内存分配。 TCMalloc 是一种快速、多线程的 malloc 实现。
  - TCMalloc的[官方介绍](https://gperftools.github.io/gperftools/tcmalloc.html)中将内存分配称为 object allocation（对象分配）


​	按照所分配对象内存的大小，TCMalloc 将内存分配分为三类：

- 小对象分配：[0kb - 255kb]
- 中对象分配：[256kb - 1mb]
- 大对象分配：[1Mb - +∞]

然后介绍几个重要概念：

- page， TCMalloc 将整个虚拟内存划分成一个个page  1个page 8kb
- Span， 由连续的n个page组成的
- PegeHeap, 以span为单位向系统申请内存， 可能只有一个page， 也可能有多个



### 2.make和new的区别

共同点： 分配内存

不同点： make给引用类型分配内存并初始化， new给string int object 分配内存， 并返回指针

### 3.值类型和引用类型

值类型： 整形， 浮点， 字符串， 数组， 布尔， struct

引用类型： channel， map， slice， 指针   

