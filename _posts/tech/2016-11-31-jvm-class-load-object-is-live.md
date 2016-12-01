---
layout: post
title: 《深入理解jvm》读书笔记之——判断对象存活的方法
category: 技术
tags: [tech]
keywords: jvm,classloader
description: 《深入理解jvm》读书笔记之——判断对象存活的方法
---


## 1、对象的状态    

对于jvm中的垃圾收集器中，判断对象是否可以被回收，哪些对象是否需要存活是有以下的方法的。     

### 1.1、引用计数法    

引用计数的实现很简单，判断方式也很高效，在大多数情况下都不错，但是如果有a对象和b对象这种相互循环引用的问题是无法有效的处理的    

### 1.2、可达性分析算法     

> 算法思路：通过一系列称为"GC ROOT"的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径被称为引用链，当一个对象到GC ROOT没有任何引用链相连接的时候，就证明这个对象是不可用的（如下图示例中的object5,6,7）。     

![](http://upload-images.jianshu.io/upload_images/584578-92b7d133d9837c4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)      

对于java而言，可作为GC ROOT的对象包括以下几种：    
- 虚拟机栈中引用的对象    
- 方法区中类静态属性引用的对象     
- 方法区中常量引用的对象    
- 本地方法栈中`JNI`引用的对象    

### 1.3、引用的类别    

对于jdk2之后，java对于引用的概念进行扩充，分为以下的几种：     
- 强引用：类似于new出来的对象，只要强引用在，gc永远不会回收     
- 软引用：是一种有用但是非必须的对象，系统在将要内存溢出异常之前，把这些对象列进回收范围里进行第二次回收，如果回收后还没有足够的内存才会抛出内存溢出，jdk2以后用softReference来实现。        
- 弱引用：也是非必须对象的一种，强度要低于软引用，被弱引用关联的对象只能生存到下一次gc发生之前。当gc时，无论内存是否充足，都会回收掉只被弱引用关联的对象，用weakReference来实现。      
- 虚引用：是最弱的一种引用关系。无法通过虚引用来获得对象实例。虚引用的目的就是让这个对象在gc被回收时收到一个系统通知，使用phantomReference来实现。    
        
### 1.4、回收的机制     

即使是可达性算法中，不可达的对象也不是一下就会被回收的。    
一般他们是要经历两次标记过程：如果对象在进行可达性分析后发现没有在GC ROOT的引用链上，他会被第一次标记并且进行一次筛选，筛选的条件是此对象是否需要执行`finalizer()`方法。当对象没有覆盖`finalizer()`方法，或者`finalizer()`方法已经被jvm调用过，虚拟机讲这两种情况都视为”没必要执行“。    
 
***注意:***我们在日常的开发中最好不要去操作`finalizer()`方法。    

### 1.5、回收方法区       

永久代的gc包含两个部分的内容：废弃常量和无用的类     

> 没有任何String对象引用常量池中的这个常量，也没有其他地方引用了这个字面量，如果这时发生内存回收，而且必要的话，这个常量就会被系统清理出常量池。常量池中的其他类（接口）、方法、字段的符号引用也与此类似。    

类需要同时满足下面3个条件才能算是“无用的类”：    
- 该类所有的实例都已经被回收，也就是Java堆中不存在该类的任何实例。     
- 加载该类的ClassLoader已经被回收。     
- 该类对应的java.lang.Class 对象没有在任何地方被引用，无法在任何地方通过反射访问该类的方法。    

虚拟机可以对满足上述3个条件的无用类进行回收，这里说的仅仅是“可以”，而并不是和对象一样，不使用了就必然会回收。    
在大量使用反射、动态代理、CGLib等ByteCode框架、动态生成JSP以及OSGi这类频繁自定义ClassLoader的场景都需要虚拟机具备类卸载的功能，以保证永久代不会溢出。