# Java IO流示例

Java中对数据的操作是通过流的方式，IO流用来处理设备之间的数据传输，上传文件和下载文件。

## File操作示例：获取路径下的文件名

```java
// 获取文件路径的方法
public static void listChildes(File file,int level){
    // 生成有层次感的空格
    StringBuffer sb=new StringBuffer("|--");
    for (int i = 0; i < level; i++) {
        sb.insert(0,"| ");
    }
    File[] childes=file.listFiles();
    // 递归出口
    int length=childes==null?0:childes.length;
    for (int i = 0; i < length; i++) {
        System.out.println(sb.toString()+childes[i].getName());
        if (childes[i].isDirectory()){
            listChildes(childes[i],level+1);
        }
    }
}
```

## 概览

InputStream和Reader是所有输入流的抽象基类，本身并不能创建实例来执行输入，但它们将成为所有输入流的模板，所以它们的方法是所有输入流都可使用的方法。 
 在InputStream里面包含如下3个方法。

- int read(); 从输入流中读取单个字节，返回所读取的字节数据（字节数据可直接转换为int类型）。
- int read(byte[] b)；从输入流中最多读取b.length个字节的数据，并将其存储在字节数组b中，返回实际读取的字节数。
- int read(byte[] b,int off,int len); 从输入流中最多读取len个字节的数据，并将其存储在数组b中，放入数组b中时，并不是从数组起点开始，而是从off位置开始，返回实际读取的字节数。

在Reader中包含如下3个方法。

- int read(); 从输入流中读取单个字符（相当于从图15.5所示的水管中取出一滴水），返回所读取的字符数据（字节数据可直接转换为int类型）。
- int read(char[] b)从输入流中最多读取b.length个字符的数据，并将其存储在字节数组b中，返回实际读取的字符数。
- int read(char[] b,int off,int len); 从输入流中最多读取len个字符的数据，并将其存储在数组b中，放入数组b中时，并不是从数组起点开始，而是从off位置开始，返回实际读取的字符数。

InputStream和Reader提供的一些移动指针的方法：

- void mark(int readAheadLimit); 在记录指针当前位置记录一个标记（mark）。
- boolean markSupported(); 判断此输入流是否支持mark()操作，即是否支持记录标记。
- void reset(); 将此流的记录指针重新定位到上一次记录标记（mark）的位置。
- long skip(long n); 记录指针向前移动n个字节/字符。



OutputStream和Writer：

- void write(int c); 将指定的字节/字符输出到输出流中，其中c即可以代表字节，也可以代表字符。
- void write(byte[]/char[] buf); 将字节数组/字符数组中的数据输出到指定输出流中。
- void write(byte[]/char[] buf, int off,int len ); 将字节数组/字符数组中从off位置开始，长度为len的字节/字符输出到输出流中。

因为字符流直接以字符作为操作单位，所以Writer可以用字符串来代替字符数组，即以String对象作为参数。Writer里面还包含如下两个方法。

- void write(String str); 将str字符串里包含的字符输出到指定输出流中。
- void write (String str, int off, int len); 将str字符串里面从off位置开始，长度为len的字符输出到指定输出流中。

## 字节流

### 基类

#### InputStream

字节输入流基类，抽象类，提供的方法如下：

```java
// 从输入流中读取数据的下一个字节
abstract int read()
// 从输入流中读取一定数量的字节，并将其存储在缓冲区数组b中
int read(byte[] b)
// 从输入流中最多读取len个字节的数据，并将其存储在数组b中，放入数组b中时，
//  并不是从数组起点开始，而是从off位置开始，返回实际读取的字节数。
int read(byte[] b, int off, int len)
// 跳过和丢弃此输入流中数据的 n个字节
long skip(long n)
// 关闭此输入流并释放与该流关联的所有系统资源
void close()
```

#### OutputStream

字节输出流基类，抽象类，提供的方法如下：

