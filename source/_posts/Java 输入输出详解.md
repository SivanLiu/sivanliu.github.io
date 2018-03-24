
Java 的 IO 通过 java.io 包下的类和接口来支持，主要包括输入、输出两种 IO 流，每种输入、输出流又可分为字节流和字符流两大类。其中字节流以字节为单位来处理输入、输出操作，而字符流则以字符来处理输入、输出操作；另外，Java 的 IO 流还使用了一种装饰器设计模式，它将 IO 流分成底层节点流和上层处理流，其中节点流用于和底层的物理存储节点直接关联————不同的物理节点获取节点流的方式可能存在一定的差异，但程序可以把不同的物理节点流包装成统一的处理流，从而允许程序使用统一的输入、输出代码来读取不同的物理存储节点的资源。

# 一、File 类
File 类是 java.io 包下代表与平台无关的文件和目录，File 可以新建、删除、重命名文件和目录，File 不能访问文件内容本身，如需访问文件本身内容，需要使用输入/输出流。
## 1. 访问文件名的方法：
* String getName(): 返回次 File 对象所表示的文件名或路径名(如果是路径，则返回最后一级子路径名)
* String getPath(): 返回此 File 对象所对应的路径名
* File getAbsoluteFile(): 返回此 File 对象的绝对路径
* File getAbsolutePath(): 返回此 File 对象所对应的绝对路径名
* String getParent(): 返回此 File 对象所对应目录(最后一级)的父目录名
* boolean renameTo(File newName): 重命名此 File 对象所对应的文件或目录，如果命名成功，则返回 true，否则返回 false

## 2. 文件检测相关方法：
* boolean exist(): 判断 File 对象所对应的文件或者目录是否存在
* boolean canWrite(): 判断 File 对象所对应的文件或者目录是否可写
* boolean canRead(): 判断 File 对象所对应的文件或目录是否可读
* boolean isFile(): 判断 File 对象所对应的是否是文件，而不是目录
* boolean isDirectory(): 判断 File 对象所对应的是否是目录，而不是文件
* boolean isAbsolute(): 判断 File 对象所对应的文件或目录是否是绝对路径(linux 路径开头是一条 /，则是绝对路径，Windows 路径开头是盘符，则是绝对路径)

## 3. 获取常规文件信息：
* long lastModified(): 返回文件的最后修改时间
* long length(): 返回文件内容的长度

## 4. 文件操作相关方法：
* boolean createNewFile(): 当此 File 对象所对应的文件不存在时，该方法将新建一个该 File 对象所指定的新文件，如果创建成功则返回 true，否则返回 false
* boolean delete(): 删除 File 对象所对应的文件或者路径
* static File createTempFile(String prefix，String suffix): 在默认的临时文件目录中创建一个临时的空文件，使用指定前缀、系统生成的随机数和给定后缀作为文件名(prefix 参数必须是 3 字节长，如 "siv"、"mail"，suffix 参数可以为 null，此时将使用默认的后缀 "tmp")
* static File createTempFile(String prefix，String suffix，File directory): 在 directory 所指定的目录中创建一个临时的空文件，使用给定前缀、系统生成的随机数和给定后缀作为文件名。
* void deleteOnExit(): 注册一个删除钩子，指定当 Java 虚拟机退出时，删除 File 对象所对应的文件和目录。

## 5. 目录操作相关方法：
* boolean mkdir(): 试图创建一个 File 对象所对应的目录，如果创建成功，则返回 true，否则返回 false，调用该方法时 File 对象必须对应一个路径，而不是一个文件
* String[] list(): 列出 File 对象的所有子文件名和路径名，返回 String[] 数组
* File listFiles(): 列出 File 对象的所有子文件和路径，返回 File 数组
* static File[] listRoots(): 列出系统所有的根路径。

