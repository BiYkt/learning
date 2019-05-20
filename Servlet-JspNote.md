## HTTP交互流程
### http协议交互流程
- 建立链接
- 发送请求(`GET` 和`POST`)
- 处理请求
- 响应请求
- 关闭连接

### 常见状态码
- `200` OK //客户端请求发送成功
- `400` Bad Request //客户端请求有语法错误，不能被服务器所理解
- `401` Unauthorized //请求未经授权，这个状态代码必须和WWW-Authenticate报头域一起使用
- `403` Forbidden //服务器收到请求，但是拒绝提供服务
- `404` Not Found //请求资源不存在，eg：输入了错误的URL
- `500` Internal Server Error //服务器发生不可预期的错误
- `503` Server Unavailable //服务器当前不能处理客户端请求，一段时间后可能恢复正常

### Servlet常见错误
- `404`错误：资源未找到
    - 原因一：在请求地址中的servlet的别名书写错误
    - 原因二：虚拟项目名称拼写错误
- `500`错误：内部服务器错误
    - 错误一：
        `java.lang.ClassNotFoundException`
    解决：在`web.xml`中校验`servlet`类的全限定路径是否拼写错误
    - 错误二：因为`servlet`方法体的代码执行错误导致
    解决：根据错误提示对`service`方法体中的代码进行错误更改
- `405`错误： 请求方式不支持
    - 原因：请求方式和`servlet`中的方法不匹配所造成。
    解决：尽量使用`service`方法进行请求处理，并且不要在`service`方法中调用父类的`service`

#### 注意：放在`Tomcat`下的`webapps`目录中的代码一旦更新需要重新复制粘贴

## Servlet的使用步骤
1. 创建普通的Java类并继承`HttpServlet`
2. 覆写`service`方法
3. 在`service`方法中写逻辑代码
4. 在`webRoot`下的`WEB-INF`文件夹下的`web.xml`文件中配置`servlet`

## Servlet生命周期
从第一次被调用时到服务器关闭的时候，若`在web.xml`中配置了`load-on-startup`生命周期则从服务器启动到关闭时

## Servlet基本方法
- `init()`方法时对Servlet进行初始化的一个方法，在Servlet第一次加载进内存时执行
- `destroy()`方法在Servlet被销毁即服务器关闭时执行
- `service()`方法可以处理`get`和` post`方式的请求，如果servlet中又service方法，会**优先调用**`service`方法对请求进行处理
- `doGet()`方法处理get方法的请求
- `doPost()`方法处理post方法的请求

##### 注意：如果在覆写的service方法中调用了父类的service,即`super.service(arg0,arg1)`则service方法处理完后会再次根据请求方式调用doGet或者doPost方法，所以一般不在覆写的service方法中调用父类的service

## request对象
- 介绍： `request对象`由Tomcat服务器创建，并作为实参传递给处理请求的`servlet`的`service`方法，`request`对象包含了请求数据。
- 作用：`request对象`中封存了当前请求的所有请求信息。
- 使用：
    1. 获取请求头数据
    ```Java
    req.getMethod(); //获取请求方式
    req.getRequestURL(); //获取请求URL信息
    req.getRequestURI(); //获取请求URI信息
    req.getScheme(); //获取协议
    ```
    2. 获取请求行数据
    ```Java
    req.getHeader("键名"); //返回指定的请求头信息
    req.getHeaderNames(); //返回请求头的键名枚举集合
    ```
    3. 获取用户数据
    ```Java
    req.getParameter("键名"); //返回指定的用户数据
    req.getParameterValues("键名"); //返回同键不同值的请求数据(多选)，返回的是数组
    req.getParameterNames(); //返回所有用户请求数据的枚举集合
    ```
    **注意： 如果要获取的请求数据不存在，不会报错，返回`null`。**

#### 注意： `request对象`由Tomcat服务器创建，并作为实参传递给处理请求的`servlet`的`service`方法 

## 请求中文乱码解决方法
1. 使用`String`进行数据重新编码
```Java
uname = new String(uname.getBytes("iso8859-1", "utf-8");
```

2. 使用公共配置
    - `get方式`： 
        1. 
        ```Java
        req.setCharacterEncoding("utf-8");
        ```
        2. 在`Tomcat`的目录下的`conf`目录中修改`server.xml`文件： 在`Connector`标签中添加属性`useBodyEncodingForURI = "true"`
    - `post方式`：
        ```Java
        req.setCharacterEncoding("utf-8");
        ```

## Servlet流程总结
1. 浏览器发起请求到服务器(请求)
2. 服务器接收浏览器的请求，进行解析，创建`request对象`存储请求数据
3. 服务器调用相对应的`servlet`进行请求处理，并将`request对象`作为实参传递给`servlet`的方法
4. `servlet`的方法执行进行请求处理
    - 设置请求编码格式
    - 设置响应编码格式
    - 获取请求信息
    - 处理请求信息
        - 创建业务层对象
        - 调用业务层对象的方法
    - 响应处理结果

## 请求转发
### 作用：
- 实现多个`servlet`联动操作处理请求，避免代码冗余，让`servlet`的职责更加明确

### 使用：
```Java
req.getRequestDispatcher("要转发的地址").forward(req,resp);
//地址：相对路径，直接书写`servlet`别名即可
```

### 特点：
- 一次请求，浏览器的地址栏信息不改变，因此每次刷新都是在原有页面上刷新，所以可能会导致表单数据的多次提交（刷新一次就提交一次）。

## request对象作用域
### 使用：
```Java
request.setAttribute(object name, object value);
request.getAttribute(object obj);
```

### 作用：
- 解决了一次请求内不同`servlet`的数据共享问题

### 作用域：
- 基于请求转发，一次请求中的所有`servlet`共享

