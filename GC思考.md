# 思考
Garbage collection: 垃圾回收
清晰的描述了GC是什么，但没有体现出GC的本质
GC的本质是自动化的内存管理
# 内存管理
## 内存管理为什么重要
计算需要两大资源--CPU和内存;资源的管理是核心问题。

那为啥cpu的管理没让程序员关心呢？

cpu管理主要由操作系统完成;
多线程，异步其实就是对CPU进行管理，所以程序员实际上还是要操心CPU的管理。
这也解释了，为什么面试时，多线程和GC是经常被问到的问题。

## 内存管理为什么需要自动化
### 如果你使用面向对象，回避使用自动化管理机制的可能性几乎为零
* 引用计数。
### 半自动化的尝试
* Rust通过手动标识，让编译器来解决。

borrowing,lifetime
让代码很难读，而且没有彻底解决问题。fn mutate_and_share<'a>(&'a mut self) -> &'a Self { &'a *self }

内存管理很繁琐很容易出错。为啥C/C++程序员会喜欢GoLang.

# GC的历史
John McCarthy身为Lisp之父和人工智能之父，同时，他也是GC之父。1960年，他在其论文中首次发布了GC算法（其实是委婉的提出）。

java发扬光大。

# GC算法的核心工作
如何分辨出什么是垃圾？这点在C++里就不容易实现，root set 遍历。
如何、何时搜索垃圾？
如何、何时清除垃圾？
# 如何评价GC算法
## 吞吐量
单位时间内的处理能力
## 最大暂停时间
GC执行过程中，应用暂停的时长。 较大的吞吐量和较短的最大暂停时间不可兼得
## 堆的使用效率
堆空间的利用率。 可用的堆越大，GC运行越快；相反，越想有效地利用有限的堆，GC花费的时间就越长。
## 访问的局部性把具有引用关系的对象安排在堆中较近的位置，就能提高在缓存中读取到想利用的数据的概率。
那我们掌握了以上本质的东西，就不会在各个算法里面晕头转向了，万变不离其宗。

# 算法介绍
## G1
两个过程
* 全局并发标记（global concurrent marking） 
* 拷贝存活对象（evacuation） 

Global concurrent marking基于SATB形式的并发标记。它具体分为下面几个阶段： 

1、初始标记（initial marking）：暂停阶段。扫描根集合，标记所有从根集合可直接到达的对象并将它们的字段压入扫描栈（marking stack）中等到后续扫描。G1使用外部的bitmap来记录mark信息，而不使用对象头的mark word里的mark bit。在分代式G1模式中，初始标记阶段借用young GC的暂停，因而没有额外的、单独的暂停阶段。 

2、并发标记（concurrent marking）：并发阶段。不断从扫描栈取出引用递归扫描整个堆里的对象图。每扫描到一个对象就会对其标记，并将其字段压入扫描栈。重复扫描过程直到扫描栈清空。过程中还会扫描SATB write barrier所记录下的引用。 

3、最终标记（final marking，在实现中也叫remarking）：暂停阶段。在完成并发标记后，每个Java线程还会有一些剩下的SATB write barrier记录的引用尚未处理。这个阶段就负责把剩下的引用处理完。同时这个阶段也进行弱引用处理（reference processing）。 注意这个暂停与CMS的remark有一个本质上的区别，那就是这个暂停只需要扫描SATB buffer，而CMS的remark需要重新扫描mod-union table里的dirty card外加整个根集合，而此时整个young gen（不管对象死活）都会被当作根集合的一部分，因而CMS remark有可能会非常慢。 

4、清理（cleanup）：暂停阶段。清点和重置标记状态。这个阶段有点像mark-sweep中的sweep阶段，不过不是在堆上sweep实际对象，而是在marking bitmap里统计每个region被标记为活的对象有多少。这个阶段如果发现完全没有活对象的region就会将其整体回收到可分配region列表中。 

Evacuation

Evacuation阶段是全暂停的。它负责把一部分region里的活对象拷贝到空region里去，然后回收原本的region的空间。 

Evacuation阶段可以自由选择任意多个region来独立收集构成收集集合（collection set，简称CSet），靠per-region remembered set（简称RSet）实现。这是regional garbage collector的特征。

在选定CSet后，evacuation其实就跟ParallelScavenge的young GC的算法类似，采用并行copying（或者叫scavenging）

算法把CSet里每个region里的活对象拷贝到新的region里，整个过程完全暂停。

从这个意义上说，G1的evacuation跟传统的mark-compact算法的compaction完全不同：前者会自己从根集合遍历对象图来判定对象的生死，不需要依赖global concurrent marking的结果，有就用，没有拉倒；而后者则依赖于之前的mark阶段对对象生死的判定

## C4
### 三个过程
#### 标记（Marking） 
找到活动对象
#### 重定位（Relocation） 
将存活对象移动到一起，以便可以释放较大的连续空间，这个阶段也可称为压缩（compaction）
#### 重映射（Remapping）
更新被移动的对象的引用。
### 标记（Marking）
标记阶段（marking phase）使用了并发标记（concurrent marking）和引用跟踪（reference-tracing）的方法来标记活动对象

