[TOC]

### 1.Golang内存管理

- #### 内存分配：

  - 是指内存在程序执行过程中分配或者回收存储空间的分配内存的方法

- #### 内存管理包含的三个组件:

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

- 小对象分配：[0kb - 255kb]中对象分配：[256kb - 1mb]
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

> - bool：1 字节（8 位）
> - int8：1 字节（8 位）
> - uint8（或 byte）：1 字节（8 位）
> - int16：2 字节（16 位）
> - uint16：2 字节（16 位）
> - int32（或 rune）：4 字节（32 位）
> - uint32：4 字节（32 位）
> - int64：8 字节（64 位）
> - uint64：8 字节（64 位）
> - float32：4 字节（32 位）
> - float64：8 字节（64 位）
> - complex64：8 字节（64 位）
> - complex128：16 字节（128 位）
> - string：1 字节/字符（默认使用 UTF-8 编码）
>
> 特殊类型:
>
> byte: 等于uint8, 常用来处理ascii字符
>
> rune: 等于int32, 常用来处理unicode字符或者utf8字符
>
> uintptr: 表示一个无符号整数, 用来存放指针, 一般是8字节64位, 但是在一些嵌入式系统中, 可能是2或者4字节

### 4.数组(array)和切片(slice)的区别

> **相同点：**
>
> 只能存储一组相同类型的数据结构
>
> 都是通过下标来访问，并且有容量长度，长度通过 len 获取，容量通过 cap 获取
>
> 数组的容量等于它的长度，因为数组在创建时就固定了长度，无法动态扩容。所以也可以认为数组没有容量这个概念
>
> **区别：**
>
> 数组是定长，访问和复制不能超过数组定义的长度，否则就会下标越界，切片长度和容量可以自动扩容
>
> 数组是值类型，切片是引用类型，每个切片都引用了一个底层数组，切片本身不能存储任何数据，都是这底层数组存储数据，所以修改切片的时候修改的是底层数组中的数据。切片一旦扩容，指向一个新的底层数组，内存地址也就随之改变
>
> 数组之间的copy是值传递, 也就是会在地址中开辟新的地址,并且复制相应的值
>
> 切片之间的copy是引用传递,也就是在copy时, 会引用到相同的底层数组   如下:
>
> ```golang
> a := [3]int{1, 1, 1}
> b := a
> a[1] = 2
> c := []int{1, 1, 1}
> d := c
> c[1] = 2
> 
> fmt.Println(a, b, c, d)  //[1 2 1] [1 1 1] [1 2 1] [1 2 1]
> ```
>
> **初始化:**
>
> ```golang
> // 数组
> var array [2]int
> var array [...]int
> array := [2]int{}
> array := [...]int{}
> array := [2]int{1,2}
> //切片
> var slice []int
> slice := []int{}
> slice := make([]int, 2) // 2是长度
> slice := make([]int, 2, 5)// 2是长度, 5是容量, 主要用于判断是否扩容, 此时底层是一个长度为5的数组
> ```

### 5.range关键字

> `range`关键字是Go语言中的一个迭代器，用于遍历数组、切片、字符串、映射(map)和通道(channel)等数据结构中的元素。

`range`的用法有两种：

> 遍历数组、切片、字符串等具有长度的数据类型时，`range`返回当前元素的索引和值。
>
> 遍历映射(map)和通道(channel)时，`range`返回当前元素的键和值。

1. 遍历数组、切片、字符串

   > 在遍历数组、切片、字符串时，`range`返回两个值，第一个值是元素的下标，第二个值是元素的值。可以使用`_`忽略其中一个值。

   ```golang
   cssCopy code
   arr := [3]int{1, 2, 3}
   for i, v := range arr {
       fmt.Printf("arr[%d]=%d\n", i, v)
   }
   // 输出结果为：
   cssCopy code
   arr[0]=1
   arr[1]=2
   arr[2]=3
   ```

2. 遍历映射(map)

   > 在遍历映射(map)时，`range`返回两个值，第一个值是键(key)，第二个值是键对应的值(value)。

   ```go
   goCopy code
   m := map[string]int{"a": 1, "b": 2}
   for k, v := range m {
       fmt.Printf("%s:%d\n", k, v)
   }
   
   //输出结果为：
   makefileCopy code
   a:1
   b:2
   ```