## 重定向
### 作用：
- 解决了表单重复提交的问题，以及当前`servlet`无法处理的请求问题。

### 使用：
```Java
resp.sendRedirect(String uri);
//示例：
resp.sendRedirect("/login/main");
```

### 特点：
- **两次请求**，两个`request对象`
- 浏览器地址栏信息改变

### 使用时机：
- 如果请求中有表单数据，而数据又比较重要，不能重复提交，建议使用重定向
- 如果请求被`servlet`接收后，无法进行处理，建议使用重定向定位到可以处理的资源

## Cookie学习
### 作用：
- 解决了发送的不同请求的数据共享问题

### 使用：
- `Cookie`的创建和存储
```Java
//创建Cookie对象
Cookie c = new Cookei(String name, String value);
//设置Cookie(可选)
    //设置有效期
    c.setMaxAge(int  seconds);
    //设置有效路径
    c.setPath(String uri);
//响应Cookie信息给客户端
resp.addCookie(c);
```

### 注意：
- 一个`Cookie对象`存储一条数据，多条数据可以创建多几个`Cookie对象`进行存储

### 特点：
- 浏览器端的数据存储技术
- 存储的数据声明在服务器端
- 临时存储：存储在浏览器的运行内存中，浏览器关闭即失效
- 定时存储：设置了`Cookie`有效期，存储在客户端的硬盘中，在有效期内符合路径要求的（跟相应`cookie`数据的项目在同一个路径）请求都会附带该信息
- 默认`cookie`信息存储好以后，每次请求都会附带，除非设置有效路径

## Session技术
### 原理：
- 用户第一次访问服务器，服务器会创建一个`session对象`给用户，并将该`session对象`的`JSESSIONID`使用`cookie技术`存储到浏览器中，保证用户的其他请求能够获取到同一个`session对象`，也保证了不同请求能够获取到共享的数据。作用时间为一次会话（浏览器开启到关闭）

### 特点：
- 存储在服务器端
- 服务器进行创建
- 依赖Cookie技术
- 一次会话
- 默认存储时间是30分钟

### 作用：
- 解决了一个用户不同请求处理的数据共享问题

### 作用域：
- 一次会话
- 在JSESSIONID和SESSION对象不失效的情况下为整个项目内。

### 使用：
- 使用时机:一般用户在登陆web项目时会将用户的个人信息存储到Sesion中，供该用户的其他请求使用。
- 创建session对象/获取session对象
```Java
HttpSession hs=req.getSession();
//如果请求中拥有session的标识符也就是JSESSIONID，则返回其对应的session队形
//如果请求中没有session的标识符也就是JSESSIONID，则创建新的session对象，并将其JSESSIONID作为从cookie数据存储到浏览器内存中
//如果session对象是失效了，也会重新创建一个session对象，并将其JSESSIONID存储在浏览器内存中。
```

- 设置session存储时间
    - 注意：在指定的时间内session对象没有被使用则销毁，如果使用了则重新计时。
```Java
hs.setMaxInactiveInterval(int seconds);
```

- 设置session强制失效
```Java
hs.invalidate();
```

- 存储和获取数据
	- 存储：`hs.setAttribute(String name,Object value);`
	- 获取：`hs.getAttribute(String name)` ,返回的数据类型为Object
	- 注意：存储的动作和取出的动作发生在不同的请求中，但是存储要先于取出执行。

### session失效处理：
- 将用户请求中的`JSESSIONID`和后台获取到的`SESSION对象的JSESSIONID`进行比对，如果一致，则`session`没有失效，如果不一致则证明`session`失效了。重定向到登录页面，让用户重新登录。

	
### 总结：
- `session`解决了一个用户的不同请求的数据共享问题，只要在`JSESSIONID`不失效和`session对象`不失效的情况下，用户的任意请求在处理时都能获取到同一个session对象。

### 注意：
- `JSESSIONID`存储在了`Cookie`的临时存储空间中，浏览器关闭即失效。

## SeveletContext对象学习
### 作用：
- 解决不同的用户使用相同的数据的问题

### 特点:
- 服务器创建
- 用户共享

### 作用域：
- 整个项目内

### 生命周期：
- 服务器启动到服务器关闭

### 使用：
- 注意:
    - 不同的用户可以给ServletContext对象进行数据的存取。
    - 获取的数据不存在返回null。
    
- 获取ServletContext对象
```Java
//第一种方式：
ServletContext sc=this.getServletContext();
//第二种方式：
ServletContext sc2=this.getServletConfig().getServletContext();
//第三种方式：
ServletContext sc3=req.getSession().getServletContext();
```
- 使用ServletContext对象完成数据共享
 ```Java
//数据存储
sc.setAttribute(String name, Object value);
//数据获取
sc.getAttribute("str") 返回的是Object类型
 ```

 - 获取项目中web.xml文件中的全局配置数据
```Java
sc.getInitParameter(String name);
//根据键的名字返回web.xml中配置的全局数据的值，返回String类型。如果数据不存在返回null。
sc.getInitParameterNames(); 
//返回键名的枚举
```

- 配置方式：**注意**一组`<context-param>`标签只能存储一组键值对数据，多组可以声明多个`<context-param>`进行存储。作用是将静态数据和代码进行解耦。
```Java
<context-param>
	<param-name>name</param-name>
	<param-value>zhangsan</param-value>
</context-param>
```

- 获取项目webroot下的资源的绝对路径。
```Java
String path=sc.getRealPath(String path);
//获取的路径为项目根目录，path参数为项目根目录中的路径
```

- 获取webroot下的资源的流对象
```Java
InputStream is = sc.getResourceAsStream(String path);
//path参数为项目根目录中的路径
//注意：此种方式只能获取项目根目录下的资源流对象，class文件的流对象需要使用类加载器获取。
```