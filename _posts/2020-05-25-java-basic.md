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
- [ ] 面向过程
- [ ] 面向对象
- [ ] 基础类库
- [ ] 异常
- [ ] 注解与反射
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



参考资料：

1. 疯狂Java讲义(第三版)-李刚
2. https://www.cnblogs.com/chenglc/p/6922834.html
