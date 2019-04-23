# 一 String.intern

## 1 String为什么不可变性

### 字符串常量池的需要

* 字符串常量池的诞生是为了`提升效率和减少内存分配`
* 程序有大部分时间在处理字符串，字符串很大概率会出现重复的情况。String的不可变性`使常量池很容易被管理和优化`


### 安全性考虑

* 字符串使用频繁，设计成不可变，`有效防止字符串被有意或者无意的篡改`
* String类被final修饰，同时所有的属性都被final修饰，即不可变


### 作为HashMap、HashTable等hash型数据key的必要

## 2 String常量池的设计

* 字符串常量存储在方法区的PermGen Space。在jdk1.7之后，字符串常量重新被移到了堆中
* 常量池指的是在编译期被确定，并被保存在已编译的.class文件中的一些数据。它包括了关于类、方法、接口等中的常量，也包括字符串常量。
* Java会确保一个常量池中相同的字符串常量有且仅有一个

## 3 String.intern方法
### 使用intern的理由
在等价比较上的性能提升并不是应该使用 intern 的理由。实际上，intern 的目的在于复用字符串对象以节省内存。
在明确知道一个字符串会出现多次时才使用 intern(),并且只用它来节省内存。
使用 intern() 方法的效率，取决于重复的字符串与唯一的字符串的比值。另外，还要看在产生字符串对象的地方，代码是不是容易进行修改。
### intern原理
intern() 方法需要传入一个字符串对象（已存在于堆上），然后检查 StringTable 里是不是已经有一个相同的拷贝。StringTable 可以看作是一个 HashSet，它将字符串分配在永久代上。StringTable 存在的唯一目的就是维护所有存活的字符串的一个对象。如果在 StringTable 里找到了能够找到所传入的字符串对象，那就直接返回它，否则，把它加入 StringTable 
### intern用法
* String str="kvill" 和 String str=new String("kvill")的区别

"kvill"都是字符串常量，它们在编译期就被确定了, 会在常量池中创建一个"kvill"字符串对象
用new String("kvill") 创建的字符串不是字符串常量，不能在编译期就确定，所以new String() 创建的字符串不放入常量池中，存放在堆空间


* String对象的创建

    String str1 = new String("kvill")；
    String str2 = new String("kvill");

new str1时创建了两个对象，先在常量池中创建的"kvill"对象，再在堆中创建string对象，注意这个创建的先后顺序;
new str2时创建一个对象，堆中的另外一个string对象
```java
    String s1=new String("str") + new String("01");//会在堆中新建s1对象"str01"; 
    String s2 = new String(s1);
```
这种方式创建s2的过程中，并不会去常量池中创建s1的"str01"对象，而是仅在堆里创建一个s2对象
示例1
```java
    String s0="kvill";
    String s1="kvill";
    String s2="kv" + "ill";
    System.out.println( s0==s1 );//true
    System.out.println( s0==s2 );//true
```
s0和s1中的"kvill"都是字符串常量，它们在编译期就被确定了, 存在常量池中，且只有一个, 所以s0==s1为true
而"kv"和"ill"也都是字符串常量，编译阶段会直接合成一个字符串，进而去常量池中查找是否存在"kvill"，所以s2在编译期就被解析为一个字符串常量，它也是常量池中"kvill"的一个引用，s0==s2为true
示例2
```java
    String s0="kvill";
    String s1=new String("kvill");
    String s2="kv" + new String("ill");
    System.out.println( s0==s1 );//false
    System.out.println( s0==s2 );//false
    System.out.println( s1==s2 );//false
```
s0还是常量池中"kvill"的引用，s1因为无法在编译期确定，所以是运行时创建的新对象"kvill"的引用
s2因为有后半部分new String("ill")所以也无法在编译期确定，所以也是一个新创建对象"kvill"的引用

* String.intern()方法的作用

    * 在jdk1.6中，当一个String实例str调用intern()方法时，Java查找常量池中是否有相同Unicode的字符串常量，如果有，则返回常量池中字符串常量的引用，如果没有，则在常量池中增加一个Unicode等于str的字符串并返回它的引用
    * 而在jdk1.7，当一个String实例str调用intern()方法时，Java查找常量池中是否有相同Unicode的字符串常量，如果有，则返回常量池中字符串常量的引用，这一点和jdk1.6没什区别。区别在于，如果没有，则不会再在常量池中增加一个Unicode等于str的字符串，而只是在常量池中生成一个指向堆中的str对象的引用，并返回



示例3
```java
    String s0= "kvill";
    String s1=new String("kvill");
    String s2=new String("kvill");
    System.out.println( s0==s1 );//false
    s1.intern();
    s2=s2.intern(); //把常量池中"kvill"的引用赋给s2
    System.out.println( s0==s1);//false,虽然执行了s1.intern(),但它的返回值没有赋给s1
    System.out.println( s0==s1.intern());//true, s1.intern()返回的是常量池中"kvill"的引用
    System.out.println( s0==s2 );//true
```
# 二 str2.intern()在jdk7和jdk6的重点分析

## 1 str2.intern()在jdk7和jdk6的区别

