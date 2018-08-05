# Java开源框架

## Spring框架
Spring是一个轻量级的支持控制反转（IoC）、依赖注入（DI）和面向切面（AOP）的容器框架。

### 控制反转（Inversion of Control，IoC）
* IoC容器就是具有依赖注入功能的容器，是可以创建对象的容器。IoC容器负责实例化、定位、配置应用程序中的对象及建立这些对象间的依赖。
* 通常new一个实例，控制权由程序员控制，而”控制反转”是指new实例工作不由程序员来做而是交给Spring容器来做。
* Spring通过控制反转的设计模式促进了松耦合，这种方式使一个对象依赖其它对象时会通过被动的方式传送进来，而不是通过手动创建这些类。
* 可以把IoC模式看做是工厂模式的升华，把IoC看作是一个大工厂，可以自由配置这个大工厂里要生成哪些对象，然后利用Java的反射技术生成相应的对象。
* 在Spring中BeanFactory是IoC容器的实际代表者，通过反射机制实例化Bean并建立Bean之间的依赖关系。

### 依赖注入（Dependency injection，DI）
依赖注入是指在容器创建对象后，处理对象的依赖关系。

#### 配置方式
* XML配置：可以通过ClassPathXmlApplicationContext和ClassPathXmlApplicationContext去加载Spring的配置文件，接着获取想要的实例bean并调用相应方法执行。ClassPathXmlApplicationContext默认加载CLASSPATH路径下的文件，只需指明对应文件的CLASSPATH路径即可，而FileSystemXmlApplicationContext默认加载项目工作路径（即项目的根目录）路径下的文件。
* 注解配置：事实上跟XML配置方式很相似，不过注解配置方式使用AnnotationConfigApplicationContext来加载Spring的配置文件，运行结果是一样的。

#### 注入方式
* setter注入：顾名思义，被注入的属性需要有set方法，支持简单类型和引用类型，是在Bean实例创建完成后执行的。
* 构造方法注入：通过构造方法注入依赖，构造函数的参数一般情况下就是依赖项，也支持简单值类型和引用类型。

### 自动装配（Auto-wiring）

#### XML配置方式

##### byType（根据类型）
* 通过使用`<bean>`的`autowire`属性设置为"byType"来启动根据类型的自动装配。
* Spring容器会基于反射查看Bean定义的类，然后找到与依赖类型相同的Bean注入，这个过程需要借助setter注入来完成，因此必须存在set方法，否则注入失败。
* 可能存一种注入失败的情况：由于是基于类型的注入，因此当XML文件中存在多个相同类型名称不同的实例Bean时，存在多种适合的选项，Spring容器无法知道该注入那种，所以注入会失败，此时可以把`<bean>`标签的`autowire-candidate`属性设置为false来过滤那些不需要注入的实例Bean。

##### byName（根据名称）
* 通过使用`<bean>`的`autowire`属性设置为"byName"来启动根据名称的自动装配。
* Spring会尝试将属性名与bean名称进行匹配，如果找到则注入依赖Bean。
* 需要了解的是如果Spring容器中没有找到可以注入的实例bean时，将不会向依赖属性值注入任何Bean，这时依赖Bean的属性可能为null。

##### constructor（根据构造函数）
* 通过使用`<bean>`的`autowire`属性设置为"constructor"来启动根据构造函数的自动装配。
* Spring容器同样会尝试找到那些类型与构造函数相同匹配的Bean然后注入。
* 存在单个实例则优先按类型进行参数匹配（无论名称是否匹配），当存在多个类型相同实例时，按名称优先匹配，如果没有找到对应名称则注入失败，此时可以把`<bean>`标签的`autowire-candidate`属性设置为`false`来过滤那些不需要注入的实例Bean。

#### 注解配置方式
使用注解前必须先注册注解驱动：`<context:annotation-config />`。

##### @Autowired
* Spring2.5引入了`@Autowired`注释，它可以对类成员变量、方法及构造函数进行标注，完成自动装配的工作。
* 通过`@Autowired`的使用标注到成员变量时不需要有set方法，默认按类型匹配。
* 在`@Autowired`中还传递了一个`required`的属性，false表示该Bean不存在时可以不注入，此时它为null；而如果是true，就强制要求注入，若不存在则抛出异常。
* 默认情况下`@Autowired`是按类型匹配的（byType），如果需要按名称（byName）匹配的话，可以使用`@Qualifier`注解与`@Autowired`结合。

