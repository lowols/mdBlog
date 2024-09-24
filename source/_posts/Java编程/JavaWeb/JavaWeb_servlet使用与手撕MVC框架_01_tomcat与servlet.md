---
title: JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet
tags:
  - JavaWeb
categories:
  - Java
date: 2024-06-15 17:48:00
---

## 学习说明

你将学到：

- 如何用tomcat【servlet规范的实现者】搭建一个网站

- 如何用servlet开发动态资源，并部署到tomcat网站
- 分析用原生servlet开发的问题
- 一步步搭建一个MVC框架，屏蔽servlet对业务代码的侵入

## 1 CS与BS

前置知识点：

Java / DB /JDBC
HTML/CSS/JS

**CS:客户端服务器架构模式，比如QQ，游戏LOL**
优点:充分利用客户端机器的资源，减轻服务器的负荷
   (一部分安全要求不高的计算任务存储任务放在客户端执行，不需要把所有的计算和存储都在服务器端执行，从而能够减轻服务器的压力，也能够减轻网络负荷)

缺点:需要安装;升级维护成本较高【以前网速慢，升级打补丁可能需要工程师从上海带着U盘坐飞机到新疆】

**BS:浏览器服务器架构模式**
优点:客户端不需要安装;维护成本较低
缺点:所有的计算和存储任务都是放在服务器端的，服务器的负荷较重;在服务端计算完成之后把结果再传输给客户端，因此客户端和服务器端会进行非常频繁的数据通信，从而网络负荷较重

## 2 Tomcat入门

### 2.1 认识Tomcat

servlet规范是JavaEE的一部分，是一套方便开发者开发Web程序的接口规范。

Tomcat是servlet接口规范的实现者之一。

> 强者制定规范，好比生活中强者制定游戏规则。

BS架构中的S【server】指的就是Tomcat这类Web服务器程序。

可以把Tomcat看做一个个Web项目的**容器【WebContainer】**。把一个项目【比如baidu】放到Tomcat容器里的过程叫做**部署**。

一个项目文件所在的文件夹叫做上下文。一个url往往包含着要访问**项目的上下文**【指向Web容器的一个文件夹】。

比如 http://localhost:8080/baidu/demo09.html中的baidu。

![02.server_Tomcat_deploy](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/02.server_Tomcat_deploy.png)

1. Tomcat的安装和配置
    1)解压:不要有中文不要有空格
    2)目录结构说明:

  > bin 可执行文件目录
  > conf 配置文件目录
  > lib 存放lib的目录
  > logs 日志文件目录
  > webapps 项目部署的目录
  >
  > work 工作目录
  >
  > temp 临时目录

  3)配置环境变量，让tomcat能够运行
  因为tomcat也是用java和C来写的，因此需要JRE，所以需要配置JAVA_HOME

  4)启动tomcat，双击bin目录下的starup.bat，然后访问主页，http://localhost:8080
  ![image-20240716115840436](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716115840436.png)

2. 新建Web项目，并在tomcat中部署最后再访问

   - 新建项目文件夹，在项目部署目录webapps下新建文件夹/webapps/baidu/WEB-INF，这样一个空项目就建好了
     ![image-20240716120244039](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716120244039.png)

   - 放置静态资源文件，一个静态网站项目就部署好了
     ![image-20240716120452882](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716120452882.png)

   - 输入url http://localhost:8080/baidu/demo09.html，访问刚刚新建的项目

     > baidu就是context root

     ![image-20240716120656795](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716120656795.png)

此时局域网内其他电脑也可以通过url访问新建的项目了，把localhost替换为机器的ip即可。

![02.server_Tomcat_deploy](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/02.server_Tomcat_deploy.png)

localhost:8080定位到tomcat程序，/baidu【context root】定位到我们的具体项目

## 3 IDEA一键部署Tomcat项目

### 3.1 IDEA中新建一个Web项目，自动生成WEB-INF

![image-20240716121647207](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716121647207.png)
可以看到自动生成的Web目录
![image-20240716121900786](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716121900786.png)
在web目录下新建一个Helloword.html
![image-20240716124335494](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716124335494.png)

