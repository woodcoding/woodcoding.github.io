---
title: Java简明学习笔记
date: 2020-05-25 20:30:00
categories: 
 - Java
tags:
 - Java

---


&ensp;&ensp;&ensp;&ensp;JAVA基础系列，包括：

- [x] 数据类型
- [x] 流程控制与数组
- [x] 面向对象
- [x] 基础类库
- [ ] 异常
- [ ] 注解
- [ ] 反射
- [ ] 输入输出流
- [ ] 多线程与多进程
- [ ] 网络编程


<!-- more -->

### 数据类型

#### 标识符命名

   Java标识符必须以字母、下划线(\_)、美元符($)开头，后面可以跟任意数目的字母、数字、下划线(\_)和美元符。

#### Java关键字
   
   |类型|关键字|
   |-|-|
   |访问控制|private、protected、public|
   |类,方法和变量修饰符|abstract、class、extends、final、implements、interface、native、new、static、strictfp、synchronized、transient、volatile|
   |程序控制|break、continue、return、do、while、if、else、for、instanceof、switch、case、default|
   |错误处理|try、catch、throw、throws|
   |包相关|import、package|
   |基本类型|boolean、byte、char、double、float、int、long、short、null、true、false|
   |变量引用|super、this、void|
   |保留字|goto、const|