3. 遍历通道(channel)

   > 当我们使用`range`遍历通道(channel)时，其实是遍历通道中的元素，通道中的元素只有值没有键。具体来说，`range`语句在每次迭代时，都会从通道(channel)中取出一个元素，并将其赋值给迭代变量

   ```go
   ch := make(chan int)
   
   go func() {
       ch <- 1
       ch <- 2
       ch <- 3
       close(ch)
   }()
   
   for v := range ch {
       fmt.Println(v)
   }
   // 1,2,3	
   ```

需要注意的是，当遍历切片或数组时，`range`返回的第一个值是元素的下标，而不是元素本身。如果不需要使用元素的下标，可以使用`_`忽略该值。

此外，当遍历通道(channel)时，如果通道没有关闭，`range`会一直阻塞等待新的数据，如果通道关闭，则会退出循环。遍历通道时，`range`返回的是通道中的值，而不是值的下标。

综上所述，`range`关键字是Go语言中用于遍历数据结构的迭代器，可以方便地遍历数组、切片、字符串、映射和通道等数据结构中的元素。

### 5.通道(channel)

通道（channel）是用来传递数据的一个数据结构。

通道可用于两个 goroutine 之间通过传递一个指定类型的值来同步运行和通讯。操作符 `<-` 用于指定通道的方向，发送或接收。如果未指定方向，则为双向通道。

通道可以用 make 函数来创建，语法如下：

```go
make(chan 元素类型, 缓冲大小)
```

其中，元素类型是通道中要传输的数据的类型，缓冲大小是通道中可以缓存的元素个数，可以是一个正整数或者省略。如果缓冲大小为 0 或者省略，那么通道就是无缓冲的，只有在发送者和接收者同时准备好时，数据才能被传输。如果缓冲大小大于 0，那么通道就是有缓冲的，发送者可以在缓冲未满时发送数据，而接收者可以在缓冲不为空时接收数据。

通道的发送和接收操作都是阻塞的，也就是说，当通道满时，发送者会一直阻塞直到有空间可以发送，当通道为空时，接收者会一直阻塞直到有数据可以接收。如果通道被关闭，那么接收者会立即接收到一个零值，而且无法再向通道发送数据。

通道可以用于实现多个 goroutine 之间的协作和同步。例如，可以用通道来实现 goroutine 之间的任务分配和结果汇总，或者用通道来控制并发访问共享资源的顺序和数量。

通道是 Go 语言中非常重要的并发原语，也是开发并发程序的基础。对于想要学习并发编程的开发者来说，深入理解通道的使用和原理是非常有必要的。

### 6.select关键字

在Go语言中，select关键字是一种用于多路复用通道的机制。它可以让我们同时等待多个通道的操作，并在其中任何一个通道就绪时立即执行相应的操作。

select语句的语法与switch语句类似，但它用于选择通道操作而非值操作。在select语句中，我们可以使用case语句来监听通道的操作，包括读、写和关闭操作。当其中任何一个通道就绪时，select语句就会立即执行相应的操作。

下面是一个简单的例子，演示了select的用法：

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    c1 := make(chan string)
    c2 := make(chan string)

    go func() {
        time.Sleep(time.Second)
        c1 <- "one"
    }()
    go func() {
        time.Sleep(time.Second * 2)
        c2 <- "two"
    }()

    for i := 0; i < 2; i++ {
        select {
        case msg1 := <-c1:
            fmt.Println("received", msg1)
        case msg2 := <-c2:
            fmt.Println("received", msg2)
        }
    }
}
```

在这个程序中，我们创建了两个通道c1和c2，并在两个不同的goroutine中分别向它们发送了数据。在主goroutine中，我们使用select语句监听这两个通道的操作，并在其中任何一个通道就绪时立即执行相应的操作。

当第一个goroutine向c1通道发送了数据后，程序会执行第一个case语句，并输出"received one"。然后程序会继续执行for循环，等待第二个通道的数据。当第二个goroutine向c2通道发送了数据后，程序会执行第二个case语句，并输出"received two"。这样程序就完成了对这两个通道的监听，并顺利退出了。

> **注意，**每次select语句只会执行一个case语句，即使多个通道都就绪了。如果多个case同时就绪，Go语言会在其中随机选择一个执行。如果所有通道都没有就绪，那么select语句就会阻塞，直到其中任何一个通道就绪为止。
>
> 在使用select语句时，我们还可以为每个case语句设置超时时间。如果在指定时间内没有任何一个通道就绪，那么程序就会执行default语句，或者继续等待通道就绪。这种机制可以有效避免因为某个通道长时间未就绪而导致程序无限阻塞的情况。

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	c := make(chan string, 1)
	go func() {
		time.Sleep(2 * time.Second)
		c <- "result"
	}()

	select {
	case res := <-c:
		fmt.Println(res)
	case <-time.After(1 * time.Second):
		fmt.Println("timeout 1")
	default:
		fmt.Println("default case")
	}
}

```