### 3.2 在IDEA中配置tomcat

- 依次点击 Edit Configuration>templates>Tomcat Server/Local>configure
  ![image-20240716122743578](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716122743578.png)
- 选择tomcat所在目录
  ![image-20240716122856056](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716122856056.png)
- 点击+号，就会看到新增的模板
  ![image-20240716123016085](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716123016085.png)
- 部署项目  点击Deployment> Artifact
  ![image-20240716123150739](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716123150739.png)
  设置context root ，比如pro07，url就要携带这个。【设置为/更方便，url中就不用携带项目文件夹名了】
  ![image-20240716123500044](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716123500044.png)
- 设置浏览器默认打开的url，开启热部署
  ![image-20240716124251692](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716124251692.png)
  Idea也可以修改端口号，也是通过设置tocat的server.xml配置文件实现的。
  ![image-20240716124137895](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716124137895.png)

### 3.3 启动tomcat，debug模式

![image-20240716124507450](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716124507450.png)
启动成功
![image-20240716124601248](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716124601248.png)

idea将绑定的tomcat的部署目录指向了当前项目的out目录，所以此时访问不到之前的baidu项目了
![image-20240716124855704](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716124855704.png)
![image-20240716124929941](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716124929941.png)

### 3.4 常见问题

#### 3.4.1 问题1，如果新建项目时，忘记勾选web怎么办？

可以通过点击project structure> Facets>新增Web

![image-20240716125542125](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716125542125.png)
选择对应的模块
![image-20240716125718559](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716125718559.png)
确认目录没问题，点击apply
![image-20240716125814166](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716125814166.png)
成功
![image-20240716125834958](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716125834958.png)

#### 3.4.2 问题2 导入的项目 web文件夹上没有蓝点

![image-20240716125933993](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716125933993.png)

projext structure> 新增 web.xml[注意目录正确]
![image-20240716130104479](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716130104479.png)

![image-20240716130153945](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716130153945.png)

蓝点出现，成功
![image-20240716130220540](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716130220540.png)

## 4 添加动态资源

### 4.1 可接收前端参数的组件

前面我们的项目里只有静态资源，如果用户想添加一些数据到数据库里，就需要添加**动态资源组件**来处理用户请求。

如图所示，

1. 用户第一次请求add.html，后台返回的静态网页里有一个表单
2. 用户填写好表单数据后
3. 点击提交，根据表单action参数，把表单数据提交到add，
4. 后台根据“add”找到对应的动态资源组件AddSerlet
5. AddServelet负责接收表单数据，调用dao持久层方法，把数据保存进数据库。

![03.第一次使用Servlet(1)](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/03.第一次使用Servlet(1).png)

### 4.2 开发动态资源组件AddServelet

#### 4.2.1 添加依赖

普通的java类无法获取前端传来的参数，需要继承一个类 HttpServelet
![image-20240716160455502](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716160455502.png)

两种方法，1，将tomcat的lib文件夹里的servelet-api.jar复制添加到项目的lib库中。

2，添加tomcat依赖

Project structure> Modules>pro07-javaweb-begin>Dependencies>➕>上一节关联的tomcat

![image-20240716160040779](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716160040779.png)

关联成功
![image-20240716160821961](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716160821961.png)

#### 4.2.2 解析参数

表单设置的method为post，servlet的doPost方法就会被执行。我们需要重写该方法接收数据参数

![image-20240716161433298](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716161433298.png)

#### 4.2.3 配置映射

在web.xml中配置路径/add对应AddServlet

![image-20240716161338745](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716161338745.png)

#### 4.2.4 启动运行

在页面填写、提交数据，控制台成功打印
![image-20240716161909625](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716161909625.png)

#### 4.2.5 保存到数据库

添加持久层相关代码，从之前的JDBC的demo项目中复制即可

![image-20240716163000351](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716163000351.png)

添加持久层相关依赖jar包，在父项目的lib中。

![image-20240716162406462](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716162406462.png)

需要更新artifact

![image-20240716162657797](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716162657797.png)

或者删掉当前的artifact，重新建。

重新启动，可以看到数据成功添加到了数据库

