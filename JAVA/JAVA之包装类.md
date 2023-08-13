# JAVA包装类
在JAVA中，基本数据类型是不能直接参与面向对象编程的，因为他们不是对象，为了解决这个问题，JAVA提供了包装类Wrapper Class来将基本数据类型转换成对象。

JAVA中的基本数据类型包括：byte,short,int,long,float,double,char,boolean。每个基本数据类型都有对应的包装类，命名规则为基本数据类型的首字母大写。

包装类的作用：
- 允许将一个基本数据类型被当作对象处理。
- 允许在类中定义基本数据类型的对象。
- 提供了一些方法，比如将一个字符串转换成对应的基本数据类型，或者将一个基本数据类型转换成字符串。

常用的包装类方法：
- 构造方法
```c
Integer(int value)
Integer(String s)
```
- valueOf（基本数据类型->包装类）
```c
int i = 123;
Integer integer = Integer.valueOf(i);
```
- xxxValue（包装类->基本数据类型类型）
```c
Integer integer = new Integer(123); 
int i = integer.intValue()
```
- toString（包装类->字符）
```c
Integer integer = new Integer(123); 
String s = integer.toString()
```
- parseXxx（字符串->基本数据类型）
```c
Integer integer = new Integer(123); 
String s = integer.toString()
```