## 6. 下面写个 Demo 测试下 File 类的功能:
* **示例**：
```java
public class FileTest {
    public static void main(String[] args) {
        File file = new File("out/FileTest");
        file.mkdirs();
        System.out.println("sivan name = " + file.getName());
        System.out.println("sivan parent = " + file.getParent());
        System.out.println("sivan absoluteFile = " + file.getAbsoluteFile());
        System.out.println("sivan absoluteFile parent = " + file.getAbsoluteFile().getParent());
        System.out.println("sivan absolutePath = " + file.getAbsolutePath());

        try {
            File tmpFile = File.createTempFile("sivan", ".txt", file);
            tmpFile.deleteOnExit();
            System.out.println("sivan tmpFile = " + tmpFile + " exist = " + tmpFile.exists());

            File newFile = new File(System.currentTimeMillis() + "");
            System.out.println("sivan newFile exist = " + newFile.exists());
            boolean create = newFile.createNewFile();
            System.out.println("sivan newFile create = " + create);
            boolean mkNewFile = newFile.mkdir();
            System.out.println("sivan mkNewFile = " + mkNewFile);

            File file2 = new File(file, "file2");
            file2.createNewFile();
            String[] list = file.list();
            if (null != list) {
                System.out.println("========当前路径下的所有文件和路径如下========");
                for (String fileName : list) {
                    System.out.println("sivan fileName = " + fileName);
                }
            }

            File[] roots = File.listRoots();
            System.out.println("*********系统所有根路径如下*********");
            if (null != roots) {
                for (File root : roots) {
                    System.out.println("sivan root = " + root);
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
* **结果**：

```groovy
sivan name = FileTest
sivan parent = out
sivan absoluteFile = /Users/sivanliu/Downloads/DemoTest/out/FileTest
sivan absoluteFile parent = /Users/sivanliu/Downloads/DemoTest/out
sivan absolutePath = /Users/sivanliu/Downloads/DemoTest/out/FileTest
sivan tmpFile = out/FileTest/sivan1008697632080559742.txt exist = true
sivan newFile exist = false
sivan newFile create = true
sivan mkNewFile = false
========当前路径下的所有文件和路径如下========
sivan fileName = sivan1008697632080559742.txt
sivan fileName = file2
*********系统所有根路径如下*********
sivan root = /

Process finished with exit code 0
```

# 二、文件过滤器
在 File 类的 list() 方法中可以接收一个 FileNameFilter 参数，通过该参数可以只列出符合条件的文件，FileNameFilter 接口里只包含了一个 accept(File dir，String name) 方法，该方法将对指定 File 的所有子目录或者文件进行迭代，如果该方法返回 true，则 list() 方法只列出该子目录或者文件

* **测试 Demo:**

```java
public class FileNameFilterTest {
    public static void main(String[] args) {
        File file = new File("out/FileTest");

        String[] nameList = file.list((dir, name) -> {
            System.out.println("dir = " + dir + " name = " + name);
            return name.endsWith(".txt") || new File(dir, name).isDirectory();
        });

        System.out.println("============== filter result ==============");
        if (null != nameList) {
            for (String name : nameList) {
                System.out.println("sivan FileNameFilter name = " + name);
            }
        }
    }
}
```
* **结果：**

```groovy
dir = out/FileTest name = test
dir = out/FileTest name = test.txt
dir = out/FileTest name = test3
dir = out/FileTest name = test1.sh
============== filter result ==============
sivan FileNameFilter name = test
sivan FileNameFilter name = test.txt
sivan FileNameFilter name = test3

