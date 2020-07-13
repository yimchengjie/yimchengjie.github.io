---
title: JavaIO
date: 2017-11-14 10:01
categories:
  - JavaSE
tags:
  - JavaSE
toc: true
---

#### 第 1 节:文件

1. **File 类型**

- `java.io.File`类的对象可以表示文件和目录，在程序中一个 File 类对象可以代表一个文件或目录
- 当创建一个 File 对象后，就可以利用它来对文件或目录的属性进行操作，如：文件名、最后修改日期、文件大小等等
- 需要注意的是，File 对象并不能直接对文件内容进行读/写操作，只能查看文件的属性

#### 第 2 节:输出输出流

1. **输入输出流的概念与作用**

   - 流的特点:
     - 流是一串连续不断的数据的集合,只能先读取前面的数据后，再读取后面的数据。不管写入时是将数据分多次写入，还是作为一个整体一次写入，读取时的效果都是完全一样的
   - 输入流：从外存读取数据到内存，输出流：将数据从内存写到外存中

2. **Java 中输入输出流的类型**

   - 对于输入和输出流，由于传输格式的不同，又分为字节流和字符流：

3. **Java 的输入输出流的继承树**

   - Java I/O 主要包括:
     - **流式部分**:IO 的主体部分；
     - **非流式部分**:主要包含一些辅助流式部分的类，如：File 类、`RandomAccessFile`类和`FileDescriptor`等类；
     - **其他类**:文件读取部分的与安全相关的类，如：`SerializablePermissio`n 类，以及与本地操作系统相关的文件系统的类，如：`FileSystem`类和`Win32FileSystem`类和`WinNTFileSystem`类。

4. **字节输出流**

   - OutputStream 提供了 3 个 write 方法来做数据的输出，这个是和 InputStream 是相对应的

5. **OutputStream**

   - OutputStream 是一个抽象类，提供了 Java 向流中以字节为单位写入数据的公开接口，大部分字节输出流都继承自 OutputStream 类

6. **DataOutput**

   - DataOutput 接口规定一组操作，用于直接向流中写入基本类型的数据和字符串：
     - DataInput 对基本数据类型的写入分别提供了不同的方法，方法名满足`writeXXX()`的规律,如`writeInt()`表示向流中写入一个 int 型数据，写入字符串的方法为`writeUTF()`

7. **常见字节输出流工具的作用与使用**

   - FileOutputStream 类用来处理以文件作为数据输出目的数据流；一个表示文件名的字符串，也可以是 File 或 FileDescriptor 对象。
   - 创建一个文件流对象有以下方法:
     - 方式 1：
  
       ```java
           File f=new File(“d:/abc.txt”);
           FileOutputStream out=new FileOutputStream (f);
       ```

     - 方式 2：
  
       ```java
           FileOutputStream out=new FileOutputStream(“d:/abc.txt”);
       ```

     - 方式 3：构造函数将 FileDescriptor()对象作为其参数。
  
       ```java
           FileDescriptor() fd=new FileDescriptor();
           FileOutputStream f2=new FileOutputStream(fd);
       ```
  
     - 方式 4：构造函数将文件名作为其第一参数，将布尔值作为第二参数。
  
       ```java
           FileOutputStream f=new FileOutputStream(“d:/abc.txt”,true);
       ```

8. **字节输入流**

   - InputStream 是输入字节数据用的类，所以 InputStream 类提供了 3 种重载的 read 方法.

9. **InputStream**

   - InputStream 也是一个抽象类，提供了 Java 中从流中以字节为单位读取数据的公开接口，大部分字节输入流都继承自 InputStream 类

10. **DataInput**

    - DataInput 接口规定一组操作，用于以一种与机器无关（当前操作系统等）的方式，直接在流中读取基本类型的数据和字符串：
    - DataInput 对基本数据类型的读取分别提供了不同的方法，方法名满足 readXXX()的规律,如 readInt()表示从流中读取一个 int 型数据读取字符串的方法为 readUTF()

11. **常见的字节输入流工具的作用与使用**

    - FileInputStream 类是 InputStream 类的子类，用来处理以文件作为数据输入源的数据流。
      - 使用方法:
        - 方式 1：
  
        ```java
        File fin=new File(“d:/abc.txt”);
        );
        ```

        - 方式 2：
  
        ```java
        FileInputStream in=new FileInputStream(“d: /abc.txt”);
        ```

        - 方式 3:
  
        ```java
        FileDescriptor() fd=new FileDescriptor();
        FileInputStream f2=new FileInputStream(fd);
        ```

    - 程序对应的基本输入为键盘输入，基本输出为显示器输出。Java 中，System 类的 in 和 out 两个成员代表了基本输入输出的抽象
      **System.in**:基本输入，对应 InputStream
      **System.out**:基本输出，对应 PrintStream
    - **RandomAccessFile**
        - RandomAccessFile 类可以在文件中==任何位置==查找或写入数据
        - RandomAccessFile==同时实现了 DataInput 和 DataOutput 接口==
        - 磁盘文件都是可以随机访问的， 但是从网络而来的数据流却不是

