# 类装载器ClassLoader

## 类的生命周期

在Java中，从类的生命周期而言，一个类包含如下阶段：

1. 装载（Loading）：查找和导入Class文件； 
2. 链接（Linking）：执行校验、准备和解析步骤，其中解析步骤是可以选择的：
   - 校验（Verification）：检查载入Class文件数据的正确性； 
   - 准备（Preparation）：给类的静态变量分配存储空间；
   - 解析（Resolution）：将符号引用转成直接引用；
3. 初始化（initialization）：对类的静态变量、静态代码块执行初始化工作。
4. 使用（Using）：对类执行操作；
5. 卸载（Unloading）：卸载类。

## 类加载

### 加载

加载(Loading)指的是将类的class文件读入到内存，并为之创建一个java.lang.Class对象，也就是说，

- 当程序中使用任何类时，系统都会为之建立一个java.lang.Class对象；
- 一旦一个类被加载到 JVM 中，同一个类就不会被再次载入了；
- 就像一个对象有一个唯一的标识一样，一个载入JVM的类也有一个唯一的标识；
  - 在 Java 中，一个类用其**全限定类名（包括包名和类名）**作为标识；
  - 在 JVM 中，一个类用其全限定类名和其类加载器作为其唯一标识；

### 类加载器ClassLoader

类装载工作由ClassLoader及其子类负责，ClassLoader是一个重要的Java运行时系统组件，它负责在运行时查找和装入Class字节码文件。JVM在运行时会产生三个ClassLoader：**根装载器**、**ExtClassLoader**（扩展类装载器）和**AppClassLoader**（系统类装载器）。 

- **根装载器**（bootstrap class loader：启动类加载器，是JVM内置的加载器）不是ClassLoader的子类，它使用C++编写，因此我们在Java中看不到它，根装载器**负责装载JRE的核心类库**（$JAVA_HOME中jre/lib/rt.jar里所有的class），如JRE目标下的rt.jar、charsets.jar等;

  ```java
  // 获得根类加载器所加载的核心类库,并能看到本机安装的Java
  //  环境变量指定的jdk中的核心jar路径；
  
  import java.net.URL;
  
  public class ClassLoaderTest {
      public static void main(String[] args) {
          URL[] urls=sun.misc.Launcher.getBootstrapClassPath().getURLs();
          for (URL url : urls) {
              System.out.println(url.toExternalForm());
          }
      }
  }
  
  // 输出
  file:/D:/exefile/Java/jre/lib/resources.jar
  file:/D:/exefile/Java/jre/lib/rt.jar
  file:/D:/exefile/Java/jre/lib/sunrsasign.jar
  file:/D:/exefile/Java/jre/lib/jsse.jar
  file:/D:/exefile/Java/jre/lib/jce.jar
  file:/D:/exefile/Java/jre/lib/charsets.jar
  file:/D:/exefile/Java/jre/lib/jfr.jar
  file:/D:/exefile/Java/jre/classes
  ```

- ExtClassLoader和AppClassLoader都是ClassLoader的子类：

  - **ExtClassLoader**负责装载**JRE扩展目录**（lib/ext或者由java.ext.dirs系统属性指定的目录中的JAR包的类），父类加载器为null；
  - **AppClassLoader**被称为系统（也称为应用）类加载器，它负责在JVM启动时加载来自Java命令的-classpath选项、java.class.path系统属性，或者CLASSPATH换将变量所指定的JAR包和类路径。程序可以通过**ClassLoader**的静态方法**getSystemClassLoader()**来获取系统类加载器。如果没有特别指定，则用户自定义的类加载器都以此类加载器作为父加载器。由Java语言实现，父类加载器为ExtClassLoader。

- 这三个类装载器之间存在父子层级关系，即根装载器是ExtClassLoader的父装载器，ExtClassLoader是AppClassLoader的父装载器。

- 默认情况下，使用APPClassLoader装载应用程序的类，实验如下：

  ```java
  public class ClassLoaderTest {
      public static void main(String[] args) {
          // 获得当前线程的类加载器
          ClassLoader loader=Thread.currentThread().getContextClassLoader();
          System.out.println(loader);
          System.out.println(loader.getParent());
          System.out.println(loader.getParent().getParent());
      }
  }
  
  // 输出：
  sun.misc.Launcher$AppClassLoader@18b4aac2
  sun.misc.Launcher$ExtClassLoader@1b6d3586
  null
      
  // 分析：从上述输出，AppClassLoader，父ClassLoader是
  //    ExtClassLoader，祖父ClassLoader是根类装载器，
  //    因为在Java中无法获得它的句柄，所以仅返回null。
  ```

### 类加载器工作模式