##### @Resource
* 与`@Autowried`具备相同功效的还有`@Resource`，默认按byName模式自动注入，由J2EE提供，需导入`javax.annotation.Resource`包，可以标注在成员变量和set方法上，但无法标注构造函数。
* `@Resource`有两个重要的属性：name和type，name属性解析为Bean的名字，type属性则解析为Bean的类型。因此，使用name属性则按byName模式自动注入，使用type属性则按byType模式自动注入。
* 倘若既不指定name也不指定type属性，则默认按byName模式注入。

##### @Value
* 上述两种自动装配的依赖注入不适合简单值类型，如int、boolean、long、String以及Enum等，对于这些类型，Spring容器也提供了`@Value`注入的方式，常常与properties文件配合使用。
* `@Value`接收一个String的值，该值指定了将要被注入到内置的java类型属性值，一般不需要类型转换，有如下两种方式：
    * SpEL：`@Value("#{configProperties['jdbc.url']}")`
    * 占位符：`@Value("${jdbc.url}")`

### Spring的Bean

#### Bean的实例化方法
* 类的构造方法（最常用）
* 静态方法构造
* 实例工厂方法构造

#### Bean的重写机制
* 在不同的XML配置文件中使用相同id命名，并声明相同类型的Bean对象时，Spring容器会默认加载最后添加的XML配置文件而忽略之前的XML配置文件，即后者会覆盖前者。
* 在Web应用开发过程中，一般都会将配置进行分层管理，然后通过一个主配置文件applicationContext.xml来聚合它。在这样的情况下分层的配置文件属于applicationContext.xml的子文件，在这样的关系遇到上述的情况，一般都是子文件的优先级高，因此会加载子文件的Bean。

#### Bean的作用域
* 所谓Bean的作用域是指spring容器创建Bean后的生存周期即由创建到销毁的整个过程。
* 声明作用域的方法：
    * 使用`<bean>`标签的`scope`属性
    * 通过注解：`@Scope("singleton")`

##### Singleton
* 每一个Bean的实例只会被创建一次，而且Spring容器在整个应用程序生存期中都可以使用该实例，相当于单例模式。
* Spring默认创建的所有Bean的作用域都是Singleton。

##### Prototype
* 每次获取Bean实例时都会新创建一个实例对象，类似new操作符。

##### Request和Session
* 在Spring2.5中专门针对Web应用程序引进了Request和Session这两种作用域。
* Request作用域：每次HTTP请求到达时，会创建一个新的Request作用域的Bean实例，该实例装载了这次请求的数据（如参数等），该Bean实例仅在当前HTTP Request内有效且唯一，而其他请求HTTP请求则创建新Bean的实例互不干扰，这个Bean实例会在处理请求结束销毁。
* Session作用域：每当创建一个新的HTTP Session时就会创建一个Session作用域的Bean，并该实例Bean伴随着本次Session的存在而存在。

##### globalSession作用域
* 类似Session作用域，相当于全局变量。

#### Bean的循环依赖问题

##### 简介
* 循环依赖其实就是循环引用，也就是两个或则两个以上的Bean互相持有对方，最终形成闭环。比如A依赖于B，B依赖于C，C又依赖于A。
* 这样的情况，这些类单独使用时不会出问题，但在使用Spring管理时可能会导致BeanCurrentlyInCreationException等异常，表示Spring解决不了该循环依赖。

##### 分类
* 构造方法循环引用：A的构造方法中引用了B，同时B的构造方法中引用了A。
* setter循环引用：A的构造方法中引用了B，同时B的某个setter引用了A，以及反之；或者A的某个setter引用了B，同时B的某个setter引用了A，以及反之。

##### 解决前提
* 循环引用的Bean的scope都是单例的，并且它们没有显式指明不需要解决循环依赖，此外还要求这些Bean没有被代理过。
* 因为Spring容器不缓存prototype的Bean，所以prototype的循环引用Bean无法完成依赖注入。
* 构造方法中的循环依赖Spring仍然没法解决。