![image-20240716162831634](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716162831634.png)

![image-20240716162852772](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716162852772.png)

## 5 常见问题

1. 项目与模块的关系 新建项目默认会建一个模块，我们将这个项目默认模块作为父模块，不写代码【删掉src文件夹】，通常会放置一些共用的类库，我们会在默认模块下创建子模块，在子模块中编写我们的代码。
2. 在模块中添加web，可在新建模块时勾选，也可添加模块后通过添加facets补充。
3. 创建artifact - 部署包
4. lib - artifact
   先有artifact，后来才添加的mysql.jar。此时，这个jar包并没有添加到部署包中
   那么在project structure中有一个Problems中会有提示的,我们点击fix选择add to...
   另外，我们也可以直接把lib文件夹直接新建在WEB-INF下。
   这样不好的地方是这个lib只能是当前这个moudle独享。如果有第二个moudle我们需要再次重复的新建lib。
5. 在部署的时候，修改application Context。然后再回到server选项卡，检查URL的值。
   URL的值指的是tomcat启动完成后自动打开你指定的浏览器，然后默认访问的网址。
   启动后，报错404.404意味着找不到指定的资源。
   如果我们的网址是：http://localhost:8080/pro01/ , 那么表明我们访问的是index.html.
   我们可以通过<welcome-file-list>标签进行设置欢迎页(在tomcat的web.xml中设置，或者在自己项目的web.xml中设置)
6. 405问题。当前请求的方法不支持。比如，我们表单method=post , 那么Servlet必须对应doPost。否则报405错误。
7. 空指针或者是NumberFormatException 。因为有价格和库存。如果价格取不到，结果你想对null进行Integer.parseInt()就会报错。错误的原因大部分是因为 name="price"此处写错了，结果在Servlet端还是使用request.getParameter("price")去获取。
8. <url-pattern>中以斜杠开头

项目与模块的关系

一般，我们先新建一个父项目Project，在这个项目下建多个模块module。

![image-20240716163406091](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716163406091.png)



父项目一般就是一个空java项目，什么都不要勾选

![image-20240716163531439](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716163531439.png)

在根据需要建子模块

![image-20240716163721202](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716163721202.png)

web模块添加artifact

第一个是未压缩的部署包，第二个是压缩的war包，tomcat可以自动解压webapps里的war包

![image-20240716164229522](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716164229522.png)

## 6 中文乱码问题

1. 设置编码
    tomcat8之前，设置编码：
      1)get请求方式：
        //get方式目前不需要设置编码（基于tomcat8）
        //如果是get请求发送的中文数据，转码稍微有点麻烦（tomcat8之前）
        String fname = request.getParameter("fname");
        //1.将字符串打散成字节数组
        byte[] bytes = fname.getBytes("ISO-8859-1");
        //2.将字节数组按照设定的编码重新组装成字符串
        fname = new String(bytes,"UTF-8");
      2)post请求方式：
        request.setCharacterEncoding("UTF-8");
    tomcat8开始，设置编码，只需要针对post方式
        request.setCharacterEncoding("UTF-8");
    注意：
        需要注意的是，设置编码(post)这一句代码必须在所有的获取参数动作之前

## 7 Servlet继承关系&Servlet生命周期

### 7.1 Servlet的继承关系 - 重点查看的是服务方法（service()）

1. 继承关系
    javax.servlet.Servlet接口
      javax.servlet.GenericServlet抽象类
          javax.servlet.http.HttpServlet抽象子类

2. 相关方法
    javax.servlet.Servlet接口:
    void init(config) - 初始化方法
    void service(request,response) - 服务方法
    void destory() - 销毁方法

  javax.servlet.GenericServlet抽象类：
    void service(request,response) - 仍然是抽象的

  javax.servlet.http.HttpServlet 抽象子类：
    void service(request,response) - 不是抽象的

- String method = req.getMethod(); 获取请求的方式