1. 检测此Class是否载入过，即在缓冲区中是否有此Class，如果有直接进入第8步，否则进入第2步；
2. 如果没有父类加载器，则要么Parent是根类加载器，要么本身就是根类加载器，则跳到第4步，如果父类加载器存在，则进入第3步；
3. 请求使用父类加载器去载入目标类，如果载入成功则跳至第8步，否则接着执行第5步；
4. 请求使用根类加载器去载入目标类，如果载入成功则跳至第8步，否则跳至第7步；
5. 当前类加载器尝试寻找Class文件，如果找到则执行第6步，如果找不到则执行第7步；
6. 从文件中载入Class，成功后跳至第8步；
7. 抛出ClassNotFountException异常；
8. 返回对应的java.lang.Class对象；

## 链接

当类被加载之后，系统为之生成一个对应的Class对象，接着将会进入连接阶段，连接阶段负责把类的二进制数据合并到JRE中。类连接又可分为如下3个阶段。

### 1. 验证

验证阶段用于检验被加载的类是否有正确的内部结构，并和其他类协调一致。Java是相对C++语言是安全的语言，例如它有C++不具有的数组越界的检查。这本身就是对自身安全的一种保护。验证阶段是Java非常重要的一个阶段，它会直接的保证应用是否会被恶意入侵的一道重要的防线，越是严谨的验证机制越安全。验证的目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，不会危害虚拟机自身安全。其主要包括四种验证，文件格式验证，元数据验证，字节码验证，符号引用验证：

- **文件格式验证**：主要验证字节流是否符合Class文件格式规范，并且能被当前的虚拟机加载处理。例如：主，次版本号是否在当前虚拟机处理的范围之内。常量池中是否有不被支持的常量类型。指向常量的中的索引值是否存在不存在的常量或不符合类型的常量；
- **元数据验证：**对字节码描述的信息进行语义的分析，分析是否符合java的语言语法的规范；
- **字节码验证：**最重要的验证环节，分析数据流和控制，确定语义是合法的，符合逻辑的。主要的针对元数据验证后对方法体的验证。保证类方法在运行时不会有危害出现；
- **符号引用验证：**主要是针对符号引用转换为直接引用的时候，是会延伸到第三解析阶段，主要去确定访问类型等涉及到引用的情况，主要是要保证引用一定会被访问到，不会出现类等无法访问的问题。

### 2. 准备

类准备阶段负责为**类的静态变量在方法区分配内存，并设置默认初始值**；在准备阶段不会分配类的实例变量的内存，实例变量会在对象实例化时随着对象一起分配在Java堆中。比如`public static int value=123;`在准备阶段时初始值为0，在初始化阶段才会变为123。

### 3. 解析

将类的二进制数据中的符号引用替换成直接引用。

- 符号引用：符号引用是以一组符号来描述所引用的目标，符号可以是任何的字面形式的字面量，只要不会出现冲突能够定位到就行。布局和内存无关。
- 直接引用：是指向目标的指针，偏移量或者能够直接定位的句柄。该引用是和内存中的布局有关的，并且一定加载进来的。

## 初始化

初始化是为类的静态变量赋予正确的初始值，准备阶段和初始化阶段看似有点矛盾，其实是不矛盾的；如果类中有语句：private static int a = 10，它的执行过程是这样的，

- 首先字节码文件被加载到内存后，先进行链接的验证这一步骤；
- 验证通过后准备阶段，给a分配内存，因为变量a是static的，所以此时a等于int类型的默认初始值0，即a=0,
- 然后到解析，到初始化这一步骤时，才把a的真正的值10赋给a,此时a=10。

## 类加载时机

### 常见情况

1. 创建类的实例，也就是new一个对象
2. 访问某个类或接口的静态变量，或者对该静态变量赋值
3. 调用类的静态方法
4. 反射（Class.forName("com.lyj.load")）
5. 初始化一个类的子类（会首先初始化子类的父类）
6. JVM启动时标明的启动类，即文件名和类名相同的那个类  

### 特殊情况

对于一个final类型的静态变量：

- 如果该变量的值在编译时就可以确定下来，那么这个变量相当于“宏变量”。Java编译器会在编译时直接把这个变量出现的地方替换成它的值，因此即使程序使用该静态变量，也不会导致该类的初始化。
- 反之，如果final类型的静态Field的值不能在编译时确定下来，则必须等到运行时才可以确定该变量的值，如果通过该类来访问它的静态变量，则会导致该类被初始化。

## 类装载机制

JVM装载类时使用“全盘负责委托机制”：