> 在上面的例子中, default语句优先级高于,超市控制语句 但是不具有超时控制功能, 如果每个case都没有就绪, 就会执行default语句

### 8.defer关键字

`defer`是Go语言中的一个关键字，用于延迟函数的执行。当一个函数中存在多个`defer`语句时，它们的执行顺序是倒序的，即后定义的`defer`语句先执行，先定义的`defer`语句后执行。`defer`语句通常用于在函数执行完毕之后释放资源、关闭文件等操作。

下面是一个使用`defer`语句的例子：

```go
func test() {
    defer fmt.Println("world")
    fmt.Println("hello")
}

// 输出:
hello
world
```

`defer`语句不仅可以延迟函数的执行，还可以在函数执行完毕之后获取函数的返回值并进行修改。例如：

```go
goCopy code
func test() (result int) {
    defer func() {
        result += 10
    }()
    return 100
}
```

> **defer的实现是通过一个栈来实现的，每当遇到一个defer语句时，会将要执行的函数推入栈中。当函数执行结束后，会依次从栈中弹出函数并执行，直到栈中的所有函数都被执行完毕。**
>
> **defer无法修改return 语句中的值**

### 9.反射

反射是指程序在运行时可以获取自身信息并操作自身的能力，而不需要预先知道程序的结构。在 Golang 中，使用 reflect 包可以实现反射。

反射的主要作用是在运行时动态地获取一个值的类型信息和值的各种属性（比如字段、方法等），并能够通过反射操作这些属性。反射常用于实现通用的代码库，比如序列化/反序列化、动态调用函数等。

在 Golang 中，使用 reflect 包进行反射主要分为以下几个步骤：

1. 获取变量的 Type 对象：使用 reflect.TypeOf 函数获取变量的 Type 对象，Type 对象包含了变量的类型信息。
2. 获取变量的 Value 对象：使用 reflect.ValueOf 函数获取变量的 Value 对象，Value 对象包含了变量的值。
3. 获取变量的类型信息：通过 Type 对象可以获取变量的类型信息，包括名称、大小、字段、方法等。
4. 获取变量的值信息：通过 Value 对象可以获取变量的值信息，包括整数、浮点数、字符串、结构体、函数等。
5. 修改变量的值信息：使用 Value 对象可以直接修改变量的值信息。

在使用反射时需要注意一些问题：

1. 反射会带来性能损失，因为它需要在运行时动态获取类型信息，而不是编译时静态绑定。
2. 反射只能操作可导出的字段和方法。
3. 对于基本类型和不可寻址的值，无法直接修改其值，需要使用指针来间接修改。
4. 反射并不是万能的，它只能用于处理编译时类型已知的情况。对于动态类型的数据，反射无能为力。

总的来说，反射是 Golang 中一个非常强大的特性，可以实现很多灵活的代码设计。但是反射也有其局限性，需要合理使用，避免对性能造成过大的影响。

### 10.slice切片扩容机制

> 1.5 版本:  new_cap =  2* old_cap
>
> 1.5 -1.17:  new_cap =  1.25 * old_cap
>
> 1.17版本之后:
>
> 容量小于1k  new_cap =  2* old_cap
>
> 容量大于1k  new_cap =  1.25 * old_cap
>
> 1.18版本之后:
>
> new_cap =  1.25 * old_cap + 0.75 * threshold   threshold为 256, 512, 1024, 2048, 4096 按照 old_cap 大小取值
>
> 最终扩容结果:
>
> ![在这里插入图片描述](/Users/alkaid/Desktop/note/assets/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAR29Hb-WcqOWKquWKmw==,size_20,color_FFFFFF,t_70,g_se,x_16.png)

### 11.单引号，双引号，反引号的区别？