Process finished with exit code 0
```

# 三、Java 的 IO 流
Java 的 IO 流是实现输入输出的基础，可以方便地实现数据的输入、输出操作，在 Java 中把不同的输入、输出源抽象表述为 "流"，通过的流的方式允许 Java 程序使用相同的方式来访问不同的输入、输出源
## 1. 流的分类：
### a. 输入流和输出流：
按照流的流向来分，可以分为输入流和输出流：
* 输入流(InputStream/Reader)：只能从中读取数据，而不能向其写入数据
* 输出流(OutputStream/Writer)：只能向其中写入数据，而不能从中读取数据
### b. 字节流和字符流：
* 字节流和字符流的用法几乎完全一样，区别在于字节流和字符流的数据单元不同————字节流操作的数据单元是 8 位的字节，而字符流操作的数据单元是 16 位的字符
* 字节流主要由 InputStream 和 OutputStream 作为基类，而字符流主要由 Reader 和 Writer 作为基类
### c. 节点流和处理流：
按照流的角色来分，可以分为节点流和处理流：
* 可以从/向一个特定的 IO 设备(如磁盘、网络)读/写数据的流，称为节点流，节点流也被称为低级流
* 处理流则用于对一个已存在的流进行连接或封装，通过封装后的流来实现读/写功能，处理流也被称为高级流

## 2. 字节流和字符流
### 2.1 InputStream 和 Reader:
InputStream 和 Reader 是所有输入流的抽象基类，本身并不能创建实例来执行输入，但它们将成为所有输入流的模板，所以它们的方法是所有输入流都可以使用的方法。
**在 InputStream 里包含如下三个方法：**
* int read(): 从输入流中读取单个字节，返回所读取的字节数据
* int read(byte[] b): 从输入流中最多读取 b.length 个字节的数据，并将其存储在字节数组 b 中，返回时机读取的字节数
* int read(byte[]，int off，int len)：从输入流中最多读取 len 个字节的数据，并将其存储在数组 b 中，放入数组 b 中时，并不是从数组起点开始，而是从 off 位置开始，返回实际读取的字节数

**在 Reader 里包含如下三个方法**：
* int read(): 从输入流中读取单个字符，返回所读取的字符数据
* int read(char[] cbuf): 从输入流中最多读取 cbuf。length 个字符的数据，并将其存储在字符数组 cbuf 中，返回实际读取的字符数
* int read(char[] cbuf，int off，int len): 从输入流中最多读取 len 个字符的数据，并将其存储在字符数组 cbuf 中，放入数组 cbuf 中时，并不是从数组起点开始，而是从 off 位置开始，返回实际读取的字符数

**下面看看 FileInputStream 读取文件的示例：**

```java
public class FileInputStreamTest {
    public static void main(String[] args) {
        FileInputStream fileInputStream = null;
        try {
            fileInputStream = new FileInputStream("out/FileTest/test.txt");
            byte[] bytes = new byte[1024];
            int hasRead;
            while ((hasRead = fileInputStream.read(bytes)) > 0) {
                System.out.println("sivan fileInputStream result = \n" + new String(bytes, 0, hasRead));
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
**结果如下：**

```groovy
sivan fileInputStream result = 
FileInputStreamTest test test test
FileInputStreamTest test test test
FileInputStreamTest test test test
```

**下面对比下 FileReader 示例：**

```java
public class FileReaderTest {
    public static void main(String[] args) {
        FileReader fileReader = null;
        try {
            fileReader = new FileReader("src/out/test.txt");
            char[] chars = new char[32];

            int hasRead;

            while ((hasRead = fileReader.read(chars)) != 0) {
                System.out.println("sivan fileReader = " + new String(chars, 0, hasRead));
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (null != fileReader) {
                    fileReader.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

**结果如下：**

```groovy
sivan fileInputStream result = 
FileInputStreamTest test test test
FileInputStreamTest test test test
FileInputStreamTest test test test
```
### 2.2 OutputStream 和 Writer

**OutputStream 和 Writer 也非常类似，两个流都提供了如下三个方法：**

* void wirte(int c): 将指定的字节/字符流输出到输出流中，其中 c 既可以代表字节，也可以代表字符
* void write(byte[]/char[] buf): 将字节数组/字符数组中的数据输出到指定输出流中
* void write(byte[]/char[] buf，int off，int len): 将字节数组/字符数组中从 off 位置开始，长度为 len 的字节/字符输出到输出流中

因为字符流直接以字符作为操作单位，所以 writer 可以用字符来代替字符数组，即以 String 对象作为参数。Writer 里还包含如下两个方法：
* void write(String str): 将 str 字符串里包含的字符输出到指定的输出流中
* void write(String str，int off，int len): 将 str 字符串里从 off 位置开始，长度为 len 的字符输出到指定输出流中


**下面看下 FileOutputStream 中的示例：**

```java
public class FileOutputStreamTest {
    public static void main(String[] args) {
        FileOutputStream fileOutputStream = null;
        FileInputStream fileInputStream = null;
        try {
            fileInputStream = new FileInputStream("out/FileTest/test.txt");
            fileOutputStream = new FileOutputStream("out/FileTest/test_out.txt");
            byte[] bytes = new byte[32];
            int hasRead;
            while ((hasRead = fileInputStream.read(bytes)) > 0) {
                fileOutputStream.write(bytes, 0, hasRead);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileOutputStream != null) {
                try {
                    fileOutputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

**结果中可以看到自动生成了 test_out.txt 文件，并且内容与 test.txt 一致**

**下面看下 FileWriter 示例：**

```java
public class FileWriterTest {
    public static void main(String[] args) {
        FileWriter fileWriter = null;
        try {
            fileWriter = new FileWriter("out/FileTest/test_writer.txt");
            fileWriter.write("sivan sivan sivan \n");
            fileWriter.write("sivan sivan sivan \n");
            fileWriter.write("sivan sivan sivan \n");
            fileWriter.write("sivan sivan sivan \n");
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(null!=fileWriter){
                try {
                    fileWriter.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

**结果是生成的 test_writer 中的内容与代码中写入的内容一致**

## 3. 输入、输出流体系
### 3.1 处理流用法：

处理流的主要用法就是用来包装节点流，程序通过处理流来执行输入/输出功能，让节点流与底层的 IO 设备、文件交互，实际识别处理流非常简单，只要流的构造器参数不是一个物理节点，而是已经存在的流，那么这种流就一定是处理流，而所有节点流都是直接以物理 IO 节点作为构造器参数的。

**处理流优点主要有以下两个方面：**
* 处理流输入/输出操作更加简单；
* 使用处理流的执行效率更高；

下面看看示例：

```java
public class PrintStreamTest {
    public static void main(String[] args) {
        PrintStream printStream = null;
        FileOutputStream fileOutputStream = null;
        try {
            fileOutputStream = new FileOutputStream("out/FileTest/test_print.txt");
            printStream = new PrintStream(fileOutputStream);
            printStream.println("sivan printstream");
            printStream.println(new PrintStreamTest());
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (null != printStream) {
                printStream.close();
            }
        }
    }
}
```
**结果生成了 test_print.txt，并且其中的内容是刚输出的内容**

通常要输出文本内容，都应该将输出流包装成 PrintStream 后进行输出

### 3.2 输入/输出流体系
Java 的输入/输出流体系提供了近 40 个类，按照功能分类，大概可以得到下面的表格结果

| 分类 | 字节输入流 | 字节输出流 | 字符输入流 | 字符输出流 |
| :--- | :--- | :--- | :--- | :--- |
| 抽象基类 | *InputStream* | *OutputStream* | *Reader* | *Writer* |
| 访问文件 | **FileInputStream** | **FileOutputStream** | **FileReader** | **FileWriter** |
| 访问数组 | **ByteArrayInputStream** | **ByteArrayOutputStream** | **CharArrayReader** | **CharArrayWriter** |
| 访问管道 | **PipedInputStream** | **PipedOutputStream** | **PipedReader** | **PipedWriter** |
| 访问字符串 |  |  | **StringReader** | **StringWriter** |
| 缓冲流 | BufferedInputStream | BufferedOutputStream | BufferedReader | BufferdWriter |
| 转换流 |  |  | InputStreamReader | OutputStreamWriter |
| 对象流 | ObjectInputStream | ObjectOutputStream |  |  |
| 抽象基类 | *FilterInputStream* | *FilterOutputStream* | *FilterReader* | *FilterWriter* |
| 打印流 |  | PrintStream |  | PrintWriter |
| 推回输入流 | PushbackInputStream |  | PushbackReader |  |
| 特殊流 | DataInputStream | DataOutputStream |  |  |

其中粗体字标出来的类代表节点流，必须直接与指定的物理节点关联；斜体字为抽象基类，无法直接创建实例

## 4. 转换流
输入/输出流体系中还提供了两个转换流，这两个转换流用于实现将字节流转换成字符流，其中 InputStreamReader 将字节输入流转换成字符输入流，OutputStreamWriter 将字节输出流转换成字符输出流

**下面看一个转换流的示例：**

```java
public class ConvertStreamTest {
    public static void main(String[] args) {
        InputStreamReader reader = new InputStreamReader(System.in);
        BufferedReader bufferedReader = new BufferedReader(reader);
        String line;
        try {
            while ((line = bufferedReader.readLine()) != null) {
                if ("exit".equals(line)) {
                    System.exit(1);
                }
                System.out.println("输入内容为： " + line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                bufferedReader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```
Java 中 System.in 代表标准输入，但这个标准输入流是 InputStream 类的实例，使用不太方便，而且键盘输入内容都是文本内容，所以可以使用 InputStreamReader 将其转换成字符输入流，普通的 Reader 读取输入内容依然不太方便，可以将普通的 Reader 再次包装成 BufferedReader，利用 BufferedReader 的 readLine() 方法可以一次读取一行。

**结果：**

```groovy
test test test 
输入内容为： test test test 
exit

Process finished with exit code 1
```

## 5. 推回输入流

在输入/输出流体系中，有两个特殊的流与众不同，就是 PushbackInputStream 和 PushbackReader，它们也提供了三个方法：
* void unreade(byte[]/char[] buf): 将一个字节/字符数组内容推回到缓冲区里，从而允许重复读取刚刚读取的内容
* void unreade(byte[]/char[] buf，int off，int len): 将一个字节/字符数组内容推回到数组里从 off 开始，长度为 len 字节/字符的内容退回到缓冲区里，从而允许重复读取刚刚读取的内容
* void unread(int b): 将一个字节/字符推回到缓冲区里，从而允许重复读取刚刚读取到的内容

**看看示例如何使用：**

```java
public class PushbackTest {
    public static void main(String[] args) {
        PushbackReader pushbackReader = null;
        try {
            pushbackReader = new PushbackReader(new FileReader("out/FileTest/test.txt"),128);
            char[] buf = new char[32];
            String lastContent = "";
            int hasRead;

            while ((hasRead = pushbackReader.read(buf)) > 0) {
                String content = new String(buf, 0, hasRead);
                int targetIndex;
                if ((targetIndex = (lastContent + content).indexOf("test")) > 0) {
                    pushbackReader.unread((lastContent + content).toCharArray());

                    if (targetIndex > 32) {
                        buf = new char[targetIndex];
                    }

                    pushbackReader.read(buf, 0, targetIndex);
                    System.out.println("sivan PushbackTest " + new String(buf, 0, targetIndex));
                    System.exit(0);
                } else {
                    System.out.println("sivan lasContent = " +lastContent);
                    lastContent = content;
                }
            }

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (null != pushbackReader) {
                    pushbackReader.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }

        }
    }
}

```

**结果如下：**

```groovy
sivan lasContent = 
sivan lasContent = test testnew PushbackReader  tes
sivan PushbackTest t 

Process finished with exit code 0
```

## 6. 重定向标准输入/输出

Java 标准输入输出分别通过 System.in 和 System.out 来代表，System 类中提供三个重定向输入/输出标准的方法:
* static void setErr(PrintStream err): 重定向"标准"错误输出流
* static void setIn(InputStream in): 重定向"标准"输入流
* static void setOut(PrintStream out): 重定向"标准"输出流

**下面看看示例：**

```java
public class RedirectTest {
    public static void main(String[] args) {
        PrintStream printStream = null;
        try {
            printStream = new PrintStream(new FileOutputStream("out/FileTest/out.txt"));
            System.setOut(printStream);
            System.out.println("sivan Redirect 11.....");
            System.out.println("sivan Redirect 22....");
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (null != printStream) {
                printStream.close();
            }
        }
    }
}
```

**结果是所有输入都会保存在 out.txt 中**

```java
public class RedirectOut {
    public static void main(String[] args) {
        FileInputStream fileInputStream = null;
        try {
            fileInputStream = new FileInputStream("out/FileTest/test.txt");
            System.setIn(fileInputStream);
            Scanner scanner = new Scanner(System.in);
            scanner.useDelimiter("\n");
            while (scanner.hasNext()) {
                System.out.println("Your input is " + scanner.next());
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } finally {
            if (null != fileInputStream) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

**结果：**

```groovy
Your input is test testnew PushbackReader  test testFileInputStreamTesttest
Your input is FileInputStreamTest test test testnew PushbackReader
Your input is FileInputStreamTest test test testnew PushbackReader
```

## 7. RandomAccessFile
RandomAccessFile 是 Java 输入输出流体系中功能最丰富的文件内容访问类，它提供了众多的方法来访问文件内容，它即可以读取文件内容，也可以向文件输出数据。与普通的输入/输出流不同的是，RandomAccessFile 支持”随机访问”的方式，程序可以直接跳转到文件的任意地方来读写数据。它的最大局限是只能读写文件，不能读写其他 IO 节点；RandomAccessFile 对象包含一个记录指针，用以标识当前读写处的位置，当程序创建一个新的 RandomAccessFile 对象时，该对象的文件记录指针对于文件头（也就是 0 处），当读写 n 个字节后，文件记录指针将会向后移动 n 个字节。

RandomAccessFile 包含两个方法来操作文件记录指针：
* long getFilePointer():返回文件记录指针的当前位置
* void seek(long pos): 将文件记录指针定位到 pos 位置

RandomAccessFile 类在创建对象时，除了指定文件本身，还需要指定一个 mode 参数，该参数指定 RandomAccessFile 的访问模式，该参数有如下四个值：
* r：以只读方式打开指定文件。如果试图对该 RandomAccessFile 指定的文件执行写入方法则会抛出 IOException
* rw：以读取、写入方式打开指定文件。如果该文件不存在，则尝试创建文件
* rws：以读取、写入方式打开指定文件。相对于 rw 模式，还要求对文件的内容或元数据的每个更新都同步写入到底层存储设备
* rwd：以读取、写入方式打开指定文件。相对于 rw 模式，还要求对文件的内容的每个更新都同步写入到底层存储设备

下面看看示例如何使用 RandomAccessFile

```java
public class RandomAccessFileTest {
    public static void main(String[] args) {
        addContent("out/FileTest/out.txt", 0, "sivan sivan sivan");
    }

    private static void addContent(String fileName, long pos, String content) {
        File tmp = null;
        FileOutputStream outputStream = null;
        FileInputStream inputStream = null;
        try {
            tmp = File.createTempFile("tmp", null);
            tmp.deleteOnExit();

            RandomAccessFile randomAccessFile = new RandomAccessFile(fileName, "rw");
            outputStream = new FileOutputStream(tmp);
            inputStream = new FileInputStream(tmp);
            randomAccessFile.seek(pos);
            byte[] buf = new byte[64];
            int hasRead;
            while ((hasRead = randomAccessFile.read(buf)) > 0) {
                outputStream.write(buf, 0, hasRead);
            }
            randomAccessFile.seek(pos);
            randomAccessFile.write(content.getBytes());
            while ((hasRead = inputStream.read(buf)) > 0) {
                randomAccessFile.write(buf, 0, hasRead);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {

            if(null!=outputStream){
                try {
                    outputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }

            if(null!=inputStream){
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```

**结果：**

```groovy
nnnnnnnnnnsivan sivan sivanxxxxxxxxxxxxxhhhhhhhhhsxxxffnnfnnanfnxxx
```
关于文件操作的就先写到这里了，后续再把遇到的坑补上！