##### 解决方案
* Java基于引用传递，当获取到对象的引用时，对象的成员属性可以延后设置。
* Spring单例对象的初始化分为以下三步：createBeanInstance()实例化、populateBean()填充属性数据、initializeBean()调用配置文件中指定的init方法或AfterPropertiesSet方法。
* Spring单例对象使用三级缓存来管理，三级缓存包括：singletonObjects单例对象的cache、earlySingletonObjects提前曝光的单例对象的cache、singletonFactories单例对象工厂的cache。
* getSingleton的时候，首先尝试从singletonObjects中获取，如果获取不到就会尝试从earlySingletonObjects中获取，如果获取不到就会尝试用singletonFactory.getObject()获取，如果获取到了就移除对应的singletonFactory，并将对应的singletonObject放入earlySingletonObjects中提前曝光，这样就可以获取到一个提前曝光的对象实例从而解决循环引用了。

#### Bean的延迟加载
* 默认情况下Spring容器在启动阶段就会创建所有Bean，这个过程被称为预先初始化，这样的好处是可以尽早发现配置错误，但如果存在大量Bean需要初始化，就会引起Spring容器启动缓慢，这时候我们会希望只有在用到某个Bean的时候再创建其实例对象，以减少内存消耗。
* 一些特定的Bean没必要在Spring容器启动阶段就创建，比如Hibernate的SessionFactory等，延迟加载它们会让Spring容器启动更轻松些，从而也减少没必要的内存消耗。
* 在Spring的XML配置文件中使用`<bean>`的`lazy-init`属性可以配置该Bean是否延迟加载；如果需要配置整个XML文件的Bean都延迟加载则使用`default-lazy-init`属性，需要注意`lazy-init`属性会覆盖`default-lazy-init`属性。
* 配置了延迟加载，在以下情况下仍会触发Bean的创建：
    * 从一个已创建的Bean引用另外一个Bean；
    * 显式地查找一个Bean。

### 事务管理：面向切面编程（AOP）
AOP就是纵向的编程，如业务1和业务2都需要一个共同的操作，与其往每个业务中都添加同样的代码，不如写一遍代码，让两个业务共同使用这段代码。所以AOP把所有共有代码全部抽取出来，放置到某个地方集中管理，然后在具体运行时，再由容器动态织入这些共有代码。

#### 相关概念
* 切面（Aspect）：其实就是共有功能的实现。如日志切面、权限切面、事务切面等。在实际应用中通常是一个存放共有功能实现的普通Java类，之所以能被AOP容器识别成切面，是在配置中指定的。
* 通知（Advice）：是切面的具体实现，在实际应用中通常是切面类中的一个方法，具体属于哪类通知，同样是在配置中指定的。
* 连接点（Joinpoint）：就是程序在运行过程中能够插入切面的地点。例如，方法调用、异常抛出或字段修改等，但Spring只支持方法级的连接点。
* 切入点（Pointcut）：用于定义通知应该切入到哪些连接点上。不同的通知通常需要切入到不同的连接点上，这种精准的匹配是由切入点的正则表达式来定义的。
* 目标对象（Target）：就是那些即将切入切面的对象，也就是那些被通知的对象。这些对象中已经只剩下干干净净的核心业务逻辑代码了，所有的共有功能代码等待AOP容器的切入。
* 代理对象（Proxy）：将通知应用到目标对象之后被动态创建的对象。可以简单地理解为，代理对象的功能等于目标对象的核心业务逻辑功能加上共有功能。代理对象对于使用者而言是透明的，是程序运行过程中的产物。
* 织入（Weaving）：将切面应用到目标对象从而创建一个新的代理对象的过程。这个过程可以发生在编译期、类装载期及运行期，当然不同的发生点有着不同的前提条件。譬如发生在编译期的话，就要求有一个支持这种AOP实现的特殊编译器；发生在类装载期，就要求有一个支持AOP实现的特殊类装载器；只有发生在运行期，则可直接通过Java语言的反射机制与动态代理机制来动态实现。

#### 通知类型