```java
// 将 b.length 个字节从指定的 byte 数组写入此输出流
void write(byte[] b)
// 将指定 byte 数组中从偏移量 off 开始的 len 个字节写入此输出流
void write(byte[] b, int off, int len)
// 将指定的字节写入此输出流
abstract void write(int b)
// 关闭此输出流并释放与此流有关的所有系统资源
void close()
// 刷新此输出流并强制写出所有缓冲的输出字节
void flush()
```

### 字节文件操作流

#### FileInputStream

字节文件输入流，从系统中的某个文件获得输入字节，用于读取诸如图像数据之类的原始字节流。

构造方法：

- FileInputStream(File file)：通过打开一个到实际文件的连接来创建一个FileInputStream，该文件通过文件系统中的File对象file指定；
- FileInputStream(String name)：通过打开一个到实际文件的连接来创建一个FileInputStream，该文件通过文件系统中的路径name指定；

示例：

```java
// 一次读取一个字符数组
public static void output(File file,int i){
    FileInputStream is=null;
    // 字节数组
    byte[] b=new byte[i];
    try {
        // 1. 根据文件创建流
        is=new FileInputStream(file);
        int i1=0;
        // 2. 一次输出一个字节数组
        // 或一次读取一个字符is.read()
        while ((i1=is.read(b))!=-1){
            System.out.print(new String(b,0,i1,"UTF-8")+" ");
        }
    } catch (UnsupportedEncodingException e) {
        e.printStackTrace();
    } catch (FileNotFoundException e) {
        e.printStackTrace();
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        // 在finally中关闭流
        if (is!=null){
            try {
                is.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### FileOutputStream

字节文件输出流是用于将数据写入到File，从程序中写入到其他位置。

构造方法：

- FileOutputStream(File file)：创建一个向指定File对象表示的文件中写入数据的文件输出流；
- FileOutputStream(File file, boolean append)：创建一个向指定File对象表示的文件中写入数据的文件输出流；
- FileOutputStream(String name)：创建一个向具有指定名称的文件中写入数据的输出文件流；
- FileOutputStream(String name, boolean append)：创建一个向具有指定name的文件中写入数据的输出文件流；

示例：

```java
public static void writeFile(String filePath,String content,boolean isAppend){
    FileOutputStream fos=null;
    try {
        // 1. 根据文件路径创建输出流
        fos=new FileOutputStream(filePath,isAppend);
        // 2. 把String转换为byte数组
        byte[] array=content.getBytes();
        // 3. 把byte数组输出
        // 如果要写入换行符：最好写成\r\n
        // 输出的目的地文件不存在，则会自动创建，不指定盘符的话，
        //  默认创建在项目目录下;
        fos.write(array);
    }catch (FileNotFoundException e){
        e.printStackTrace();
    }catch (IOException e) {
        e.printStackTrace();
    }finally {
    	// 关闭流
        if (fos!=null){
            try {
                fos.close();
            }catch (IOException e){
                e.printStackTrace();
            }
        }
    }
}
```

### 字节缓冲流（高效流）

#### BufferedInputStream

字节缓冲输入流，提高了读取效率。

构造方法：

- BufferedInputStream(InputStream in)：创建一个 BufferedInputStream并保存其参数，即输入流in，以便将来使用；
- BufferedInputStream(InputStream in, int size)：创建具有指定缓冲区大小的 BufferedInputStream并保存其参数，即输入流in以便将来使用；

示例：

```java
public static void bufferedIn(InputStream in,int size) {
    BufferedInputStream bis=null;
    byte[] b=new byte[20];
    int len=0;
    try {
        // 创建流
        bis=new BufferedInputStream(in,size);
        // 一次输出一个字符数组
        while ((len=bis.read(b))!=-1){
            System.out.println(new String(b,0,len,"UTF-8"));
        }
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        if (bis!=null){
            try {
                bis.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

#### BufferedOutputStream

字节缓冲输出流，提高了写出效率。

构造方法：

- BufferedOutputStream(OutputStream out)：创建一个新的缓冲输出流，以将数据写入指定的底层输出流；
- BufferedOutputStream(OutputStream out, int size)：创建一个新的缓冲输出流，以将具有指定缓冲区大小的数据写入指定的底层输出流；

常用方法：

- void write(byte[] b, int off, int len)：将指定 byte 数组中从偏移量 off 开始的 len 个字节写入此缓冲的输出流；
- void write(int b)：将指定的字节写入此缓冲的输出流；
- void flush()：刷新此缓冲的输出流；

示例：

```java
public static void bufferedOut(OutputStream out,String content,int size){
    BufferedOutputStream bos=null;
    try {
        bos=new BufferedOutputStream(out, size) ;
        bos.write(content.getBytes());
        bos.write("\r\n".getBytes());
        // 刷新此缓冲的输出流
        bos.flush();
    } catch (IOException e) {
        e.printStackTrace();
    }finally {
        if(bos!=null){
            try {
                bos.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

测试：

```java
public static void main(String[] args) throws IOException {
    String content="世界如此美好";
    // 追加写入
    OutputStream out=new FileOutputStream(new File("hello.txt"),true);
    bufferedOut(out,content,20);
    InputStream in=new FileInputStream(new File("hello.txt"));
    bufferedIn(in,20);
}
```

## 字符流

### 基类

#### Reader

读取字符流的抽象类。

常用方法：

```java
// 从输入流中读取单个字符，返回所读取的字符数据（字节数据可直接转换为int类型）。
int read()
// 从输入流中最多读取b.length个字符的数据，并将其存储在字节数组b中，返回
// 实际读取的字符数。
int read(char[] cbuf)
// 从输入流中最多读取len个字符的数据，并将其存储在数组b中，放入数组b中时，
// 并不是从数组起点开始，而是从off位置开始，返回实际读取的字符数
abstract int read(char[] cbuf, int off, int len)
// 跳过字符
long skip(long n)
// 关闭该流并释放与之关联的所有资源
abstract void close()
```

#### Writer

写入字符流的抽象类。

常用方法：

```java
// 写入字符数组
void write(char[] cbuf)
// 写入字符数组的某一部分
abstract void write(char[] cbuf, int off, int len)
// 写入单个字符
void write(int c)
// 写入字符串
void write(String str)
// 写入字符串的某一部分
void write(String str, int off, int len)

// 将指定字符添加到此 writer
Writer append(char c)
// 将指定字符序列添加到此 writer
Writer append(CharSequence csq)
// 将指定字符序列的子序列添加到此 writer.Appendable
Writer append(CharSequence csq, int start, int end)

// 关闭此流，但要先刷新它
abstract void close()
// 刷新该流的缓冲
abstract void flush()
```

### 字符转换流

#### InputStreamReader

InputStreamReader：字节流转字符流，它使用的字符集可以由名称指定或显式给定，否则将接受平台默认的字符集。

构造方法：

```java
// 创建一个使用默认字符集的 InputStreamReader
InputStreamReader(InputStream in)
// 创建使用给定字符集的 InputStreamReader
InputStreamReader(InputStream in, Charset cs)
// 创建使用给定字符集解码器的 InputStreamReader
InputStreamReader(InputStream in, CharsetDecoder dec)
// 创建使用指定字符集的 InputStreamReader
InputStreamReader(InputStream in, String charsetName)
特有方法：
//返回此流使用的字符编码的名称 
String getEncoding() 
```

示例：

```java
// 字节字符转换流
public static Reader getReader(InputStream in, String encoding) throws UnsupportedEncodingException {
    return new InputStreamReader(in,encoding);
}
public static void main(String[] args) throws IOException {
    InputStream in=new FileInputStream(new File("hello.txt"));
    Reader reader=getReader(in,"UTF-8");
    char[] c=new char[20];
    int i=0;
    // 输出单个字符：int i=reader.read();  (char)i
    while((i=reader.read(c))!=-1){
        System.out.println(c);
        System.out.println("------------------");
    }
    reader.close();
}
```

#### OutputStreamWriter

OutputStreamWriter：字节流转字符流。

 构造方法：

```java

// 创建使用默认字符编码的 OutputStreamWriter
OutputStreamWriter(OutputStream out)
// 创建使用给定字符集的 OutputStreamWriter
OutputStreamWriter(OutputStream out, Charset cs)
// 创建使用给定字符集编码器的 OutputStreamWriter
OutputStreamWriter(OutputStream out, CharsetEncoder enc)
// 创建使用指定字符集的 OutputStreamWriter
OutputStreamWriter(OutputStream out, String charsetName)
特有方法：
//返回此流使用的字符编码的名称 
String getEncoding() 
```

示例：

```java
// 字符字节转换流
public static Writer getWriter(OutputStream out,String decoding) throws UnsupportedEncodingException {
    return new OutputStreamWriter(out,decoding);
}
public static void main(String[] args) throws IOException {
    String content="世界如此美好";
    OutputStream out=new FileOutputStream(new File("hello.txt"),true);
    Writer writer=getWriter(out,"UTF-8");

    writer.write(content);
    writer.append("，世界如此职棒");
    writer.flush();
    writer.close();
}
```

### 字符缓冲流

#### BufferedReader

BufferedReader：字符缓冲流，从字符输入流中读取文本，缓冲各个字符，从而实现字符、数组和行的高效读取。

构造方法：

```
// 创建一个使用默认大小输入缓冲区的缓冲字符输入流
BufferedReader(Reader in)
// 创建一个使用指定大小输入缓冲区的缓冲字符输入流
BufferedReader(Reader in, int sz)
特有方法：
// 读取一个文本行
String readLine()
```

示例：

```java
// 字节字符转换流
public static Reader getReader(InputStream in, String encoding) throws UnsupportedEncodingException {
    return new InputStreamReader(in,encoding);
}
public static void main(String[] args) throws IOException {
    InputStream in=new FileInputStream(new File("hello.txt"));
    BufferedReader reader=new BufferedReader(getReader(in,"UTF-8"));
    char[] c=new char[20];
    int i=0;
    // 输出单个字符：int i=reader.read();  (char)i
    while((i=reader.read(c))!=-1){
        System.out.println(c);
        System.out.println("------------------");
    }
    reader.close();
}
```

#### BufferedWriter

BufferedWriter：字符缓冲流，将文本写入字符输出流，缓冲各个字符，从而提供单个字符、数组和字符串的高效写入。

 构造方法：

```
// 创建一个使用默认大小输出缓冲区的缓冲字符输出流
BufferedWriter(Writer out)
// 创建一个使用给定大小输出缓冲区的新缓冲字符输出流
BufferedWriter(Writer out, int sz)
特有方法：
// 写入一个行分隔符
void newLine() 
```

示例：

```java
// 字符字节转换流
public static Writer getWriter(OutputStream out,String decoding) throws UnsupportedEncodingException {
    return new OutputStreamWriter(out,decoding);
}
public static void main(String[] args) throws IOException {
    String content="世界如此美好";
    OutputStream out=new FileOutputStream(new File("hello.txt"),true);
     BufferedWriter writer=new BufferedWriter(getWriter(out,"UTF-8"));

    writer.write(content);
    writer.append("，世界如此职棒");
    writer.flush();
    writer.close();
}
```

#### FileReader、FileWriter

- FileReader：InputStreamReader类的直接子类，用来读取字符文件的便捷类，使用默认字符编码。
- FileWriter：OutputStreamWriter类的直接子类，用来写入字符文件的便捷类，使用默认字符编码。