- ==“全盘负责”==：是指当一个ClassLoader装载一个类的时，除非显式地使用另一个ClassLoader来加载，否则该类所依赖及引用的类也由这个ClassLoader载入；
- ==“委托机制”==：是指先委托父装载器寻找目标类，只有在找不到的情况下才从自己的类路径中查找并装载目标类。这一点是从安全角度考虑的，试想如果有人编写了一个恶意的基础类（如java.lang.String）并装载到JVM中将会引起多么可怕的后果。但是由于有了“全盘负责委托机制”，java.lang.String永远是由根装载器来装载的，这样就避免了上述事件的发生。
- ==“缓存机制”==：缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区中搜寻该Class，只有当缓存区中不存在该Class对象时，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓冲区中。这就是为什么修改了Class后，必须重新启动JVM，程序所做的修改才会生效的原因。

### 双亲委派机制

加载顺序：==用户类加载器--->系统类加载器--->扩展类加载器--->根类加载器==

原理：如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器。如果父类加载器可以完成类加载任务，就成功返回，倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，这就是双亲委派模式。

优势：

- 避免重复加载：Java类随着它的类加载器一起具备了一种带有优先级的层次关系，通过这种层级关可以避免类的重复加载，当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次。
- 安全性：java核心api中定义类型不会被随意替换，假设通过网络传递一个名为java.lang.Integer的类，通过双亲委托模式传递到启动类加载器，而启动类加载器在核心Java API发现这个名字的类，发现该类已被加载，并不会重新加载网络传递的过来的java.lang.Integer，而直接返回已加载过的Integer.class，这样便可以防止核心API库被随意篡改。

## ClassLoader重要方法

在Java中，ClassLoader是一个抽象类，位于java.lang包中。下面对该类的一些重要接口方法进行介绍：

- `Class loadClass(String name)` ：name参数指定类装载器需要装载类的名字，必须使用全限定类名，如com.baobaotao.beans.Car。该方法有一个重载方法`loadClass(String name ,boolean  resolve)`，resolve参数告诉类装载器是否需要解析该类。在初始化类之前，应考虑进行类解析的工作，但并不是所有的类都需要解析，如果JVM只需要知道该类是否存在或找出该类的超类，那么就不需要进行解析；
- `Class defineClass(String name, byte[] b, int off, int len)`： 将类文件的字节数组转换成JVM内部的java.lang.Class对象。字节数组可以从本地文件系统、远程网络获取。name为字节数组对应的全限定类名；
- `Class findSystemClass(String name)`： 从本地文件系统载入Class文件，如果本地文件系统不存在该Class文件，将抛出ClassNotFoundException异常。该方法是JVM默认使用的装载机制；
- `Class findLoadedClass(String name)` ：调用该方法来查看ClassLoader是否已装入某个类。如果已装入，那么返回java.lang.Class对象，否则返回null。如果强行装载已存在的类，将会抛出链接错误；
- `ClassLoader getParent()`： 获取类装载器的父装载器，除根装载器外，所有的类装载器都有且仅有一个父装载器，ExtClassLoader的父装载器是根装载器，因为根装载器非Java编写，所以无法获得，将返回null；

除JVM默认的三个ClassLoader以外，可以编写自己的第三方类装载器，以实现一些特殊的需求。类文件被装载并解析后，在JVM内将拥有一个对应的java.lang.Class类描述对象，该类的实例都拥有指向这个类描述对象的引用，而类描述对象又拥有指向关联ClassLoader的引用，如图所示。

每一个类在JVM中都拥有一个对应的java.lang.Class对象，它提供了类结构信息的描述。数组、枚举、注解以及基本Java类型（如int、double等），甚至void都拥有对应的Class对象。Class没有public的构造方法。Class对象是在装载类时由JVM通过调用类装载器中的defineClass()方法自动构造的。

## Java反射

Class反射对象描述类语义结构，可以从Class对象中获取构造函数、成员变量、方法类等类元素的反射对象，并以编程的方式通过这些反射对象对目标类对象进行操作。这些反射对象类在java.reflect包中定义，下面是最主要的三个反射类：

- `Constructor`：类的构造函数反射类，通过`Class#getConstructors()`方法可以获得类的所有构造函数反射对象数组。在JDK5.0中，还可以通过`getConstructor(Class...  parameterTypes)`获取拥有特定输入参数的构造函数反射对象。Constructor的一个主要方法是`newInstance(Object[] initargs)`，通过该方法可以创建一个对象类的实例，相当于new关键字。在JDK5.0中该方法演化为更为灵活的形式：`newInstance (Object... initargs)`。
- `Method`：类方法的反射类，通过`Class#getDeclaredMethods()`方法可以获取类的所有方法反射类对象数组Method[]。在JDK5.0中可以通过`getDeclaredMethod(String name, Class...  parameterTypes)`获取特定签名的方法，name为方法名；Class...为方法入参类型列表。Method最主要的方法是`invoke(Object obj, Object[] args)`，obj表示操作的目标对象；args为方法入参。在JDK 5.0中，该方法的形式调整为`invoke(Object obj, Object...  args)`。此外Method还有很多用于获取类方法更多信息的方法： 
  - Class getReturnType()：获取方法的返回值类型；
  - Class[] getParameterTypes()：获取方法的入参类型数组；
  - Class[] getExceptionTypes()：获取方法的异常类型数组；
  - Annotation[][] getParameterAnnotations()：获取方法的注解信息，JDK 5.0中的新方法；