12. **ByteArrayOutpuStream/ByteArrayInputStream**

    - 一对输入输出工具为我们提供了在内存中利用 byte[]进行缓冲流操作的工具
    - ByteArrayOutputStream 提供工具将内存中以串行序列存在的流式数据以一个字节为单位进行切分，形成一个 byte[]数组
    - 而 ByteArrayInputStream 则正好相反，提供工具将内存中的 byte[]数组中的数据进行串行序列化拼接，形成一个可供操作的流式数据
    - 从功能上看，ByteArrayOutpuStream 可以将任意数据组合转换为 byte[]，而 ByteArrayInputStream 可以将这个数组还原，从而以流的形式读取任意数据组合

13. **字符输出流**

    - 考虑到 Java 是跨平台的语言，要经常操作 Unicode 编码的文件，使用基于字符为读、写基本单元的字符流操作文件是有必要的,以字符为单位进行数据输出的工具继承自 Writer

14. **字符输出流的统一数据写入方法**

    - Writer 和 OutputStream 类似也提供了统一的往流中写入数据的方法，和 OutputStream 不同的是，写入数据的单位由字节变成了字符

15. **字符输出流工具的作用与使用**

    - FileWriter 类称为文件写入流，以字符流的形式对文件进行写操作
    - FileWriter 将逐个向文件写入字符，效率比较低下，因此一般将该类对象包装到缓冲流中进行操作
    - 还可以使用 PrintWriter 对流进行包装，提供更方便的字符输出格式控制

16. **字符输入流**

    - 以字符为单位进行数据读取的工具继承自 Reader，Reader 会将读取到的数据按照标准的规则转换为 Java 字符串对象

17. **字符输入流的统一数据读取方法**

    - 字符输入流 Reader 也提供的统一读取数据的方法（和 InputStream 不同，实际开发时更多的调用不同 Reader 提供的特殊读取方法，如 BufferedReader 的 readLine()，能够简化操作）

18. **常见的字符输入流工具的作用与使用**

    - FileReader 类称为文件读取流，允许以字符流的形式对文件进行读操作
    - 与 FileWriter 相似，该类将从文件中逐个地读取字符，效率比较低下，因此一般也将该类对象包装到缓冲流中进行操作

19. **字节流与字符流的适配器**

    - 在某些时候虽然我们操作的是字符串，但是不得不面对数据来源是 InputStream（字节输入流）的情况，在这种情况下，Java 提供了将 InputStream 和 Reader 之间进行转换的工具，事实上，字节输出流和字符输出流之间也存在这种工具，称为：字节流与字符流的适配器：
      - InputStreamReader： + 字节流通向字符流的桥梁，它使用指定的 charset 读取字节并将其解码为字符。它使用的字符集可以由名称指定或显式给定，或者可以接受平台默认的字符集 + 每次调用 InputStreamReader 中的一个 read() 方法都会导致从底层输入流读取一个或多个字节。要启用从字节到字符的有效转换，可以提前从底层流读取更多的字节，使其超过满足当前读取操作所需的字节 + OutputStreamWriter： + 字符流通向字节流的桥梁，使用指定的 charset 将要写入流中的字符编码成字节。它使用的字符集可以由名称指定或显式给定，否则将接受平台默认的字符集 + 每次调用 write() 方法都会导致在给定字符（或字符集）上调用编码转换器。在写入底层输出流之前，得到的这些字节将在缓冲区中累积。可以指定此缓冲区的大小，不过，默认的缓冲区对多数用途来说已足够大。注意，传递给 write() 方法的字符没有缓冲

#### 第 3 节:对象序列化

Java 平台允许我们在内存中创建可复用的 Java 对象，但一般情况下，只有当 JVM 处于运行时，这些对象才可能存在，即，这些对象的生命周期不会比 JVM 的生命周期更长。但在现实应用中，就可能要求在 JVM 停止运行之后能够保存(持久化)指定的对象，并在将来重新读取被保存的对象。Java 对象序列化就能够帮助我们实现该功能

1. 对象序列化的作用

   - 使用 Java 对象序列化，在保存对象时，会把其状态保存为一组字节，在未来，再将这些字节组装成对象。必须注意地是，对象序列化保存的是对象的"状态"，即它的成员变量。由此可知，对象序列化不会关注类中的静态变量
   - 除了在持久化对象时会用到对象序列化之外，在网络中传递对象时，也会用到对象序列化。Java 序列化 API 为处理对象序列化提供了一个标准机制

2. 序列化接口

   - 在 Java 中，只要一个类实现了`java.io.Serializable`接口，那么它就可以被序列化
   - `java.io.Serializable`是一个标识接口，即意味着它仅仅是为了说明类的可序列化属性，接口没有包含任何需要子类实现的抽象方法

3. 对象序列化和反序列化

   - 将对象的状态信息保存到流中的操作，称为序列化，可以使用 Java 提供的工具`ObjectOutputStream. writeObject(Serializable obj)`来完成
   - 从流中读取对心状态信息的操作称为反序列化，可以使用 Java 提供的工具`ObjectInputStream.readObject()`来完成
