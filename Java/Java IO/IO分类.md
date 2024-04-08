- [IO 流](#io-流)
- [传输方式 划分](#传输方式-划分)
  - [InputStream 类](#inputstream-类)
  - [OutputStream 类](#outputstream-类)
  - [Reader 类](#reader-类)
  - [Writer 类](#writer-类)
- [操作对象 划分](#操作对象-划分)
  - [文件](#文件)
  - [内存（数组）](#内存数组)
  - [管道](#管道)
  - [基本数据类型](#基本数据类型)
  - [对象](#对象)
  - [缓冲](#缓冲)

# IO 流

字节流和字符流的区别：

字节流一般用来处理图像、视频、音频、PPT、Word等类型的文件。字符流一般用于处理纯文本类型的文件，如TXT文件等，但不能处理图像视频等非文本文件。用一句话说就是：字节流可以处理一切文件，而字符流只能处理纯文本文件。

字节流本身没有缓冲区，缓冲字节流相对于字节流，效率提升非常高。而字符流本身就带有缓冲区，缓冲字符流相对于字符流效率提升就不是那么大了。

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405271042299.png)


# 传输方式 划分

## InputStream 类

```Java
int read()：读取数据
int read(byte b[], int off, int len)：从第 off 位置开始读，读取 len 长度的字节，然后放入数组 b 中
long skip(long n)：跳过指定个数的字节
int available()：返回可读的字节数
void close()：关闭流，释放资源
```

## OutputStream 类

```java
void write(int b)： 写入一个字节，虽然参数是一个 int 类型，但只有低 8 位才会写入，高 24 位会舍弃（这块后面再讲）
void write(byte b[], int off, int len)： 将数组 b 中的从 off 位置开始，长度为 len 的字节写入
void flush()： 强制刷新，将缓冲区的数据写入
void close()：关闭流
```

## Reader 类

```Java
int read()：读取单个字符
int read(char cbuf[], int off, int len)：从第 off 位置开始读，读取 len 长度的字符，然后放入数组 b 中
long skip(long n)：跳过指定个数的字符
int ready()：是否可以读了
void close()：关闭流，释放资源
```

## Writer 类

```Java
void write(int c)： 写入一个字符
void write( char cbuf[], int off, int len)： 将数组 cbuf 中的从 off 位置开始，长度为 len 的字符写入
void flush()： 强制刷新，将缓冲区的数据写入
void close()：关闭流
```

```Java
'''字符流带缓冲区'''
// OutputStreamWriter 类的 write 方法
// 声明一个 char 类型的数组，用于写入输出流
private char[] writeBuffer;

// 定义 writeBuffer 数组的大小，必须 >= 1
private static final int WRITE_BUFFER_SIZE = 1024;

// 写入给定字符串中的一部分到输出流中
public void write(String str, int off, int len) throws IOException {
    // 使用 synchronized 关键字同步代码块，确保线程安全
    synchronized (lock) {
        char cbuf[];
        // 如果 len <= WRITE_BUFFER_SIZE，则使用 writeBuffer 数组进行写入
        if (len <= WRITE_BUFFER_SIZE) {
            // 如果 writeBuffer 为 null，则创建一个大小为 WRITE_BUFFER_SIZE 的新 char 数组
            if (writeBuffer == null) {
                writeBuffer = new char[WRITE_BUFFER_SIZE];
            }
            cbuf = writeBuffer;
        } else {    // 如果 len > WRITE_BUFFER_SIZE，则不永久分配非常大的缓冲区
            // 创建一个大小为 len 的新 char 数组
            cbuf = new char[len];
        }
        // 将 str 中的一部分（从 off 开始，长度为 len）拷贝到 cbuf 数组中
        str.getChars(off, (off + len), cbuf, 0);
        // 将 cbuf 数组中的数据写入输出流中
        write(cbuf, 0, len);
    }
}
```

```java
// 字节流
try (FileInputStream fis = new FileInputStream("input.txt");
     FileOutputStream fos = new FileOutputStream("output.txt")) {
    byte[] buffer = new byte[1024];
    int len;
    while ((len = fis.read(buffer)) != -1) {
        fos.write(buffer, 0, len);
    }
} catch (IOException e) {
    e.printStackTrace();
}

// 字符流
try (FileReader fr = new FileReader("input.txt");
     FileWriter fw = new FileWriter("output.txt")) {
    char[] buffer = new char[1024];
    int len;
    while ((len = fr.read(buffer)) != -1) {
        fw.write(buffer, 0, len);
    }
} catch (IOException e) {
    e.printStackTrace();
}
```

# 操作对象 划分

![](https://gitee.com/wanglongxin666/pictures/raw/master/img/202405271111426.png)

## 文件

文件流也就是直接操作文件的流，可以细分为字节流（FileInputStream 和 FileOuputStream）和字符流（FileReader 和 FileWriter）。

```Java
'''字节流'''
// 声明一个 int 类型的变量 b，用于存储读取到的字节
int b;
// 创建一个 FileInputStream 对象，用于读取文件 fis.txt 中的数据
FileInputStream fis1 = new FileInputStream("fis.txt");

// 循环读取文件中的数据
while ((b = fis1.read()) != -1) {
    // 将读取到的字节转换为对应的 ASCII 字符，并输出到控制台
    System.out.println((char)b);
}

// 关闭 FileInputStream 对象，释放资源
fis1.close();


// 创建一个 FileOutputStream 对象，用于写入数据到文件 fos.txt 中
FileOutputStream fos = new FileOutputStream("fos.txt");

// 向文件中写入数据，这里写入的是字符串 "沉默王二" 对应的字节数组
fos.write("沉默王二".getBytes());

// 关闭 FileOutputStream 对象，释放资源
fos.close();


'''字符流'''
// 声明一个 int 类型的变量 b，用于存储读取到的字符
int b = 0;

// 创建一个 FileReader 对象，用于读取文件 read.txt 中的数据
FileReader fileReader = new FileReader("read.txt");

// 循环读取文件中的数据
while ((b = fileReader.read()) != -1) {
    // 将读取到的字符强制转换为 char 类型，并输出到控制台
    System.out.println((char)b);
}

// 关闭 FileReader 对象，释放资源
fileReader.close();

// 创建一个 FileWriter 对象，用于写入数据到文件 fw.txt 中
FileWriter fileWriter = new FileWriter("fw.txt");

// 将字符串 "沉默王二" 转换为字符数组
char[] chars = "沉默王二".toCharArray();

// 向文件中写入数据，这里写入的是 chars 数组中的所有字符
fileWriter.write(chars, 0, chars.length);

// 关闭 FileWriter 对象，释放资源
fileWriter.close();
```

## 内存（数组）

数组流可以用于在内存中读写数据，比如将数据存储在字节数组中进行压缩、加密、序列化等操作。它的优点是不需要创建临时文件，可以提高程序的效率。但是，数组流也有缺点，它只能存储有限的数据量，如果存储的数据量过大，会导致内存溢出。

```Java
// 创建一个 ByteArrayInputStream 对象，用于从字节数组中读取数据
InputStream is = new BufferedInputStream(
        new ByteArrayInputStream(
                "沉默王二".getBytes(StandardCharsets.UTF_8)));

// 定义一个字节数组用于存储读取到的数据
byte[] flush = new byte[1024];

// 定义一个变量用于存储每次读取到的字节数
int len = 0;

// 循环读取字节数组中的数据，并输出到控制台
while (-1 != (len = is.read(flush))) {
    // 将读取到的字节转换为对应的字符串，并输出到控制台
    System.out.println(new String(flush, 0, len));
}

// 关闭输入流，释放资源
is.close();


// 创建一个 ByteArrayOutputStream 对象，用于写入数据到内存缓冲区中
ByteArrayOutputStream bos = new ByteArrayOutputStream();

// 定义一个字节数组用于存储要写入内存缓冲区中的数据
byte[] info = "沉默王二".getBytes();

// 向内存缓冲区中写入数据，这里写入的是 info 数组中的所有字节
bos.write(info, 0, info.length);

// 将内存缓冲区中的数据转换为字节数组
byte[] dest = bos.toByteArray();

// 关闭 ByteArrayOutputStream 对象，释放资源
bos.close();
```

## 管道

Java 中的管道和 Unix/Linux 中的管道不同，在 Unix/Linux 中，不同的进程之间可以通过管道来通信，但 Java 中，通信的双方必须在同一个进程中，也就是在同一个 JVM 中，管道为线程之间的通信提供了通信能力。

```Java
// 创建一个 PipedOutputStream 对象和一个 PipedInputStream 对象
final PipedOutputStream pipedOutputStream = new PipedOutputStream();
final PipedInputStream pipedInputStream = new PipedInputStream(pipedOutputStream);

// 创建一个线程，向 PipedOutputStream 中写入数据
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            // 将字符串 "沉默王二" 转换为字节数组，并写入到 PipedOutputStream 中
            pipedOutputStream.write("沉默王二".getBytes(StandardCharsets.UTF_8));
            // 关闭 PipedOutputStream，释放资源
            pipedOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
});

// 创建一个线程，从 PipedInputStream 中读取数据并输出到控制台
Thread thread2 = new Thread(new Runnable() {
    @Override
    public void run() {
        try {
            // 定义一个字节数组用于存储读取到的数据
            byte[] flush = new byte[1024];
            // 定义一个变量用于存储每次读取到的字节数
            int len = 0;
            // 循环读取字节数组中的数据，并输出到控制台
            while (-1 != (len = pipedInputStream.read(flush))) {
                // 将读取到的字节转换为对应的字符串，并输出到控制台
                System.out.println(new String(flush, 0, len));
            }
            // 关闭 PipedInputStream，释放资源
            pipedInputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
});

// 启动线程1和线程2
thread1.start();
thread2.start();
```

## 基本数据类型

```Java
// 创建一个 DataInputStream 对象，用于从文件中读取数据
DataInputStream dis = new DataInputStream(new FileInputStream("das.txt"));

// 读取一个字节，将其转换为 byte 类型
byte b = dis.readByte();

// 读取两个字节，将其转换为 short 类型
short s = dis.readShort();

// 读取四个字节，将其转换为 int 类型
int i = dis.readInt();

// 读取八个字节，将其转换为 long 类型
long l = dis.readLong();

// 读取四个字节，将其转换为 float 类型
float f = dis.readFloat();

// 读取八个字节，将其转换为 double 类型
double d = dis.readDouble();

// 读取一个字节，将其转换为 boolean 类型
boolean bb = dis.readBoolean();

// 读取两个字节，将其转换为 char 类型
char ch = dis.readChar();

// 关闭 DataInputStream，释放资源
dis.close();

// 创建一个 DataOutputStream 对象，用于将数据写入到文件中
DataOutputStream das = new DataOutputStream(new FileOutputStream("das.txt"));

// 将一个 byte 类型的数据写入到文件中
das.writeByte(10);

// 将一个 short 类型的数据写入到文件中
das.writeShort(100);

// 将一个 int 类型的数据写入到文件中
das.writeInt(1000);

// 将一个 long 类型的数据写入到文件中
das.writeLong(10000L);

// 将一个 float 类型的数据写入到文件中
das.writeFloat(12.34F);

// 将一个 double 类型的数据写入到文件中
das.writeDouble(12.56);

// 将一个 boolean 类型的数据写入到文件中
das.writeBoolean(true);

// 将一个 char 类型的数据写入到文件中
das.writeChar('A');

// 关闭 DataOutputStream，释放资源
das.close();
```

## 对象

```Java
public static void main(String[] args) {
    try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("person.dat"))) {
        Person p = new Person("张三", 20);
        oos.writeObject(p);
    } catch (IOException e) {
        e.printStackTrace();
    }

    try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("person.dat"))) {
        Person p = (Person) ois.readObject();
        System.out.println(p);
    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
    }
}
```

## 缓冲

```java
// 创建一个 BufferedInputStream 对象，用于从文件中读取数据
BufferedInputStream bis = new BufferedInputStream(new FileInputStream("data.txt"));

// 创建一个字节数组，作为缓存区
byte[] buffer = new byte[1024];

// 读取文件中的数据，并将其存储到缓存区中
int bytesRead;
while ((bytesRead = bis.read(buffer)) != -1) {
    // 对缓存区中的数据进行处理
    // 这里只是简单地将读取到的字节数组转换为字符串并打印出来
    System.out.println(new String(buffer, 0, bytesRead));
}

// 关闭 BufferedInputStream，释放资源
bis.close();

// 创建一个 BufferedOutputStream 对象，用于将数据写入到文件中
BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream("data.txt"));

// 创建一个字节数组，作为缓存区
byte[] buffer = new byte[1024];

// 将数据写入到文件中
String data = "沉默王二是个大傻子!";
buffer = data.getBytes();
bos.write(buffer);

// 刷新缓存区，将缓存区中的数据写入到文件中
bos.flush();

// 关闭 BufferedOutputStream，释放资源
bos.close();

// 创建一个 BufferedReader 对象，用于从文件中读取数据
BufferedReader br = new BufferedReader(new FileReader("data.txt"));

// 读取文件中的数据，并将其存储到字符串中
String line;
while ((line = br.readLine()) != null) {
    // 对读取到的数据进行处理
    // 这里只是简单地将读取到的每一行字符串打印出来
    System.out.println(line);
}

// 关闭 BufferedReader，释放资源
br.close();

// 创建一个 BufferedWriter 对象，用于将数据写入到文件中
BufferedWriter bw = new BufferedWriter(new FileWriter("data.txt"));

// 将数据写入到文件中
String data = "沉默王二，真帅气";
bw.write(data);

// 刷新缓存区，将缓存区中的数据写入到文件中
bw.flush();

// 关闭 BufferedWriter，释放资源
bw.close();
```