- `Field`：类的成员变量的反射类，通过`Class#getDeclaredFields()`方法可以获取类的成员变量反射对象数组，通过`Class#getDeclaredField(String name)`则可获取某个特定名称的成员变量反射对象。Field类最主要的方法是`set(Object obj, Object  value)`，obj表示操作的目标对象，通过value为目标对象的成员变量设置值。如果成员变量为基础类型，用户可以使用Field类中提供的带类型名的值设置方法，如`setBoolean(Object obj, boolean value)`、`setInt(Object obj, int value)`等。

此外，Java还为包提供了Package反射类，在JDK  5.0中还为注解提供了AnnotatedElement反射类。总之，Java的反射体系保证了可以通过程序化的方式访问目标类中所有的元素，对于private或protected的成员变量和方法，只要JVM的安全机制允许，也可以通过反射进行调用，请看下面的例子：

```java
// Student类
public class Student {
    private String name;
    private int age;

    public Student() {
    }
    public int say(){
        System.out.println("name"+name+" age:"+age);
        return 1;
    }
    public int change(String name,int i){
        this.name=name;
        this.age=i;
        return i;
    }
}

// 测试类
import java.lang.reflect.AnnotatedType;
import java.lang.reflect.Field;
import java.lang.reflect.Method;

public class ClassLoaderTest {
    public static void main(String[] args) throws ClassNotFoundException,
            IllegalAccessException, InstantiationException, 
    		NoSuchFieldException, NoSuchMethodException {
        // 获取当前线程的加载器
        ClassLoader loader=Thread.currentThread().getContextClassLoader();
        // 加载类
        Class<?> clazz=loader.loadClass("top.songfang.Student");
        Student student=(Student)clazz.newInstance();

        // 加载字段
        Field name=clazz.getDeclaredField("name");
        // 取消Java语言检查，以访问private变量
        name.setAccessible(true);
        name.set(student,"zhangs");

        Field age=clazz.getDeclaredField("age");
        age.setAccessible(true);
        age.set(student,20);
                
        // public Method[] getMethods()
        // 返回一个包含方法对象的数组，该方法对象反映该类对象所表示的类或接口的所有公共方法，
        // 包括类或接口声明的方法以及从超类和超接口继承的方法。
                
        // public Method[] getDeclaredMethods()
        // 返回一个包含方法对象的数组，该方法对象反映该类对象所表示的类或接口的所有声明方法，
        // 包括public、protected、default (package)访问和private方法，但不包括继承的方法
        Method[] method=clazz.getDeclaredMethods();
        for (Method method:methods) {
            System.out.println(method.getName());
        }
                
        // public Method getDeclaredMethod(String name,
        //        Class<?>... parameterTypes)
        // 返回一个方法对象，该方法对象反映该类对象所表示的类或接口的指定声明方法。
        // name参数是一个字符串，它指定所需方法的简单名称，parameterTypes参数是一个类对象数组，
        // 它按照声明的顺序标识方法的形式参数类型。如果在一个类中声明了多个具有相同参数类型的方法，
        // 并且其中一个方法的返回类型比其他方法更具体，则返回该方法;否则，任意选择其中一个方法。
        Method method=clazz.getDeclaredMethod("change",String.class,int.class);
        method.invoke(student,"zhangs",50);
        Method method1=clazz.getDeclaredMethod("say");
        method1.invoke(student);
        
        // public AnnotatedType getAnnotatedReturnType()
        // 返回一个带注释的类型对象，该对象表示使用类型来指定由该可执行文件表示的方法/构造函数的
        // 返回类型。如果这个可执行对象表示构造函数，那么带注释的type对象表示构造对象的类型。
        // 如果此可执行对象表示方法，则带注释的type对象表示使用类型来指定方法的返回类型。
        
    }
}

```

在访问private、protected成员变量和方法时必须通过`setAccessible(boolean access)`方法取消Java语言检查，否则将抛出IllegalAccessException。 如果JVM的安全管理器设置了相应的安全机制，调用该方法将抛出SecurityException。