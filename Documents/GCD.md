# Swift多线程
* [GCD](#GCD)     
  * [概述](#概述)      
  * [任务](#任务)     
    * [同步sync](#同步sync)     
    * [异步async](#异步async)     
  * [队列](#队列)     
    * [串行队列](#串行队列)     
    * [并行队列](#并行队列)     
  * [队列与优先级](#队列与优先级)     
    * [基础定义结构体](#基础定义结构体)     
    * [创建两个队列具有相同的优先级](#创建两个队列具有相同的优先级)     
    * [创建两个不同优先级](#创建两个不同优先级)     
  * [延时执行](#延时执行)     
    * [基础定义结构体](#基础定义结构体)     
    * [wallDeadline和deadline](#wallDeadline和deadline)     
    * [DispatchGroup](#DispatchGroup)     
    * [DispatchWorkItem](#DispatchWorkItem)     
    * [DispatchSemaphore](#DispatchSemaphore)     
  * [总结](#总结)     

# GCD
Grand Central Dispatch
## 概述
* 为多核的并行运算提出的解决方案，会自动的合理的利用更多的cpu内核，会自动的管理线程的生命周期（创建，调度，销毁）；
* 使用c语言，但用到了block闭包，相当于面向对象，封装，所以用起来方便；

## 任务
操作，即要干什么，就是一段代码，在GCD中就是一个block，同步执行和异步执行两种执行方式，区别是：是否会阻塞当前线程，直到block中的任务执行完毕。

### 同步sync
会阻塞当前线程，并等待block中的任务完成，然后当前线程才会继续向下运行

```swift
let queue = DispatchQueue(label: "com.iii17-grace.GCD")

queue.sync {
    for i in 0...5{
        print(i)
    }
}

for i in 6...9{
    print(i)
}

//输出：0123456789
```


### 异步async
不会阻塞当前线程，当前线程会继续向下运行

```swift
let queue = DispatchQueue(label: "com.iii17-grace.GCD")

queue.async {
    for i in 0...5{
        print(i)
    }
}

for i in 10...14{
    print(i)
}
//输出：10 0 11 1 12 2 13 3 14 4 5

```

## 队列
用于存放任务
### 串行队列
GCD会按FIFO取出来一个，执行一个，取下一个，执行下一个… 按任务添加的顺序，依次执行；

```swift
//异步async， 因为同步还是按顺序一个个执行

//不指定队列类型，默认就是串行
let queue = DispatchQueue(label: "com.iii17-grace.GCD")

queue.async {
    for i in 0...5{
        print(i)
    }
}

queue.async {
    for i in 6...9{
        print(i)
    }
}

queue.async {
    for i in 10...14{
        print(i)
    }
}

//输出： 1 2 3 4 5 6 7 8 9 10 11 12 13 14
```

### 并行队列
GCD会按FIFO取出来，但取出来之后是放到别的线程，然后再取出来一个又放到另一个线程，由于取的速度很快，忽略不计，所以看起来是所有任务一起执行的。【GCD会根据系统资源控制并行的数量】

```swift
//参数attributes另一个值是initiallyInactive表示任务不会自动执行，需要程序员手动触发: queue.activate，然后走串行，如果不设置就是默认任务创建完就自动执行
let queue = DispatchQueue(label: "com.iii17-grace.GCD", attributes: .concurrent)

//如果想并行且有触发
//let queue = DispatchQueue(label: "com.iii17-grace.GCD", attributes: [.concurrent, .initiallyInactive])


queue.async {
    for i in 0...5{
        print(i)
    }
}

queue.async {
    for i in 6...9{
        print(i)
    }
}

queue.async {
    for i in 10...14{
        print(i)
    }
}

//输出：0 6 10 7 1 8 11 9 2 12 3 13 4 14 5
```

## 队列与优先级
GCD内采用DispatchQoS结构体，如果没有指定QoS，会使用default

### 基础定义结构体
主队列默认使用拥有最高优先级，即userInteractive，所以慎用这一优先级，否则极有可能会影响用户体验。
一些不需要用户感知的操作，例如缓存等，使用utility即可
```swift
//以下等级由高到低
public struct DispatchQoS : Equatable {

     public static let userInteractive: DispatchQoS //用户交互级别，需要在极快时间内完成的，例如UI的显示
     
     public static let userInitiated: DispatchQoS  //用户发起，需要在很快时间内完成的，例如用户的点击事件、以及用户的手势
     。
     public static let `default`: DispatchQoS  //系统默认的优先级，
     
     public static let utility: DispatchQoS   //实用级别，不需要很快完成的任务
     
     public static let background: DispatchQoS  //用户无法感知，比较耗时的一些操作

     public static let unspecified: DispatchQoS
}
```

### 创建两个队列具有相同的优先级
```swift
let queue1 = DispatchQueue(label: "com.iii17-grace.queue1", qos: .utility)
let queue2 = DispatchQueue(label: "com.iii17-grace.queue2", qos: .utility)

queue1.async {
    for i in 0...5{
        print(i)
    }
}

queue2.async {
    for i in 6...9{
        print(i)
    }
}

//输出：0 5 1 6 2 7 3 8 4 9
```



### 创建两个不同优先级
queue1 > queue2，结果交替输出，但是系统优先把资源分配给优先级高的queue1

```swift
let queue1 = DispatchQueue(label: "com.iii17-grace.queue1", qos: .default)
let queue2 = DispatchQueue(label: "com.iii17-grace.queue2", qos: .utility)

queue1.async {
    for i in 0...5{
        print(i)
    }
}

queue2.async {
    for i in 6...9{
        print(i)
    }
}

//输出：0 5 1 2 3 4 6 7 8 9


```

## 延时执行
通过对已经创建的队列，调用延时函数即可: asyncAfter(...)
### 基础定义结构体
```swift
public enum DispatchTimeInterval : Equatable {

    case seconds(Int)     //秒

    case milliseconds(Int) //毫秒

    case microseconds(Int) //微妙

    case nanoseconds(Int)  //纳秒

    case never
}

```

### wallDeadline和deadline
* 当系统睡眠后,wallDeadline会继续，但是deadline会被挂起;
* 例如：设置参数为60分钟，当系统睡眠50分钟，wallDeadline会在系统醒来之后10分钟执行，而deadline会在系统醒来之后60分钟执行;

```swift
let queue = DispatchQueue(label: "com.iii17-grace.queue1")

var time = DispatchTimeInterval.seconds(5)

queue.asyncAfter(wallDeadline: .now() + time) {
    print("wall deadline done")
}

queue.asyncAfter(deadline: .now() + time) {
    print("deadline done")
}

```


### DispatchGroup
如果想等到所有的队列的任务执行完毕再进行某些操作时，可以使用DispatchGroup来完成

```swift
let group = DispatchGroup()

let queue1 = DispatchQueue(label: "com.iii17-grace.queue1")
let queue2 = DispatchQueue(label: "com.iii17-grace.queue2")



queue1.async(group: group){
    for i in 0...5{
        print(i)
    }
}


queue2.async(group: group) {
    for i in 6...9{
        print(i)
    }
}


//如果想等待某一队列先执行完毕再执行其他队列可以使用wait
//group.wait()

//为防止队列执行任务时出现阻塞，导致线程锁死，可以设置超时时间
//group.wait(timeout: <#T##DispatchTime#>)
//group.wait(wallTimeout: <#T##DispatchWallTime#>)


group.notify(queue: DispatchQueue.main){
    print("group done")
    
}

//输出：0 6 1 7 2 3 8 4 5 9 group done
```


### DispatchWorkItem
可以通过此api设置队列执行的任务
* 初始化闭包

```swift
let workItem = DispatchWorkItem {
    for i in 0..<10 {
        print(i)
    }
}
```

* 两种调用方法

```swift
//perform
 DispatchQueue.global().async {
     workItem.perform()
 }
 
 //作为参数传给async
 DispatchQueue.global().async(execute: workItem)
```

* 内部方法及属性

```swift
init(qos: DispatchQoS = default, flags: DispatchWorkItemFlags = default,
    block: @escaping () -> Void)
    
//qos
优先级

//DispatchWorkItemFlags
public struct DispatchWorkItemFlags : OptionSet, RawRepresentable {

//执行情况
    //常用于读写隔离，与信号量相对应学习
    public static let barrier: DispatchWorkItemFlags 

    
    public static let detached: DispatchWorkItemFlags

    
    public static let assignCurrentContext: DispatchWorkItemFlags

//覆盖
    //没有优先级
    public static let noQoS: DispatchWorkItemFlags
    
    //继承queue的优先级
    public static let inheritQoS: DispatchWorkItemFlags
    
    //覆盖queue的优先级
    public static let enforceQoS: DispatchWorkItemFlags
}


```

* 即使设置了DispatchWorkItem但仅仅只设置了优先级并不会对任务执行顺序有任何影响
* 例如

```swift
//queue2优先级高于queue1
let queue1 = DispatchQueue(label: "com.iii17-grace.queue1", qos: .utility)
let queue2 = DispatchQueue(label: "com.iii17-grace.queue2", qos: .userInitiated)


//workItem1优先级=queue2优先级，并使用enforceQoS进行强制覆盖
let workItem1 = DispatchWorkItem(qos: .userInitiated, flags: .enforceQoS){
    for i in 0...5{
        print(i)
    }
}

let workItem2 = DispatchWorkItem{
    for i in 6...9{
        print(i)
    }
}

//此时queue1优先级被workItem1强制覆盖为和queue2相等的优先级
queue1.async(execute: workItem1)

queue2.async(execute: workItem2)

//所以输出是在两个相等的优先级前提下输出，两个队列交替运行
//输出： 6 0 7 1 8 2 9 3 4 5

```

* 也有wait和notify方法，和DispatchGroup用法相同

### DispatchSemaphore
同步一个异步队列，用信号量

```swift
//wait()会使信号量减一，如果信号量大于1则会返回.success，否则返回timeout（超时），也可以设置超时时间

func wait(wallTimeout: DispatchWallTime) -> DispatchTimeoutResult
func wait(timeout: DispatchTime) -> DispatchTimeoutResult


//signal()会使信号量加一，返回当前信号量
func signal() -> Int


```

例如
```swift
let semaphore = DispatchSemaphore(value: 1)

let queue = DispatchQueue(label: "com.ffib.blog.queue", qos: .utility, attributes: .concurrent)


for i in 0..<5 {
   
    if semaphore.wait(wallTimeout: .distantFuture) == .success {
        queue.async {
           
            print(i)
               
            
            semaphore.signal()
        }
    }
}

//输出：0 1 2 3 4








let semaphore = DispatchSemaphore(value: 1)

let queue = DispatchQueue(label: "com.ffib.blog.queue", qos: .utility, attributes: .concurrent)


for i in 0..<5 {
   
   
        queue.async {
           
            print(i)
               
            
            semaphore.signal()
        }
    
}

//输出：0 1 3 2 4
//因为queue创建的是异步，这几个queue的任务都是一块儿走的
```







## 总结

|      | 同步执行        | 异步执行        |
|------|-------------|-------------|
| 串行队列 | 当前线程，一个一个执行 | 其他线程，一个一个执行 |
| 并行队列 | 当前线程，一个一个执行 | 开很多线程，一起执行  |