#### 基本数据类型
   1. 整型
      - byte: 内存占8位，表示范围-128~127
      - short： 内存占16位，表示范围-32768-32767
      - int: 占32位，表示$-2^{31}$ - $2^{31}-1$，默认定义的数字是int
      - long：占64位，表示$-2^{63}$ - $2^{63}-1$
   
   2. 字符型
      - char：Java使用16位的Unicode字符作为编码方式，也就是2字节，使用单引号(')包含。
  
      Java并没有提供表示字符串的基本类型，所以字符串使用String类来表示，使用双引号(")表示。

   3. 浮点型
      - float: 单精度浮点型，占4字节，32位
      - double: 双精度浮点型，占8字节，64位
   
   4. 数值下划线分隔
      Java7引入的新功能，数字可以使用下划线进行分隔，便于直观看出有多少位。如：```3.14_15_92_654```
    
   5. 布尔型
      boolean: 表示逻辑上的"真"或"假"。数值只能是true或者false，不能用0和1表示，其他类型的数据也不能转化成boolean类型。虽然boolean只需要一位就可以表示了，但是由于计算机通常使用8位作为内存最小分配单位，所以大部分情况下占用还是8位。

### 流程控制与数组

#### 流程控制

1. if-else结构
2. switch分支结构，如：
   ```java
   switch (expression){
       case condition1:{
           func(condition1);
           break;
       }
       case condition2:{
           func(condition2);
           break;
       }
       default:{
           func(1);
       }
   }
   ```
3. while结构，如：
   ```java
   while(expression){
       func(expression);
   }
   ```

4. do-while结构，如：
    ```java
    do{
        func();
    }while(expression);
    ```

5. for循环结构，如：
   ```java
   for(int i=0;i<10;i++){
       System.out.println(i);
   }
   ```

#### 数组

1. 定义
   Java定义数组时，以下两种定义方式都支持：

   ```java
   type[] arrayName;    //推荐使用，语义清晰，引用类型
   tpye arrayName[];
   ```

2. 初始化
   - 静态初始化
   ```java
   intArr = new int[]{5, 6, 7, 8}
   ```

   - 动态初始化
   ```java
   intArr = new int[5];
   ```

3. 使用
   - 获取长度： arr.length
   - foreach循环：
   ```java
   for(String book : books){
       System.out.println(books);
   }
   ```

4. Java8增强的工具类Arrays
   Java提供的Arrays类包含了一些static修饰的方法可以直接操作数组，详情可以自己看文档。



### 面向对象(上)

#### 类对象

Java中定义类和C++差不多，类似如下：

```java
public class Person{
   public String name;
   public int  age;
   public void say(String content){
      System.out.println(content);
   }
}
```



**注意：** Java中一个.java文件只能拥有一个public的类。对于各种修饰符类的访问权限在后面进行介绍。

#### 对象、引用、指针

Java中使用对象如下：

```java
public static void main(String[] args){
   Person p = new Person();
}
```

注意到代码中的p变量和Person对象，p变量其实是个引用，这个变量存储在栈中，而真正的对象Person存在堆中，p变量是对Person对象的引用，这和其他语言是一致的。引用和指针很像、但是引用和指针有区别（注意Java中的null很特殊，虽然可以使用p=null这种语法，但是并不能说p为空，关于Java中null的详细定义网上议论纷纷，还需继续深入学习）：
 - 引用不可用为空，指针可以为空。
 - 引用在创建时必须初始化，引用到一个有效对象，而指针在定义时不用。
 - 引用初始化以后不能被改变，指针可以改变所指的对象。

通过this引用可以在类的内部引用本身。


#### 局部变量初始化

Java中可以使用`{}`对类的成员变量进行初始化。在定义一个局部变量的时候系统并不会为之分配内存，只有赋值后系统才会分配内存。

#### 访问控制

```mermaid
graph LR;

id1([priavate]) --> id2([default])
id2 --> id3([protected])
id3 --> id4([public])
```
**注意** 访问级别由小到大。

访问控制级别表

| |private|default|protected|public|
|-|-|-|-|-|
|同一个类中|√|√|√|√|
|同一个包中|-|√|√|√|
|子类中|-|-|√|√|
|全局范围内|-|-|-|√|

#### 包管理

pacakge、import和import static
   
Java中允许使用pacakge对一组相同功能进行管理，但是要满足两个条件：
- 源文件使用package语句指定包名。
- class文件必须放在对应的目录下。

使用import语句可从其他包中导入类，使用import static可从其他包的类中导入静态方法或者静态属性。

#### 构造器
   Java构造器和C++也类似。不过要注意，构造器代码进行之前，系统已经为对象分配好内存空间了，只是暂时还不能访问而已，构造器只是可以对这个对象进行初始化操作而已。使用this关键字可以在重载的构造器中调用另一个构造器，并且**必须作为构造器执行体第一条语句**。

#### 继承
   使用extends关键字进行类的继承，而且只能单继承，如下：
   
   ```java
   public class Fruit{
      public void info(){
         System.out.println("我是水果");
      }
   }

   public class Apple extends Fruit{
      @Override
      public void info(){
         super.info();
         System.out.println("我是苹果");
      }
   }

   ```

   使用`super`关键字可以使用父类的资源。当程序创建子类时，如果存在同名的实例变量，则系统会为他们都分配内存，而不是只分配一份。

#### 多态
   Java引用变量有两个类型，一个是编译时类型，一个是运行时类型。编译时类型由声明该变量时使用的类型决定。运行时类型由实际赋值的类型决定。因此就有可能出现多态。比如Dog、Cat都继承了Animal。则声明以下代码时：
   ```java
   Animal dog = new Dog();
   Animal cat = new Cat();
   dog.say();
   cat.say();
   ```
   由于猫狗的叫声不一样，所以可能出现“汪汪汪”和“喵喵喵”的区别。

   **注意** 使用instanceof可以判断对象是否是某种类型，方便正确进行多态转换。只要是类型相同或者具有继承关系则返回true。如:
   
   ```java
   hello instanceof Animal
   ```


### 面向对象(下)

#### Java8增强的包装类

JDK1.5开始提供了自动装箱和自动拆箱的功能。Java中对8种基本数据类型提供了包装类。如下：

|基本数据类型|包装类|
|-|-|
|byte|Byte|
|short|Short|
|int|Integer|
|long|Long|
|char|Character|
|float|Float|
|double|Double|
|boolean|Boolean|


#### 对象比较

由于包装类对象其实是引用类型，所以以下代码会输出false：

```java
System.out.println(new Integer(2)==new Integer(2));
```

但是由于Java可以自动装箱，以下代码会输出true：

```java
Integer a = 127;
Integer b = 127;
System.out.println(a == b);
```

因为在Java中对-128--127做了缓存，所以a和b其实是一个对象。同理也可以应用到Java的字符串对象，Java对字符串常量做了常量池管理，所以如果是静态定义相同的字符串值是相等的，但是字符串运算或者new出来的即使值相等，==也是false。如下：

```java
String s0 = "Java学习";
String s1 = "Java学习";
String s2 = "Java";
String s3 = "学习";
String s4 = s2 + s3;
System.out.println(s4==s1);
System.out.println(s0==s1);
```

所以要想比较两个值相等，应该使用equals方法。

#### final修饰符

final修饰的变量不可被改变，一旦获得了初始值，该final变量的值就不能被重新赋值。而且final修饰的成员必须由程序员显式赋值。final修饰的成员不可重新赋值，但是如果改成员是一个引用对象，那么引用对象的成员是可以改变的。final在Java中的作用其实相当于C++中的“宏定义”。下面我们来看一个例子：

```java
String s1 = "Java学习";
final String s2 = "Java";
final String s3 = "学习";
String s4 = s2 + s3;
System.out.println(s4==s1);
```

代码和上面的差不多，区别就是给s2和s3加上了final修饰的关键字，现在执行，你会发现s4和s1输出的是true而不是false。这是因为final修饰的值是编译时就已经确定的，所以会纳入常量池。

#### 抽象类

先看一个抽象类的例子：

```java
abstract class JavaAbstract{
    public void test(){
        System.out.println("test");
    }

    abstract void go();
}

class AbstractImpl extends JavaAbstract{
    public void go(){
        System.out.println("Go!");
    }
}
```

Java中的抽象类使用abstract关键对抽象类进行定义。抽象类可以理解为一个模板，可以提供一些已经实现的方法，以及定义一些必须实现的抽象方法。这个模板不能被实例化，必须要被其他类继承并实现对应的抽象方法后才能进行实例化。由于abstract必须被继承实现，因此他和final、static、private有一些互斥关系，这个需要想清楚。

#### 接口

先看一个接口的例子：

```java
interface JavaInterface{
    public void test();
    public void go();
}

class InterfaceImpl implements JavaInterface{
    public void test(){
        System.out.println("test interface");
    }

    public void go(){
        System.out.println("go interface");
    }
}

```
Java中的接口使用interface关键字进行定义。接口可以理解为规范，这和抽象类的模板不同。相当于电脑上有内存条插槽，你只要实现了相关的内存条制造规范，那你就能插上电脑使用。这个主要是弥补Java单继承的不足，因为接口可以同时实现多个。这里放一下接口和抽象类的区别：

- 接口只能包含抽象方法和默认方法，而抽象类可以提供普通方法（至于默认方法和普通方法的区别？感觉这个默认方法就是个大坑，其实是因为Java中一些接口更新导致新版本编译不过的而提供的一个补救措施，但是却带来了多继承的问题，详情请自己了解，主要就是记住，少考虑默认方法，多继承下类方法优先）。
- 接口里不能定义静态方法，抽象类可以（书上是这么说的，但是Java8增强了，接口可以定义static方法，做加法真是害死人啊）。
- 接口中定义的成员属性都是静态的，而抽象类可以提供普通变量。
- 接口不含构造器，抽象类可以。
- 接口不能有初始化语句块，抽象类可以。
- 接口可以多继承，抽象类只能单继承。


#### 内部类

可以在内的内部再定义类，主要是为了更好的封装，如果只是一次性使用的类可以使用内部类进行定义。内部类中使用`外部类名.this.varName`对外部类进行访问。

#### Lambda表达式

Lambda表达式是Java8的重要更新，其实就是用更简洁的代码来创建只有一个抽象方法的接口(又称之为函数接口)。相比起Py的lambda表达式，这个语法真是太尴尬了。例如：

```java
Eatable obj = () -> {
   System.out.println("hh");
};
obj.eat();
```

#### 枚举类

和其他语言都差不多，直接举例吧：

```java
enum Gender{
    MALE("男"),
    FEMALE("女");

    private final String name;

    private Gender(String name){
        this.name = name;
    }
    public String getName(){
        return this.name;
    }
}
```

### 基础类库  


#### 基本库

Java的基础类库都是一些方法的调用，需要手动实践理解。其中提供了包括系统相关(System、Runtime等)、字符串相关(String、StringBuffer、StringBuilder等)、对象(Object)、数学(Math、BigDecimal等)、日期(Date、Calendar等)、正则、国际化等等。

```java

public class JavaBasicUtils {
    public static void main(String[] args) {
        Runtime rt = Runtime.getRuntime();
        System.out.println(rt.availableProcessors());
        System.out.println(rt.freeMemory());
        System.out.println(rt.maxMemory());

        Random rd1 = new Random(System.currentTimeMillis());
        Random rd2 = new Random(System.currentTimeMillis());

        System.out.println(rd1.nextInt());
        System.out.println(rd2.nextInt());

        ThreadLocalRandom tlr1 = ThreadLocalRandom.current();
        ThreadLocalRandom tlr2 = ThreadLocalRandom.current();
        System.out.println(tlr1.nextInt());
        System.out.println(tlr2.nextInt());

        Date d1 = new Date();
        Date d2 = new Date(System.currentTimeMillis()+100);
        System.out.println(d1);
        System.out.println(d2);
        System.out.println(d1.compareTo(d2));
        System.out.println(d1.before(d2));

        System.out.println(Locale.Category.FORMAT);
//        Locale myLocale = Locale.getDefault(Locale.Category.FORMAT);
        Locale myLocale = new Locale("en", "US");
        System.out.println(myLocale);
        ResourceBundle bundle = ResourceBundle.getBundle("mess", myLocale);
        System.out.println(bundle.getString("hello"));
        System.out.println(MessageFormat.format(bundle.getString("welcome"), new Date()));
    }
}
```

#### 集合

1. 概述
   Java中集合类型主要由两个接口派生：Collection和Map。详情查看下面的的图：
   ![](/resources/images/2020-06-04/2020-06-04-20-48-04.png)

   ![](/resources/images/2020-06-04/2020-06-04-20-49-55.png)

Collection中定义了一些基本方法，如add、clear、contains等。同时可以使用Iterator迭代器进行元素迭代，如下所示：

```java
public class JavaSet {
    public static void main(String[] args) {
        Collection c = new ArrayList();

        c.add("Java");
        c.add(100);
        c.add(100);

        System.out.println(c.size());
        System.out.println(c.contains(100));
        System.out.println(c.toString());

        c.forEach(obj-> System.out.println(obj));

        Iterator it = c.iterator();
        while (it.hasNext()){
            System.out.println(it.next());
            if(it.next().toString().equals("Java")){
               it.remove();
            }
        }

    }
}
```

同时Java还支持链式操作，如下：

```java
books.stream().filter(ele->((String)ele).contains("Java")).count();
```

2. Set
   Set是一个元素不重复的元组，在Java中有好几种数据结构的实现，根据不同的数据结构可以实现不同的操作。
   
   - HashSet：经典Set实现，不保证元素排列顺序，非线程安全，元素值可以为null（注意对象的比较，如果要保证自定义对象唯一，需要重写equals和hashCode方法）
   - LinkedHashSet：从名字可以看出是链表实现的Set，按照添加元素的先后添加元素，因此性能略低于HashSet，但是在迭代访问全部元素时性能很好。
   - TreeSet:从名字可以看出是树结构实现的Set，非线程安全，由于也是SortedSet接口的实现，所以元素有序，可以有序访问元素以及上一个、下一个等。
   - EnumSet：专为枚举类设计的集合类，非线程安全。
   
3. List
   - ArrayList是最常用的List实现，数组实现，允许对元素进行快速随机访问，当数组大小不满足时需要增加存储时需要将已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。
   - Vector与ArrayList一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一个线程能够写Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，访问它比访问ArrayList慢。但是Vector提供了一个Stack子类，即栈的实现。
   - LinkedList是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较慢。另外，他还提供了List接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。
   
4. Queue
   队列的实现。一个特别的实现是优先队列(PriorityQueue)。如下：

   ```java
   PriorityQueue pq = new PriorityQueue();
   pq.offer(6);
   pq.offer(11);
   pq.offer(8);
   System.out.println(pq.poll());
   ```

5. Map
   字典的实现，就是key-value的形式。同Set一样，Java中也有多种实现：
   - HashMap与HashTable：HashMap非线程安全，允许null为key，而HashTable则相反，HashMap是比较常用的，保证线程安全可以使用后面讲的Collection类。
   - LinkedHashMap：主要是基于Hash再加上前后指针的双向链表实现，性能略低于HashMap，但是迭代全部元素时性能挺好，可以保持插入顺序或者访问顺序进行读取。
   - TreeMap：基于红黑树数据结构实现，可以保证key有序。
   - WeakHashMap：key对象只持有对实际对象的弱引用，当垃圾回收了key对应的对象后，此key将被删除。
   - IdentityHashMap：只有当两个key对象完全相等，即==号判断为true时才是相同的key，因此可以添加两个new String("Java")到Map中。
   - EnumMap：顾名思义，只保存枚举类的Map。
   
6. Collections
   操作集合的工具类，提供了集合的多种操作：
   - 排序操作：reverse、shuffle、sort、swap、rotate等。
   - 查找替换：binarySearch、max、min、fill等。
   - 同步控制：使用Collections.synchronizedXXX等方法可以让非线程安全的结合变成线程安全，但是通过Iterator、Spliterator或Stream遍历这个新List时，需要在外部做好同步。如：
  
      ```java
      List list = Collections.synchronizedList(new ArrayList());
      ```

### 泛型

### 异常

### 注解

### 反射

### 输入输出流

### 多线程

### 网络编程


参考资料：

1. 疯狂Java讲义(第三版)-李刚
2. https://www.cnblogs.com/chenglc/p/6922834.html
3. https://www.cnblogs.com/wanlipeng/archive/2010/10/21/1857791.html