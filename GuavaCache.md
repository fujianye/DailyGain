# 一、Guava cache使用介绍
Guava Cache是在内存中缓存数据，相比较于数据库或redis存储，访问内存中的数据会更加高效。
Guava官网介绍，下面的这几种情况可以考虑使用Guava Cache：
  * 1、愿意消耗一些内存空间来提升速度。
  * 2、预料到某些键会被多次查询。
  * 3、缓存中存放的数据总量不会超出内存容量。

不适宜使用Guava Cache的情况：
  * 1、所有机器共享缓存，要保证一致性

所以，可以将程序频繁用到的少量数据存储到Guava Cache中，以改善程序性能。下面对Guava Cache的用法进行详细的介绍。
Guava cache可以做如下设置：
 * 设置最大存储
 * 设置过期时间
 * 弱引用
 * 显示清除
 * 移除监听器
 * 自动加载
 * 统计信息
 * 刷新
 * ...

# 二、与Redis对比


# 三、与Map做缓存的对比
服务是在启动的时候一下把常用的交易配置信息是从DB或者文件中查出来放在Map里面来做缓存。
如果想更新交易配置信息是不是需要每次都重启服务器，或者开几个后门接口用来更新Map信息。
这样还得考虑线程安全的问题。


# 四、
 