在标记阶段中，GC线程会从线程栈和寄存器中的活动对象开始，遍历所有的引用，标记找到的对象，这些GC线程会遍历堆上所有的可达（reachable）对象.----在这个阶段，C4算法与其他并发标记器的工作方式非常相似。

C4算法的标记器与其他并发标记器的区别

在并发标记阶段中，如果应用程序线程修改未标记的对象，那么该对象会被放到一个队列中，以备遍历。这就保证了该对象最终会被标记，也因为如此，C4垃圾回收器或另一个应用程序线程不会重复遍历该对象。这样就节省了标记时间，消除了递归重标记（recursive remark）的风险。

如果C4算法的实现是基于脏卡表（dirty-card tables）或其他对已经遍历过的堆区域的读写操作进行记录的方法，那垃圾回收线程就需要重新访问这些区域做重标记。在极端条件下，垃圾回收线程会陷入到永无止境的重标记中 —— 至少这个过程可能会长到使应用程序因无法分配到新的内存而抛出OOM错误。但C4算法是基于LVB（load value barrier）实现的，LVB具有自愈能力，可以使应用程序线程迅速查明某个引用是否已经被标记过了。如果这个引用没有被标记过，那么应用程序会将其添加到GC队列中。一旦该引用被放入到队列中，它就不会再被重标记了。应用程序线程可以继续做它自己的事。

脏对象（dirty object）和卡表（card table） 由于某些原因（例如在一个并发垃圾回收周期中，对象被修改了），垃圾回收器需要重新访问某些对象，那么这些对象脏对象（dirty object）。这这些脏对象，或堆中脏区域的引用，通过会记录在一个专门的数据结构中，这就是卡表。

在C4算法中，并没有重标记（re-marking）这个阶段，在第一次便利整个堆时就会将所有可达对象做标记。因为运行时不需要做重标记，也就不会陷入无限循环的重标记陷阱中，由此而降低了应用程序因无法分配到内存而抛出OOM错误的风险。

#### 重定位（Relocation）
重定位阶段是由GC线程和应用程序线程以协作的方式，并发完成的。

这是因为GC线程和应用程序线程会同时工作，无论哪个线程先访问将被移动的对象，都会以协作的方式帮助完成该对象的移动任务。

因此，应用程序线程可以继续执行自己的任务，而不必等待整个垃圾回收周期的完成。

应用程序线程先访问了要被移动的对象，那么应用程序线程也会帮助完成移动该对象的工作的初始部分，这样，它就可以很快的继续做自己的任务。虚拟地址（指相关引用）可以指向新的正确位置，内存也可以快速回收。

如果是GC线程先访问到了将被移动的对象，那事情就简单多了，GC线程会执行移动操作的。

如果在重映射阶段（re-mapping phase，后续会提到）也访问这个对象，那么它必须检查该对象是否是要被移动的。如果是，那么应用程序线程会重新定位这个对象的位置，以便可以继续完成自己任务。（对大对象的移动是通过将该对象打碎再移动完成的）
重映射（Remapping）
在重定位阶段，某些指向被移动的对象的引用会自动更新。

但是，并不是所有的

#### 重映射阶段（re-mapping phase）
负责完成对那些活动对象已经移出，但仍指向那些的引用进行更新。

在重定位阶段，活动对象已经被移动到了一个新的内存页中。在重定位之后，GC线程立即开始更新那些仍然指向之前的虚拟地址空间的引用，将它们指向那些被移动的对象的新地址。 

但如果在GC完成对所有引用的更新之前，应用程序线程想要访问这些引用的话，会出现什么情况呢？

如果在重映射阶段，应用程序线程访问了处于非稳定状态的引用，它会找到该引用的正确指向。如果应用程序线程找到了正确的引用，它会更新该引用的指向。当完成更新后，应用程序线程会继续自己的工作。

协作式的重映射保证了引用只会被更新一次，该引用下的子引用也都可以指向正确的新地址。

真的是无暂停的么
在重映射阶段，正在跟踪引用的线程仅会被中断一次，而这次中断仅仅会持续到对该引用的检索和更新完成，在这次中断后，线程会继续运行。

## ZGC
### 三个过程
ZGC与C4是非常相似的，也分为标记，重定位，重映射三个阶段。

区别在于，标记时的两个stw阶段，以及有色指针的引入。

标记
1 start mark : 暂停阶段,遍历所有线程堆栈以标记root set。 它通常由本地和全局变量组成，但也包括其他内部VM结构（例如JNI句柄）。

2 并发标记 

3 end mark 清空并遍历所有线程局部标记缓冲区。 由于GC可能会发现一个未标记的大型子图，因此可能需要更长时间。 ZGC通过在1毫秒后停止标记阶段的结束来避免这种情况。它返回到并发标记阶段，直到遍历整个图，然后可以再次开始标记阶段的结束。

重定位
1 prepare 并发阶段

2 starting 暂停阶段，与starting mark 阶段非常相似，区别在于relocate root set里的对象

3 relocate 并发阶段

重映射
