由于Servlet/JSP默认是以多线程模式执行的，所以，在编写代码时需要注意多线程的同步问题。

存在多线程问题的程序例子
首先有一个JSP页面，其中有一个简单的表单
```html
<form action="MultiThreadServlet">
	<input type="text" name="username">
	<input type="submit" value="submit">
</form>
```
提交表单后，转向一个Servlet进行处理:
获取请求中的参数，并且调用setAttribute方法将其值存储，转向下一个jsp页面：
```java
import java.io.IOException;
import java.io.PrintWriter;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
public class MultiThreadServlet extends HttpServlet
{
    //使用成员变量
    private String username;
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
    {
        //从请求中得到参数，即用户名
        username = request.getParameter("username");
        
        //得到当前线程的名字
        System.out.println("Thread Name: " + Thread.currentThread().getName());
        
        
        //模拟一些后端的业务处理
        try
        {
            
            Thread.sleep(10000);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        
        request.setAttribute("username", username);
        //请求转发
        request.getRequestDispatcher("hello.jsp").forward(request, response);
    }
}
```

中间让线程停留了10秒钟，来模拟一些操作。

在下一个JSP页面中将该值显示出来：
```java
<body>
	username: <%= request.getAttribute("username")%>
</body>
```


** 这样做有什么问题呢？ **
打开浏览器，输入访问地址后，输入一个用户名zhangsan，再打开一个窗口，输入用户名lisi。
两个浏览器窗口都提交以后，过了一定时间，可以看到两边返回值都是lisi。

** 问题原因 **
Servlet的多线程同步问题：
Servlet本身是单实例的，这样当有多个用户同时访问某个Servlet时，会访问该唯一的Servlet实例中的成员变量，如果对成员变量进行写入操作，那就会导致Servlet的多线程问题，即数据不一致。

** 解决同步问题的方案 **
有三种方法可以解决，建议使用第一种方法
* 不使用成员变量，而使用局部变量，因为局部变量在每个线程中都有各自的实例
```java
public class MultiThreadServlet extends HttpServlet
{
    //使用成员变量
    //private String username;
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException
    {
        //从请求中得到参数，即用户名
        String username = request.getParameter("username");
        
        //得到当前线程的名字
        System.out.println("Thread Name: " + Thread.currentThread().getName());
        
        
        //模拟一些后端的业务处理
        try
        {
            
            Thread.sleep(10000);
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
        
        request.setAttribute("username", username);
        //请求转发
        request.getRequestDispatcher("hello.jsp").forward(request, response);
    }
}
```

* 使用同步代码块
synchronized{ }

* Servlet实现javax.serlvet.SingleThreadModel
（Servlet2.4中已经废弃了该接口），此时Servlet容器将保证Servlet实例以单线程方式运行，也就是说，同一时刻，只会有一个线程执行Servlet的service()方法。

** 总结 **
在Servlet中，如果要对某个变量做写入操作，一定不要使用成员变量，而要使用局部变量。