单引号，表示byte类型或rune类型，对应 uint8和int32类型，默认是 rune 类型。byte用来强调数据是raw data，而不是数字；而rune用来表示Unicode的code point。

双引号，才是字符串，实际上是字符数组。可以用索引号访问某字节，也可以用len()函数来获取字符串所占的字节长度。

反引号，表示字符串字面量，但不支持任何转义序列。字面量 raw literal string 的意思是，你定义时写的啥样，它就啥样，你有换行，它就换行。你写转义字符，它也就展示转义字符。

### 12.map

无序

> 无序的, map 因扩张⽽重新哈希时，各键值项存储位置都可能会发生改变，顺序自然也没法保证了，所以官方避免大家依赖顺序，直接打乱处理。就是 for range map 在开始处理循环逻辑的时候，就做了随机播种

**map 中删除一个 key，它的内存会释放么**

> 如果删除的元素是值类型，如int，float，bool，string以及数组和struct，map的内存不会自动释放
>
> 如果删除的元素是引用类型，如指针，slice，map，chan等，map的内存会自动释放，但释放的内存是子元素应用类型的内存占用
>
> 将map设置为nil后，内存被回收。
>
> 如果删除的元素是值类型，如int，float，bool，string以及数组和struct，map的内存不会自动释放
>
> 如果删除的元素是引用类型，如指针，slice，map，chan等，map的内存会自动释放，但释放的内存是子元素应用类型的内存占用
>
> 将map设置为nil后，内存被回收。

**并发**

> map本身是线程不安全, 如果需要并发, 需要内置的**sync.Map** , 或读写锁手动控制

**初始化**

> 未初始化的map == nil, 可以读,可以取长度,  不能写, 会报panic

**map key 类型**

> ![image-20230419174926300](/Users/alkaid/Desktop/note/assets/image-20230419174926300.png)
>
> 在 Go 语言中，`slice` 是一种动态数组，可以根据需要动态增加或缩小长度，也可以在中间插入或删除元素。因此，`slice` 的长度和容量可能不同，而且底层数组的地址也可能会发生改变。
>
> 由于 `map` 的键必须是可比较的类型，而 `slice` 的长度、容量和底层数组的地址可能会发生改变，因此 `slice` 不是可比较的类型，也就不能作为 `map` 的键。
>
> `map` 是一种引用类型，它本身是一个指向底层数据结构的指针。由于 `map` 是一个指针类型，因此它们在使用时必须初始化，如果未初始化，它们将默认为 `nil`。
>
> 在 Go 语言中，可比较的类型是指可以使用 `==` 运算符进行比较的类型，例如 `int`、`string`、`bool` 等。对于可比较的类型，可以将它们作为 `map` 的键或结构体的字段。
>
> 但是，由于 `map` 本身是一个指针类型，而且底层数据结构的内部布局是不公开的，因此在编译时无法确定两个 `map` 是否相等。因此，`map` 不是可比较的类型，也就不能将它们作为 `map` 的键或结构体的字段。

**数据结构和扩容机制**

> Go map的底层实现是一个**哈希表**（**数组 + 链表**），使用**拉链法消除哈希冲突**，因此实现map的过程实际上就是实现[哈希表](https://so.csdn.net/so/search?q=哈希表&spm=1001.2101.3001.7020)的过程。
>
> 先来看下go map底层的具体结构：
>
> ```go
> type hmap struct {
> 	count      int            // 元素个数，调用len(map)返回这个值
> 	B          uint8          // bucket数量是2^B, 最多可以放 loadFactor * 2^B 个元素，再多就要扩容了
> 	hash0      uint32         // hash seed
> 	buckets    unsafe.Pointer // 指向bucket数组的指针（存储key val）；大小：2^B 
> 	oldbuckets unsafe.Pointer // 扩容时，buckets 长度是 oldbuckets 的两倍
> 	// ...
> }
> type bmap struct {
> 	topbits  [8]uint8     // 高位哈希值数组
> 	keys     [8]keytype   // 存储key的数组
> 	values   [8]valuetype // 存储val的数组
> 	overflow uintptr      // 指向当前bucket的溢出桶
> 	// 为缓解当存在多个key计算后的哈希值低8位相同的个数大于一个bucket所能存放的数目8个时，且这个map还没达到扩容条件时，做的一种存储设计。
> }
> ```