- 各种if判断，根据请求方式不同，决定去调用不同的do方法
  if (method.equals("GET")) {
   this.doGet(req,resp);
  } else if (method.equals("HEAD")) {
   this.doHead(req, resp);
  } else if (method.equals("POST")) {
   this.doPost(req, resp);
  } else if (method.equals("PUT")) {

 3. 在HttpServlet这个抽象类中，do方法都差不多:
 protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
    String protocol = req.getProtocol();
    String msg = lStrings.getString("http.method_get_not_supported");
    if (protocol.endsWith("1.1")) {
    resp.sendError(405, msg);
    } else {
    resp.sendError(400, msg);
    }
    }

 3. 切换项目验证

 调用父类的方法，默认会报不支持的错误信息
 ![image-20240716185329283](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716185329283.png)

 ![image-20240716185224026](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716185224026.png)

 删掉tomcat原来

 的artifact，添加pro08模块的artifact

 ![image-20240716184652259](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716184652259.png)

 ![image-20240716184810948](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716184810948.png)

![image-20240716185018486](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716185018486.png)

3.小结：

1) 继承关系： HttpServlet -> GenericServlet -> Servlet
2) Servlet中的核心方法： init() , service() , destroy()
3) 服务方法： 当有请求过来时，service方法会自动响应（其实是tomcat容器调用的）
        在HttpServlet中我们会去分析请求的方式：到底是get、post、head还是delete等等
        然后再决定调用的是哪个do开头的方法
        那么在HttpServlet中这些do方法默认都是405的实现风格-要我们子类去实现对应的方法，否则默认会报405错误
4) 因此，我们在新建Servlet时，我们才会去考虑请求方法，从而决定重写哪个do方法

### 7.2 Servlet的生命周期

1） 生命周期：从出生到死亡的过程就是生命周期。对应Servlet中的三个方法：init(),service(),destroy()
2） 默认情况下：
    第一次接收请求时，这个Servlet会进行实例化(调用构造方法)、初始化(调用init())、然后服务(调用service())
    从第二次请求开始，每一次都是服务
    当容器关闭时，其中的所有的servlet实例会被销毁，调用销毁方法
3） 通过案例我们发现：

    - Servlet实例tomcat只会创建一个，所有的请求都是这个实例去响应。
        - 默认情况下，第一次请求时，tomcat才会去实例化，初始化，然后再服务.这样的好处是什么？ 提高系统的启动速度 。 这样的缺点是什么？ 第一次请求时，耗时较长。
        - 因此得出结论： 如果需要提高系统的启动速度，当前默认情况就是这样。如果需要提高响应速度，我们应该设置Servlet的初始化时机。
        4） Servlet的初始化时机：
        - 默认是第一次接收请求时，实例化，初始化
        - 我们可以通过<load-on-startup>来设置servlet启动的先后顺序,数字越小，启动越靠前，最小值0
        5） Servlet在容器中是：单例的、线程不安全的
        - 单例：所有的请求都是同一个实例去响应

### 7.3 线程不安全

- 线程不安全：一个线程需要根据这个实例中的某个成员变量值去做逻辑判断。但是在中间某个时机，另一个线程改变了这个成员变量的值，从而导致第一个线程的执行路径发生了变化
- 我们已经知道了servlet是线程不安全的，给我们的启发是： 尽量的不要在servlet中定义成员变量。如果不得不定义成员变量，那么不要去：①不要去修改成员变量的值 ②不要去根据成员变量的值做一些逻辑判断

![01.Servlet是线程不安全的](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/01.Servlet是线程不安全的.png)

## 8 Http协议

1） Http 称之为 超文本传输协议
2） Http是无状态的
3） Http请求响应包含两个部分：请求和响应

  - 请求：
    请求包含三个部分： 1.请求行 ； 2.请求消息头 ； 3.请求主体
    1)请求行包含是三个信息： 1. 请求的方式 ； 2.请求的URL ； 3.请求的协议（一般都是HTTP1.1）
    2)请求消息头中包含了很多客户端需要告诉服务器的信息，比如：我的浏览器型号、版本、我能接收的内容的类型、我给你发的内容的类型、内容的长度等等
    3)请求体，三种情况
      get方式，没有请求体，但是有一个queryString
      post方式，有请求体，form data
      json格式，有请求体，request payload
  - 响应：
    响应也包含三本： 1. 响应行 ； 2.响应头 ； 3.响应体
    1)响应行包含三个信息：1.协议 2.响应状态码(200) 3.响应状态(ok)
    2)响应头：包含了服务器的信息；服务器发送给浏览器的信息（内容的媒体类型、编码、内容长度等）
    3)响应体：响应的实际内容（比如请求add.html页面时，响应的内容就是<html><head><body><form....）