##### 前置通知
* 通过@Before注解进行标注，该通知在目标函数执行前执行，并且可以直接传入切点表达式。
* 参数是JoinPoint，是Spring提供的静态变量，可以通过它获取目标对象的信息，如类名称、方法参数、方法名称等，该参数是可选的。

##### 后置通知
* 通过@AfterReturning注解进行标注，该通知在目标函数执行完成后执行，并可以获取到目标函数最终的返回值returnVal。
* 当目标函数没有返回值时，returnVal将返回null。
* 必须通过returning = “returnVal”注明参数的名称而且必须与通知函数的参数名称相同。
* 请注意，在任何通知中这些参数都是可选的，需要使用时直接填写即可，不需要使用时，可以完成不用声明出来。

##### 异常通知
* 通过@AfterThrowing注解进行标注，该通知只有在异常时才会被触发，并由throwing来声明一个接收异常信息的变量。
* 同样异常通知也可以用JoinPoint参数，需要时加上即可。

##### 最终通知
* 通过@After注解进行标注，该通知有点类似于finally代码块，只要应用了无论什么情况下都会执行。
* 参数也是JoinPoint。

##### 环绕通知
* 通过@Around注解进行标注，该通知既可以在目标方法前执行也可在目标方法之后执行，更重要的是环绕通知可以控制目标方法是否指向执行。
* 但即使如此，我们应该尽量以最简单的方式满足需求，在仅需在目标方法前执行时，应该采用前置通知而非环绕通知。
* 第一个参数必须是ProceedingJoinPoint，通过该对象的proceed()方法来执行目标函数，proceed()的返回值就是环绕通知的返回值。
* 同样的，ProceedingJoinPoint对象也可以获取目标对象的信息，如类名称、方法参数、方法名称等。

#### 织入类型
对于织入这个概念，可以简单理解为把切面应用到目标函数的过程，这个过程一般分为动态织入和静态织入。

##### 静态织入
* AspectJ采用静态织入的方式，主要采用编译期织入，即在编译期间使用AspectJ的acj编译器（类似javac）把aspect类编译成字节码后，在java目标类编译时织入，即先编译aspect类再编译目标类。
* 除了编译期织入，还存在链接期（编译后）织入，即将aspect类和java目标类同时编译成字节码文件后，再进行织入处理，这种方式比较有助于已编译好的第三方jar和Class文件进行织入操作。

##### 动态织入
* 动态织入的方式是在运行时动态将要增强的代码织入到目标类中，这样往往是通过动态代理技术完成的，如Java JDK的动态代理和CGLIB的动态代理。
* Java JDK的动态代理：底层通过反射实现，要求目标对象有接口类。
* CGLIB的动态代理：底层通过继承实现，因此可以减少没必要的接口。

#### 组成部分
* Spring使用AOP配置事务管理由以下三个部分组成：DataSource、TransactionManager和代理机制。
* 无论哪种配置方式，一般变化的只是代理机制这部分，而DataSource、TransactionManager这两部分只是会根据数据访问方式有所变化，比如使用Hibernate进行数据访问时，DataSource实际为SessionFactory，TransactionManager的实现为HibernateTransactionManager。
* 代理机制分为Java JDK的动态代理和CGLIB的动态代理。

### SpringMVC
1. 客户端请求提交到DispatcherServlet。
2. 由DispatcherServlet控制器查询HandlerMapping，找到并分发到指定的Controller中。
3. Controller调用业务逻辑处理后，返回ModelAndView。
4. DispatcherServlet查询一个或多个ViewResolver视图解析器，找到ModelAndView指定的视图。
5. 视图负责将结果显示到客户端。

## Servlet
* Servlet是用来处理客户端请求并产生动态网页内容的Java类。Servlet主要是用来处理或者是存储HTML表单提交的数据，产生动态内容，在无状态的HTTP协议下管理状态信息。
* Servlet链是把一个Servlet的输出发送给另一个Servlet的方法。第二个Servlet的输出可以发送给第三个Servlet，依次类推。链条上最后一个Servlet负责把响应发送给客户端。

