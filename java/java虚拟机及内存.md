# Java虚拟机运行时数据区
1. 程序计数器
2. Java堆
3. Java虚拟机栈
4. 本地方法栈
5. 方法区  

方法区、Java堆所有线程共有 , 其他线程私有

## 程序计数器
1. 一块较小的内存空间,作用可以看做是当前线程所执行的字节码的行号指示器
2. 如果线程正在执行的是一个Java方法,计数器记录的是正在执行的虚拟机字节码指令地址;如果执行的是Native方法,这个计数器值为空
3. 内存区域小,是唯一一个在Java虚拟机中没有规定任何OutOfMemoryError情况的区域

## Java虚拟机栈
1. 特征:  
线程私有  
后进先出  
存储栈帧,支撑Java方法的调用、执行和退出  
可能出现OutOfMemoryError和StackOverflowError  
2.栈帧  
Java虚拟机栈中存储的内容，它被用于存储数据和部分过程结果的数据结构，也同时被用来处理动态链接、方法返回值和异常分派  
完整栈帧包含:局部变量表、操作数栈、动态链接信息方法正常完成异常完成信息  
局部变量表:  
局部变量表的容量是以局部变量槽Slot为最小单位,由编译期决定  
单个Slot可以存储一个类型为boolean、byte、char、short、float、reference的数据,两个Slot可以存储一个类型为long或double的数据  
局部变量表用于方法间的参数传递,及方法执行过程中存储基础数据类型的值和对象的引用  
操作数栈:  
后进先出栈,由若干Entry组成,长度由编译期决定  
单个Entry即可存储一个Java虚拟机定义的任意数据类型的值,包括long、double类型,但存储long和double类型的Entry深度为2,其他深度为1  
方法执行过程中,栈帧用于存储计算参数和计算结果;
方法调用时,操作数栈用来准备调用方法参数以及接收方法返回结果

## Java本地方法栈  (Native)  
1. 特征:  
线程私有  
后进先出  
作用是支撑Native方法的调用、执行和退出  
可能出现OutOfMemoryError和StackOverflowError  

HotSpot将虚拟机栈和本地方法栈二和为一  

## Java堆
1. 特征  
全局共享  
通常是Java虚拟机中最大的一块内存  
作用是作为Java对象的主要存储区域  
JVMS明确要求该区域需要实现自动内存管理,即GC,Java堆是GC机制主要管理的部分,所以Java堆也被称为GC堆  
可能出现OutOfMemoryError  

```Java
Object object = new Object;
```
以这一句为例,等号左侧创建局部变量,在Java栈的本地变量表Slot中,预留了reference类型的引用位置  
等号右侧则在Java堆中创建了对象实例,并把它赋值到Java栈预留的Slot中

## Java方法区
1. 特征
全局共享  
作用是存储Java类的结构信息
可能出现OutOfMemoryError
### 运行时常量池  
1. 特征
全局共享  
方法区一部分  
允许运行时加入数据(intern)
存储Java类文件常量池中的符号信息
可能出现OutOfMemoryError

## 直接内存  
1. 特征  
随NIO引入,目的是避免在Java堆和Native堆中来回复制数据带来性能损耗  
全局共享  
可能出现OutOfMemoryError
