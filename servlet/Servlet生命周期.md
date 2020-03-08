# Servlet生命周期

Servlet生命周期可被定义为从创建到毁灭的整个过程：

- Servlet通过调用init()方法进行初始化；
- Servlet调用service()方法来处理客户端的请求；
- Servlet通过调用destory()方法终止（结束）。
- Servlet由JVM的垃圾回收器进行垃圾回收。

## init()方法

- init 方法被设计成只调用一次。它在第一次创建 Servlet 时被调用，在后续每次用户请求时不再调用。
- Servlet 创建于用户第一次调用对应于该 Servlet 的 URL 时，但是您也可以指定 Servlet 在服务器第一次启动时被加载。

- 当用户调用一个 Servlet 时，就会创建一个 Servlet 实例，每一个用户请求都会产生一个新的线程，适当的时候移交给 doGet 或 doPost 方法。init() 方法简单地创建或加载一些数据，这些数据将被用于 Servlet 的整个生命周期。

init定义：

```java
public void init() throws ServletException {
  // 初始化代码...}
```

## service()方法

- service() 方法是执行实际任务的主要方法。Servlet 容器（即 Web 服务器）调用 service() 方法来处理来自客户端（浏览器）的请求，并把格式化的响应写回给客户端。

- 每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。service() 方法检查 HTTP  请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 doGet、doPost、doPut，doDelete 等方法。
- service() 方法由容器调用，service 方法在适当的时候调用 doGet、doPost、doPut、doDelete 等方法。所以，您不用对  service() 方法做任何动作，您只需要根据来自客户端的请求类型来重写 doGet() 或 doPost() 即可。doGet() 和 doPost() 方法是每次服务请求中最常用的方法。下面是这两种方法的特征。

service定义：

```java
public void service(ServletRequest request, 
                    ServletResponse response) 
      throws ServletException, IOException{}
```

## doGet()方法



