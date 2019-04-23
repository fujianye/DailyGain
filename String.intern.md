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
最初的猜想是'java'看起来像个保留字，是不是在JVM启动的时候已经写到常量池里了，类似的还有'main'、'int'、'float'。
于是测试了下以下例子,进一步验证了猜想
```java
    String str1 = new StringBuilder("jc").append( "vc" ).toString();//true
    System.out.println(str1.intern()==str1);
    String str2=new StringBuilder("mai").append( "n" ).toString();  //false
    System.out.println(str2.intern()==str2);    
    String str3=new StringBuilder("in").append( "t" ).toString();   //flase
    System.out.println(str3.intern()==str3);    
    String str4=new StringBuilder("flo").append( "at" ).toString(); //flase
    System.out.println(str4.intern()==str4);
```
# 四 如何确定 intern 的效率
最好的方法是对整个堆执行一次堆转储。堆转储也会在发生 OutOfMemoryError 时执行。
在 MAT （内存分析工具，译者注）中打开转储文件，然后选择 java.lang.String，依次点击“Java Basics”、“Group By Value”。

根据堆的大小，上面的操作可能耗费比较长的时间。最后可以看到类型这样的结果。按 “Retained Heap” 或者是 “Objects” 列进行排序，可以发现一些有趣的东西：

从这快照中我们可以看到，空的字符串占用了大量的内存！两百万个空字符串对象占用了总共 130 MB 的空间。另外可以看到一部分被加载的 JavaScript 脚本，一些作为键的字符串，它们被用于定位。另外，还有一些与业务逻辑相关的字符串。

这些与业务逻辑相关的字符串是最容易进行 intern 操作的，因为我们清楚地知道它们是在什么地方被加载进内存的。对于其他字符串，可以通过 “Merge shortest Path to GC Root” 选项来找到它们被存储的位置，这个信息也许能够帮助我们找到该使用 intern 的地方。

## intern 的利弊
既然 intern() 方法有这些好处，为什么不经常使用呢？原因在于它会降低代码效率。下面给出一个例子：
```java
private static final int MAX = 40000000;
public static void main(String[] args) throws Exception {
    long t = System.currentTimeMillis();
    String[] arr = new String[MAX];
    for (int i = 0; i < MAX; i++) {
        arr[i] = new String(DB_DATA[i % 10]);
        // and: arr[i] = new String(DB_DATA[i % 10]).intern();
    }
    System.out.println((System.currentTimeMillis() - t) + "ms");
    System.gc();
    System.out.println(arr[0]);
}
```
代码中使用了字符串数组来维护到字符串对象的强引用，另外我们还打印了数组的第一个元素来避免数组由于代码优化而将数组给销毁了。接着从数据库加载 10 个不同的字符串，但在这里我使用了 new String() 来创建一个临时的字符串，这和从数据库里读是一样的。最后我们调用了系统的 GC() 方法，这样就能排除其他不相关对象的影响，保证结果的正确。 在 64 位，8 G 内存，i5-2520M 处理器的 Windows 系统上运行上面的代码， 环境为 JDK 1.6.0_27，指定虚拟机参数 -XX:+PrintGCDetails -Xmx6G -Xmn3G 记录垃圾回收日志。结果如下：