示例4

jdk7
```java
    String str2 = new String("str")+new String("01");//生成了3个对象, 常量池中对象"str"和"01"，堆中的字符串对象"str01"
    str2.intern(); //jdk1.7中，在常量池中找不到对象"str01"，所以会在常量池生成一个引用，指向堆中的字符串对象"str01"
    String str1 = "str01";//这句话是直接在常量池中生成"str01"对象，由于在常量池中有这么一个引用，指向堆中的字符串对象"str01"，所以将这个引用给    str1，而不会再在常量池中生成"str01"对象了。
    System.out.println(str2==str1);//所以str1和str2是指向同一个对象，jdk1.7中返回true
```

jdk6
```java
    String str2 = new String("str")+new String("01");//生成了3个对象, 常量池中对象"str"和"01"，堆中的字符串对象"str01"
    str2.intern(); //jdk1.6会在常量池生成一个字符串对象"str01"
    String str1 = "str01";//str1指向常量池字符串对象"str01"
    System.out.println(str2==str1);//由于str2和str1指向不同，jdk1.6中返回false
```
2  str2.intern()在jdk1.7典型问题分析
示例5
```java
    String str2 = new String("str")+new String("01");
    str2.intern();
    String str1 = "str01";
    System.out.println(str2==str1);//jdk1.7中返回true。
```
调换代码顺序
```java
    String str2 = new String("str")+new String("01");//生成了3个对象, 常量池中对象"str"和"01"，堆中的字符串对象"str01"
    String str1 = "str01";//直接在常量池中生成"str01"对象
    str2.intern(); //jdk1.7中，在常量池中已经有对象"str01"了，返回常量池中"str01"对象的引用
    System.out.println(str2==str1);//所以str1和str2是指向不同对象，返回false
```
示例6
```java
    String str2 = new String("str");//生成了2个对象, 常量池中对象"str"，堆中的字符串对象"str"
    str2.intern(); //jdk1.7中，在常量池中已经有对象"str"了，返回常量池中"str01"对象的引用
    String str1 = "str";//str1直接指向常量池中已有的"str01"对象
    System.out.println(str2==str1);//所以str1和str2是指向不同对象，返回false
```
示例7
```java
    String s1 = new String("str");//这句代码执行时，先在常量池中创建的"str"对象，即使发现"str"对象不存在，也无法生成指向堆中的字符串对象s1的引用，因为此时堆中的字符串对象s1还没有被创建，所以最终还是在常量池中创建的"str"对象。
    System.out.println(s1.intern() == s1);//s1.intern()指向了常量池中的"str"对象，s1指向了堆，所以返回false

    String s1=new String("str") + new String("01");//生成了3个对象，常量池中对象"str"和"01"，堆中的字符串对象"str01"
    System.out.println(s1.intern() == s1);//注意s1.intern()，"str01"在常量池中不存在, 所以会返回s1的引用；这里返回true，注意与上面的区别
```
示例8
```java
    String s1=new String("str") + new String("01");//生成了3个对象，常量池中对象"str"和"01"，堆中的字符串对象"str01"
    String s2 = new String(s1);//这种方式创建String，并不会去常量池创建s1中的"str01"对象，而是仅在堆里创建一个s2对象
    System.out.println(s2.intern() == s2); //s2.intern()发现没常量池中没有"str01"对象, 将堆中s2的引用返回；结果为true
```
# 三 其他问题
```java
        String s1 = “abc”;
        String s2 = “a”;
        String s3 = “bc”;
        String s4 = s2 + s3;
        System.out.println(s1 == s4);
```
输出false，因为s2+s3实际上是使用StringBuilder.append来完成，会生成不同的对象。
```java
        String s1 = “abc”;
        final String s2 = “a”;
        final String s3 = “bc”;
        String s4 = s2 + s3;
        System.out.println(s1 == s4);
```
输出true，因为final变量在编译后会直接替换成对应的值，所以实际上等于s4=“a”+”bc”，而这种情况下，编译器会直接合并为s4=“abc”，所以最终s1==s4。
```java
        String str1 = new StringBuilder("计算机").append("软件").toString();
        System.out.println(str1.intern() == str1);

        String str2 = new StringBuilder("ja").append("va").toString();
        System.out.println(str2.intern() == str2);
```
在Jdk1.6的时候均返回false，这个容易理解，因为intern()方法会把首次遇到的字符串实例复制到永久代中，而new StringBuilder创建出来的对象是在堆上的，所以str1.intern()拿出来的对象跟新创建的对象不相等。
而在JDK1.7上，第一个true，第二个flase.
第一个返回true的原因是 JDK1.7等虚拟机的intern()实现不会复制实例，而是在常量池中记录首次出现的实例引用，因此第一个返回的是true，这里也没有问题。
至于第二个返回false的例子，书上的解析是
>java这个字符串在执行StringBuilder.toString()之前已经出现过，字符串常量池中早已有它的引用。所以返回false

链接：https://www.jianshu.com/p/b98851899f37
转载
https://www.jianshu.com/p/d5ecfceccccd
本文出自zhh_happig的简书博客，谢谢
