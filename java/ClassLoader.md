# ClassLoader

## 层次

- BootstrapClassLoader：启动类加载器
  - 用来加载%JAVA_HOME%/jre/lib路径下的类；
  - -Xbootclasspath参数可以用来指定加载路径；
  - 由C++实现，无法通过ExtClassLoader.getParent()方法获得；
  - sun.misc.Launcher.getBootstrapClassPath()方法可以获得启动类加载器目录；
- ExtClassLoader：扩展类加载器
  - 用来加载%JAVA_HOME%/jre/lib/ext路径以及java.ext.dirs系统变量指定类路径下的类；
- AppClassLoader：应用程序类加载器
  - 加载应用程序ClassPath下的类（包含jar包），属于java应用程序默认的类加载器；
- 用户自定义类加载器
  - 用户根据自定义需求，自己定制加载的逻辑，继承AppClassLoader，仅覆盖findClass()将继续遵守双亲委派模型；
- *ThreadContextClassLoader：
  - 线程上下文加载器，可以是以上任意一种类加载器；

**加载步骤**：

1. 虚拟机启动会初始化BootstrapClassLoader；
2. 然后再Launcher类中去加载ExtClassLoader、AppClassLoader，并将AppClassLoader的parent设置为ExtClassLoader；
3. 并设置线程上下文加载器。

```java
import java.net.URL;

public class ClassloaderTest {
    public static void main(String[] args) {
        // 获取启动类加载器的加载路径
        for(URL item:sun.misc.Launcher.
            getBootstrapClassPath().getURLs()){
            System.out.println(item);
        }
        // 获取本类的加载器
        ClassLoader loader=ClassloaderTest.class.getClassLoader();
        System.out.println(loader);
        System.out.println(loader.getParent());
        System.out.println(loader.getParent().getParent());
    }
}
// 输出：
file:/D:/exefile/Java/jre/lib/resources.jar
file:/D:/exefile/Java/jre/lib/rt.jar
file:/D:/exefile/Java/jre/lib/sunrsasign.jar
file:/D:/exefile/Java/jre/lib/jsse.jar
file:/D:/exefile/Java/jre/lib/jce.jar
file:/D:/exefile/Java/jre/lib/charsets.jar
file:/D:/exefile/Java/jre/lib/jfr.jar
file:/D:/exefile/Java/jre/classes
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@1b6d3586
null
```

**双亲委派模型**：

当一个类加载器去加载类时会先尝试让其父类加载器去加载，如果父类加载不了再尝试自身加载；

好处：保证安全与稳定性，可以保证基础类只加载一次，不会让jvm加载同名的类。

## 实现一个类加载器

实现一个自己的类加载器只需要继承ClassLoader然后覆盖loadClass(String name)方法即可以完成一个带有双亲委派模型的类加载器。然后parent需要自己设置，因此可以放在构造方法中设置。

不继承AppClassLoader的原因：AppClassLoader和ExtClassLoader加载器都是Launcher的静态类，都是包访问路径权限的(private修饰)。

ClassLoader#loadClass源码：

```java
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
    synchronized (getClassLoadingLock(name)) {
        // 检查是否已经加载过该类，加载过的类会有缓存
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
                if (parent != null) {
                    // 父类不为空先由父类加载
                    c = parent.loadClass(name, false);
                } else {
                    // 父类为null就由BootstrapClassLoader加载
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }

            if (c == null) {
                // 父类不能加载该类就让当前类加载
                long t1 = System.nanoTime();
                c = findClass(name);
            }
        return c;
    }
}
```

代码热替换，在不重启服务器的情况下修改类的代码并使之生效：

```java
// 自定义ClassLoader
import java.io.ByteArrayOutputStream;
import java.io.FileInputStream;
import java.io.IOException;
import java.util.Set;

public class MyClassLoader extends ClassLoader {
    // 用于读取.Class文件的路径
    private String swapPath;
    // 用于标记这些name的类是先由自身加载的
    private Set<String> useMyClassLoaderLoad;

    private MyClassLoader(String swapPath, Set<String> useMyClassLoaderLoad) {
        this.swapPath = swapPath;
        this.useMyClassLoaderLoad = useMyClassLoaderLoad;
    }

    @Override
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        Class<?> c=findLoadedClass(name);
        if (c==null && useMyClassLoaderLoad.contains(name)){
            // 特殊的类让我自己加载
            c=findClass(name);
            if (c!=null){
                return c;
            }
        }
        return super.loadClass(name);
    }

    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        // 根据文件系统路径加载class文件，并返回byte数组
        byte[] classBytes=getClassByte(name);
        // 调用ClassLoader提供的方法，将二进制数组转换为Class类的实例
        return defineClass(name,classBytes,0,classBytes.length);
    }

    private byte[] getClassByte(String name) {
        String className=name.substring(
            name.lastIndexOf('.')+1,name.length())+".class";
        System.out.println(className);
        try {
            FileInputStream fileInputStream=new FileInputStream(swapPath+className);
            byte[] buffer=new byte[1024];
            int length=0;
            ByteArrayOutputStream byteArrayOutputStream=new ByteArrayOutputStream();
            while ((length=fileInputStream.read(buffer))>0){
                byteArrayOutputStream.write(buffer,0,length);
            }
            return byteArrayOutputStream.toByteArray();
        }catch (IOException e){
            e.printStackTrace();
        }
        return new byte[]{};
    }
}
// 测试类
public class Test {
    public void printVersion(){
        System.out.println("当前版本为36");
    }
}
// 定时任务
import java.lang.reflect.InvocationTargetException;
import java.util.HashSet;
import java.util.Set;
import java.util.Timer;
import java.util.TimerTask;

public class TimerTest {
    private static Set<String> set=new HashSet<>();
    public static void main(String[] args) {
        new Timer().schedule(new TimerTask() {
            @Override
            public void run() {
                String path=MyClassLoader.class.getResource("").getPath();
                // 每次都实例化一个ClassLoader，这里传入swap路径，和需要加载的类名
                String className="top.songfang.connection.classloaderTest.Test";
                MyClassLoader myClassLoader=new MyClassLoader(path,set);
                try {
                    Object o=myClassLoader.loadClass(className).newInstance();
                    // 不能使用((Test)o).printVersion(),因为Test.class会隐性加载当前类的
                    // ClassLoader加载，而Main默认加载为AppClassLoader，非自定义的；
                    // 同一个类被不同的加载器加载，会被认为是不同的类抛出ClassCastException
                    o.getClass().getMethod("printVersion").invoke(o);
                }catch (InstantiationException|IllegalAccessException
                        |ClassNotFoundException|NoSuchMethodException
                        |InvocationTargetException ignored){
                }
            }
        },0,2000);
    }
}
```