没有使用 intern() 方法的结果：
```java
1519ms
[GC [PSYoungGen: 2359296K->393210K(2752512K)] 2359296K->2348002K(4707456K), 5.4071058 secs] [Times: user=8.84 sys=1.00, real=5.40 secs] 
[Full GC (System) [PSYoungGen: 393210K->392902K(2752512K)] [PSOldGen: 1954792K->1954823K(1954944K)] 2348002K->2347726K(4707456K) [PSPermGen: 2707K->2707K(21248K)], 5.3242785 secs] [Times: user=3.71 sys=0.20, real=5.32 secs] 
DE
Heap
 PSYoungGen      total 2752512K, used 440088K [0x0000000740000000, 0x0000000800000000, 0x0000000800000000)
  eden space 2359296K, 18% used [0x0000000740000000,0x000000075adc6360,0x00000007d0000000)
  from space 393216K, 0% used [0x00000007d0000000,0x00000007d0000000,0x00000007e8000000)
  to   space 393216K, 0% used [0x00000007e8000000,0x00000007e8000000,0x0000000800000000)
 PSOldGen        total 1954944K, used 1954823K [0x0000000680000000, 0x00000006f7520000, 0x0000000740000000)
  object space 1954944K, 99% used [0x0000000680000000,0x00000006f7501fd8,0x00000006f7520000)
 PSPermGen       total 21248K, used 2724K [0x000000067ae00000, 0x000000067c2c0000, 0x0000000680000000)
  object space 21248K, 12% used [0x000000067ae00000,0x000000067b0a93e0,0x000000067c2c0000)
```
使用了 intern() 方法的结果：
```java
4838ms
[GC [PSYoungGen: 2359296K->156506K(2752512K)] 2359296K->156506K(2757888K), 0.1962062 secs] [Times: user=0.69 sys=0.01, real=0.20 secs] 
[Full GC (System) [PSYoungGen: 156506K->156357K(2752512K)] [PSOldGen: 0K->18K(5376K)] 156506K->156376K(2757888K) [PSPermGen: 2708K->2708K(21248K)], 0.2576126 secs] [Times: user=0.25 sys=0.00, real=0.26 secs] 
DE
Heap
 PSYoungGen      total 2752512K, used 250729K [0x0000000740000000, 0x0000000800000000, 0x0000000800000000)
  eden space 2359296K, 10% used [0x0000000740000000,0x000000074f4da6f8,0x00000007d0000000)
  from space 393216K, 0% used [0x00000007d0000000,0x00000007d0000000,0x00000007e8000000)
  to   space 393216K, 0% used [0x00000007e8000000,0x00000007e8000000,0x0000000800000000)
 PSOldGen        total 5376K, used 18K [0x0000000680000000, 0x0000000680540000, 0x0000000740000000)
  object space 5376K, 0% used [0x0000000680000000,0x0000000680004b30,0x0000000680540000)
 PSPermGen       total 21248K, used 2725K [0x000000067ae00000, 0x000000067c2c0000, 0x0000000680000000)
  object space 21248K, 12% used [0x000000067ae00000,0x000000067b0a95d0,0x000000067c2c0000)
```
可以看到结果差别十分的大。在使用 intern() 方法的时候，程序耗时多了 3 秒，但节省了很大一块内存。使用 intern() 方法的程序占用了 253472K(250M) 内存，而不使用的占用了 2397635K (2.4G)。从这些可以看出使用 intern 的利弊。

# 五 用String.intern()作为synchronized的对象锁
String.intern放进的StringTable是一个固定大小的Hashtable，默认值是1009，如果放进StringTable的String非常多，就会造成Hash冲突严重，从而导致链表会很长，而链表长了后直接会造成的影响就是当调用String.intern时性能会大幅下降（因为要一个一个找）。

现在仔细想想，看来当时这个case并不是因为频繁抛异常造成的，而是因为这个case中抛的是NoSuchMethodException，而抛这个异常的原因是因为调用了Class.getMethod找方法没找到，在class.getMethod这方法的实现里会调用name.intern，而很不幸的是这个case里传入的name会根据请求而变，因此导致了StringTable中放入了很多的String，hash冲突严重，链表变长，从而才导致了造成了String.intern过程变得比较耗CPU。

JDK为了解决这个问题，在6u32以及JDK 7的版本里支持了StringTable大小的配置功能，可在启动参数上增加-XX:StringTableSize来设置，具体的信息见：http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6962930
不过目前JDK未提供方法来查看StringTable中各桶的链表长度，如果提供这个的话就更好了
了解String.intern()在jdk7的变化后,我们为了在单例类里并发时对同一个用户保证操作原子性,会加同步块,例如:
```java
synchronized (("" + userId).intern()) {
            // TODO:something
   }
 ```
这个在jdk6里问题不算大,因为String.intern()会在perm里产生空间,如果perm空间够用的话,这个不会导致频繁Full GC,
但是在jdk7里问题就大了,String.intern()会在heap里产生空间,而且还是老年代,如果对象一多就会导致Full GC时间超长!!!
慎用啊!
解决办法见下面：
这里要引用强大的google-guava包，这个包不是一般的强大，是完全要把apache-commons*取缔掉的节奏啊！！！
```java
Interner<String> pool = Interners.newWeakInterner();
synchronized ( pool.intern("BizCode"+userId)){
    //TODO:something
}
```
代码参考TEST类：https://chromium.googlesource.com/external/guava-libraries/+/release15/guava-tests/test/com/google/common/collect/InternersTest.java
 原理？折腾一下看看这个类的原码吧~其实实现并不难，就是折腾而已~API上是这么说的：
> Interners.newWeakInterner()
>Returns a new thread-safe interner which retains a weak reference to each instance it has interned, and so does not prevent these instances from being garbage-collected. This most likely does not perform as well as newStrongInterner(), but is the best alternative when the memory usage of that implementation is unacceptable. Note that unlike String.intern(), using this interner does not consume memory in the permanent generation.

这样就可以解决FULL GC问题了吧。效果如何？试试看。

厄.其实这样也会使堆产生很多String,但能被回收掉


参考资料：
1.https://www.zhihu.com/question/57124207/answer/151835713
2.https://www.zhihu.com/question/51102308/answer/124441115
3.http://blog.csdn.net/raintungli/article/details/38595573
4.https://www.jianshu.com/p/d5ecfceccccd
5.https://www.jianshu.com/p/b98851899f37
