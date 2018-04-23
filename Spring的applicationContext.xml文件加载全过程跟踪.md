

### Spring的applicationContext.xml文件加载全过程跟踪



> Talk is cheap, show me the code.

#### 一. spring mvc 下的跟踪

web.xml中配置了spring mvc的DispatcherServlet(下面是示例文件部分内容)：

![1523864585(1)](C:\Users\huang_bin\Pictures\Saved Pictures\1523864585(1).png)

可以看到，配置了一个<code>contextConfigLocation</code>属性，其值如图。

因此，以DispatcherServlet入手进行跟踪。

由于是在Tomcat环境下，tomcat启动之后，会根据web.xml配置环境，包括servlet。根据**servlet规范**，servlet是由init()方法进行初始化的。所以，寻找init()方法。

***DispatcherServlet***中没有找到，寻找父类***FrameworkServlet***，也没有找到，找***FrameworkServlet***的父类***HttpServletBean***，找到如下：

![捕获](C:\Users\huang_bin\Pictures\Saved Pictures\捕获.PNG)

寻找***initServletBean();***在***FrameworkServlet***中找到，如图：

![3](C:\Users\huang_bin\Pictures\Saved Pictures\3.PNG)

进入***initWebApplicationContext()***方法，如下图：

![4](C:\Users\huang_bin\Pictures\Saved Pictures\4.PNG)

截止到现在,并没有进行任何webApplicationContext变量的操作，因此其值为null.

进入***findWebApplicationContext***，会得到null。

继续跟踪进入***createWebApplicationContext***:

![5](C:\Users\huang_bin\Pictures\Saved Pictures\5.PNG)

这里通过***wac.setConfigLocation(getContextConfigLocation())***设置了web.xml路径中configLocation的值。

然后下面就到了 ***configureAndRefreshWebApplicationContext***方法，如图：

![6](C:\Users\huang_bin\Pictures\Saved Pictures\6.PNG)

注意到这里执行了wac的refresh方法。进入wac的类，发现是个接口，那么就需要找出wac的实际实现类。

最后，可以发现是由 ***private Class<?> contextClass = DEFAULT_CONTEXT_CLASS;***决定的。因此发现如下：

![7](C:\Users\huang_bin\Pictures\Saved Pictures\7.PNG)

这样，继续深入 ***XmlWebApplicationContext***类，通过查看层层继承关系，发现在 ***AbstractApplicationContext***类中，其 ***refresh***方法如下：

![8](C:\Users\huang_bin\Pictures\Saved Pictures\8.PNG)

显然，这里获取了一个factory，让我们来详细看一下这个factory是什么类型的。

打开 ***obtainFreshBeanFactory***方法：

![9](C:\Users\huang_bin\Pictures\Saved Pictures\9.PNG)

最后发现在其子类的 ***getBeanFactory***方法中。前面我们已经知道wac的实际类型是  ***XmlWebApplicationContext***，因此我们到这个类里面来找。

在实际类型 ***XmlWebApplicationContext*** 的父类 ***AbstractRefreshableApplicationContext***中，可以看到实现了:

![10](C:\Users\huang_bin\Pictures\Saved Pictures\10.PNG)

但是只是返回了beanFactory而已，那么beanFactory到底是什么时候被填充进去的呢。在上上图中发现调用了 ***refreshBeanFactory***方法，而在***AbstractRefreshableApplicationContext***中也存在这个方法，可以确定上上图调用的是 ***AbstractRefreshableApplicationContext***的 ***refreshBeanFactory***方法。该方法如图：

![11](C:\Users\huang_bin\Pictures\Saved Pictures\11.PNG)

调用了 ***createBeanFactory***方法，继续跟踪，![12](C:\Users\huang_bin\Pictures\Saved Pictures\12.PNG)

可以发现，这个factory是 ***DefaultListableBeanFactory***类型的。

到这里，可以确定beanfactory的类型了。回到 ***refreshBeanFactory***方法，可以看到一个***loadBeanDefinitions***方法，因此可以确定， ***applicationContext.xml***文件是在这一语句中被解析和存进 ***beanfactory***的。