### 生命周期
* 加载Servlet：当Tomcat第一次访问Servlet的时候，Tomcat会负责创建Servlet的实例。
* 初始化：当Servlet被实例化后，Tomcat会调用init()方法初始化这个对象。
* 处理服务：当浏览器访问Servlet的时候，Servlet会调用service()方法处理请求。
* 销毁：当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动调用destroy()方法，让该实例释放掉所占的资源。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁。
* 卸载：当Servlet调用完destroy()方法后，等待垃圾回收。如果有需要再次使用这个Servlet，会重新调用init()方法进行初始化操作。

### 相关API

#### doGet与doPost方法的两个参数
* HttpServletRequest：封装了与请求相关的信息
* HttpServletResponse：封装了与响应相关的信息

#### 获取页面元素的几种方式
* request.getAttribute()：一般用于获取request域对象的数据，如在跳转之前把数据使用setAttribute来放到request对象上。
* request.getParameter()：返回客户端的请求参数的值。
* request.getParameterNames()：返回所有可用属性名的枚举。
* request.getParameterValues()：返回包含参数的所有值的数组。

### 线程安全
* 由于Servlet是单例的，当多个用户访问Servlet的时候，服务器会为每个用户创建一个线程，当多个用户并发访问Servlet共享资源的时候就会出现线程安全问题。
* 原则：如果一个Servlet需要多个用户共享，则应当在访问该Servlet的时候，加同步机制synchronized或Lock。

### 常见问题

#### JSP与Servlet的区别
* Servlet是服务器端的程序，动态生成HTML页面发送到客户端，但是这样程序里会有很多out.println()，Java与HTML语言混在一起很乱，所以后来Sun公司推出了JSP。
* 其实JSP就是Servlet，每次运行的时候JSP都首先被编译成Servlet文件，然后再被编译成.class的字节码文件运行。有了JSP，在MVC项目中Servlet不再负责动态生成页面，转而去负责控制程序逻辑的作用，控制JSP与Java Bean之间的交互。

#### JSP中动态include和静态include的区别
* include指令用于把另一个页面包含到当前页面中，在转换成Servlet时包含进去的。
* 动态include用JSP标签（`<jsp:include page="included.jsp" flush="true" />`）实现，总是会检查所含文件中的变化，适合用于包含动态页面，并且可以带参数。
* 静态include用include伪码（`<%@ include file="included.html" %>`）实现，不会检查所含文件的变化，适用于包含静态页面。

#### 转发forward和重定向redirect的区别

##### 实际发生位置不同
* 转发是由服务器进行跳转的，浏览器地址栏不变。转发只有一次的HTTP请求，一次转发中request和response对象都是同一个。
* 重定向是由浏览器进行跳转的，浏览器地址栏会发生变化。重定向的原理是由response的状态码和Location头的组合而实现的，所以实现重定向会发出两个HTTP请求，request域对象是无效的，因为它不是同一个request对象。

##### 传递数据的类型不同
* 转发的request对象可以传递各种类型的数据（使用request.getAttribute()接口）。
* 重定向只能传递字符串（HTTP Request的请求参数）。

##### 跳转的时间不同
* 转发：执行到跳转语句时就会立刻跳转。
* 重定向：整个页面执行完之后才执行跳转。

##### 应用场景
* 转发：访问Servlet处理业务逻辑，然后转发到JSP显示处理结果，浏览器地址栏不变。
* 重定向：提交表单，处理成功后重定向到另一个JSP，防止表单重复提交，浏览器地址栏变了。

## JDBC
* JDBC是Java对数据库操作的封装，允许开发者用Java操作数据库，而不需要关心底层特定数据库的细节。
* JDBC驱动提供了特定厂商对JDBC API接口类的实现，驱动必须要提供java.sql包下面这些类的实现：Connection，Statement，PreparedStatement，CallableStatement，ResultSet和Driver。
* Class.forName方法用来载入跟数据库建立连接的驱动。
* 数据库连接池：像打开关闭数据库连接这种和数据库的交互可能是很费时的，尤其是当客户端数量增加的时候，会消耗大量的资源。所以可以在应用服务器启动的时候建立很多个数据库连接并维护在一个池中。连接请求由池中的连接提供。在连接使用完毕以后，把连接归还到池中，以用于满足将来更多的请求。

## Hibernate

## MyBatis

## JPA

