java高级特性主要有集合框架及泛型，实用类，输入和输出处理，注解与多线程，网络编程与XML技术。

**集合框架**

是一套性能优良、使用方便的接口和类（位于java.util包中）解决数组在存储上不能很好适应元素数量动态变化，查找效率低的缺陷

集合接口： Map、Collection\(子接口List、Set\) 、 Iterator

接口实现类：HashMap TreeMap 、ArrayList LinkedList、 HashSet TreeSet 实现map、list、set接口

集合工具类：Arrays 、Collections 提供对集合元素进行操作的算法

**泛型集合**

泛型即参数化类型，通过指定集合中的元素类型来实现约束

作用：将对象的类型作为参数，指定到其他类或者方法上，从而保证类型转换的安全性和稳定性

**实用类**

Java API：Java应用程序的编程接口、Java帮助文档

实用类： 由Java API提供的常用类

学习这部分一定要多看 Java API 。

**输入/输出和反射**

**IO流常用基类**

注意：\( \)里面是子类 如File\*\*类，Buffered\*\*类

Buffered\*\*类带有缓冲区，有按行读取内容的readLine\(\)方法

**字节流**

字节输入流：InputStream \(FileInputStream、BufferedInputStream\)

字节输出流：OutputStream \(FileOutputStream、BufferedOutStream\)

**字符流**

字符输入流：Reader \(FileReader、BufferedReader\)

字符输出流：Writer \(FileWriter、BufferedWriter\)

**Java反射**

反射：指java程序能自描述和自控制，它允许程序在运行时才加载、探知、使用编译期间完全未知的类

反射机制：指在运行状态中，动态获取类信息以及动态调用对象方法的功能

**注解**

Java代码里的特殊标记。它为在代码中添加用Java程序无法表达的额外信息提供了一种形式化的方法。注解可以看成修饰符，修饰程序元素。

注解可以在编译、类加载、运行时被读取。而注释不会被程序所读取。

**线程调度**

多个线程处于可运行状态，线程调度会根据优先级来决定线程进入可运行状态的次序。

线程的优先级用1～10 表示，10的优先级最高，默认值是5

**网络编程技术**

网络：是信息传输、接收、共享的虚拟平台，把各个点、面、体的信息联系到一起，从而实现资源共享

网络编程：通过使用套接字来达到进程间通信目的的编程

**XML简介**

XML\(Extensibel Markup Language\)：即可扩展标记语言，是一种简单的数据存储语言，使用一些列简单的标记描述数据。

特点：与操作系统、开发平台无关；规范统一

作用：数据交互；配置应用程序和网站；Ajax基石

