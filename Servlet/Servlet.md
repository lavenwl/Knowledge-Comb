### 什么是servlet

Java Servlet 是运行在Web服务器或应用服务器上的程序, 他是浏览器或客户端, 与 服务器数据或程序的中间层.

### Servlet的三种创建方式

见文章[评论](https://www.runoob.com/servlet/servlet-intro.html)

```java

```

### Servlet的生命周期

1.加载：容器通过类加载器使用Servlet类对应的文件来加载Servlet

2.创建：通过**调用Servlet的构造函数来创建一个Servlet实例**

3.初始化：通过调用Servlet的init()方法来完成初始化工作，**这个方法是在Servlet已经被创建，但在向客户端提供服务之前调用。**

4.处理客户请求：Servlet创建后就可以处理请求，当有新的客户端请求时，Web容器都会**创建一个新的线程**来处理该请求。接着调用Servlet的

Service()方法来响应客户端请求（Service方***根据请求的method属性来调用doGet（）和doPost（））

5.卸载：**容器在卸载Servlet之前**需要调用destroy()方法，让Servlet释放其占用的资源。