## 9 会话

1） Http是无状态的
    - HTTP 无状态 ：服务器无法判断这两次请求是同一个客户端发过来的，还是不同的客户端发过来的
        - 无状态带来的现实问题：第一次请求是添加商品到购物车，第二次请求是结账；如果这两次请求服务器无法区分是同一个用户的，那么就会导致混乱
        - 通过会话跟踪技术来解决无状态的问题。



2） 会话跟踪技术

    - 客户端第一次发请求给服务器，服务器获取session，获取不到，则创建新的，然后响应给客户端
        - 下次客户端给服务器发请求时，会把sessionID带给服务器，那么服务器就能获取到了，那么服务器就判断这一次请求和上次某次请求是同一个客户端，从而能够区分开客户端
        - 常用的API：
          request.getSession() -> 获取当前的会话，没有则创建一个新的会话
          request.getSession(true) -> 效果和不带参数相同
          request.getSession(false) -> 获取当前会话，没有则返回null，不会创建新的

```
  session.getId() -> 获取sessionID
  session.isNew() -> 判断当前session是否是新的
  session.getMaxInactiveInterval() -> session的非激活间隔时长，默认1800秒。比如几分钟不操作，银行网站提示“会话已失效，请重新登录”
  session.setMaxInactiveInterval()
  session.invalidate() -> 强制性让会话立即失效
  ....
```

![02.会话跟踪技术](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/02.会话跟踪技术.png)

3） session保存作用域

![03.session保存作用域](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/03.session保存作用域.png)

  - session保存作用域是和具体的某一个session对应的
  - 常用的API：
    void session.setAttribute(k,v)
    Object session.getAttribute(k)
    void removeAttribute(k)

## 10 服务器内部转发以及客户端重定向

1） 服务器内部转发 : request.getRequestDispatcher("...").forward(request,response);

  - 一次请求响应的过程，对于客户端而言，内部经过了多少次转发，客户端是不知道的，地址栏没有变化

![04.服务器内部转发](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/04.服务器内部转发.png)





2） 客户端重定向： response.sendRedirect("....");

  - 两次请求响应的过程。客户端肯定知道请求URL有变化
  - 地址栏有变化

![05.客户端重定向](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/05.客户端重定向.png)

![](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716222622290.png)

## 11 Thymeleaf - 视图模板技术

![06.水果库存系统首页实现思路](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/06.水果库存系统首页实现思路.png)

1） 添加thymeleaf的jar包
![image-20240716223419610](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716223419610.png)

2） 新建一个Servlet类ViewBaseServlet
3） 在web.xml文件中添加配置

   - 配置前缀 view-prefix
   - 配置后缀 view-suffix

4） 使得我们的Servlet继承ViewBaseServlet

5） 根据逻辑视图名称 得到 物理视图名称
//此处的视图名称是 index
//那么thymeleaf会将这个 逻辑视图名称 对应到 物理视图 名称上去
//逻辑视图名称 ：   index
//物理视图名称 ：   view-prefix + 逻辑视图名称 + view-suffix
//所以真实的视图名称是：      /       index       .html
super.processTemplate("index",request,response);
6） 使用thymeleaf的标签
  th:if   ,  th:unless   , th:each   ,   th:text

启动成功

![image-20240716224213299](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716224213299.png)

// 200 : 正常响应
// 404 : 找不到资源
// 405 : 请求方式不支持
// 500 : 服务器内部错误

用注解，可以省略在web.xml配置

![image-20240716223135318](http://cdn.jsdelivr.net/gh/lowols/Pictures@main/JavaWeb_servlet使用与手撕MVC框架_01_tomcat与servlet_Img/image-20240716223135318.png)