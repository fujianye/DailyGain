# IOC和DI
控制反转(IOC)和依赖注入(DI)是Spring中最重要的核心概念之一，而两者实际上是一体两面的。

## 依赖注入
* 一个类依赖另一个类的功能，那么就通过注入，如构造器、setter方法等，将这个类的实例引入。
* 侧重于实现。
## 控制反转
* 创建实例的控制权由一个实例的代码剥离到IOC容器控制，如xml配置中。
* 侧重于原理。
* 反转了什么：原先是由类本身去创建另一个类，控制反转后变成了被动等待这个类的注入。
