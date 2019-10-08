# 什么是SOLID原则？
S.O.L.I.D.是面向对象设计的最重要几个原则的首字母缩写。
S single responsibility Principle 单一职责
O open closed principle 开放封闭
L Liskvo Subsituation principle 里式替换
I interface segregation principle 接口隔离
D dependency inversion principle 依赖倒置

# single responsibility 单一职责原则
一个类只承担一种职责。也就是，一个类只做一件事。
如果多个职责在一个类中，修改一个职责会影响其他职责。
所以如果要做多件事，那就分解这个类。
## 优点
1.类的复杂度降低，一个类只负责一个功能，其逻辑要比负责多项功能简单的多。
2.类的可读性增强，阅读起来轻松。
3.可维护性强，一个易读、简单的类自然也容易维护。
4.变更引起的风险降低，变更是必然的，如果单一职责原则遵守的好，当修改一个功能时，可以显著降低对其他功能的影响。

# open closed 开放封闭原则
类应该对扩展开放，对修改关闭。
开放封闭原则的思想就是设计的时候，尽量让设计的类做好后就不再修改，如果有新的需求，通过新加类的方式来满足，而不去修改现有的类(代码)。那么在实际的项目开发中，是否能做到绝对的对修改关闭呢？答案一般也是否定的。既然这样，那么就要求我们在开发前，去找出变化点，然后针对变化点构造抽象，隔离出这些变化。由此可见，实现开闭原则关键是抽象。


# Liskvo Subsituation 里式替换原则


# Interface segregation 接口隔离原则

# Dependency inversion 依赖倒置原则
