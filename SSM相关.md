### 什么是Spring beans
那些形成Spring应用的主干的java对象。它们被Spring IOC容器初始化，装配，和管理。这些beans通过容器中配置的元数据创建。比如，以XML文件中的形式定义。

### Spring框架设计的核心：IoC容器和AOP模块。
- 通过IoC容器管理POJO对象以及他们之间的耦合关系；通过AOP以动态非侵入的方式增强服务。
- IoC让相互协作的组件保持松散的耦合，而AOP编程允许你把遍布于应用各层的功能分离出来形成可重用的功能组件。

### 什么是Spring IOC 容器
Ioc即控制反转， 意思是将传统的代码对对象的控制权交给容器，通过容器实现对对象的装配和管理。

### Ioc有什么作用
Spring Ioc负责创建对象，管理对象，装配对象，配置对象，并且管理这些对象的生命周期
- 管理对象的创建、维护依赖关系
- 解耦，用容器去维护具体的对象
- 托管的类的产生过程，比如我们需要在类的产生过程中做一些处理，最直接的例子就是代理，如果有容器程序可以把这部分处理交给容器，应用程序则无需去关心类是如何完成代理的

### Ioc是怎么实现的
反射 + 工厂模式
```java
interface Fruit {
    public abstract void eat();
}
class Apple implements Fruit {
    public void eat() {
        System.out.println("Apple");
    }
}
class Orange implements Fruit {
    public void eat() {
        System.out.println("Orange");
    }
}

class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f = (Fruit)Class.forName(ClassName).newInstance();
    }
}
public static void main(String[] a) {
    Fruit f = Factory.getInstance("io.github.dunwu.spring.Apple");
    if (f != null) {
        f.eat();
    }
}
```

### 说说Spring 里用到了哪些设计模式
- 单例模式：Spring 中的 Bean 默认情况下都是单例的。无需多说。
- 工厂模式：工厂模式主要是通过 BeanFactory 和 ApplicationContext 来生产 Bean 对象。
- 代理模式：最常见的 AOP 的实现方式就是通过代理来实现，Spring主要是使用 JDK 动态代理和 CGLIB 代理。
- 模板方法模式：主要是一些对数据库操作的类用到，比如 JdbcTemplate、JpaTemplate，因为查询数据库的建立连接、执行查询、关闭连接几个过程，非常适用于模板方法。

### BeanFactory和ApplicationContext的关系
- BeanFactory 简单粗暴，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供注册（put），获取（get）这两个功能。我们可以称之为 “低级容器”
- ApplicationContext 继承了多个接口，比 BeanFactory 多了更多的功能，例如资源的获取，支持多种消息（例如 JSP tag 的支持），对 BeanFactory 多了工具级别的支持等待。

### 依赖注入
组件之间的依赖关系由容器在应用系统运行期来决定，也就是由容器动态的将某种依赖关系的目标对象实例注入到应用系统的各个关联组件当中。

### 依赖注入的基本原则
应用组件不应该负责查找资源或者其他依赖的协作对象。配置对象的工作应该由IoC容器负责，“查找资源”的逻辑应该从应用组件的代码中抽取出来，交给IoC容器负责。

### 有哪些不同类型的依赖注入实现方式
- setter方法注入：容器通过调用无参构造器或无参static工厂方法实例化bean之后，调用该bean的setter方法，即实现了基于setter的依赖注入。
- 构造器注入：通过容器触发一个类的构造器来实现的，该类有一系列参数，每个参数代表一个对其他类的依赖。

### 如何解决循环依赖问题
首先，Spring 解决循环依赖有两个前提条件：
1. 不全是构造器方式的循环依赖
2. 必须是单例
   
本质上解决循环依赖的问题就是使用三级缓存，通过三级缓存提前拿到未初始化的对象。
- 第一级缓存：用来保存实例化、初始化都完成的对象
- 第二级缓存：用来保存实例化完成，但是未初始化完成的对象
- 第三级缓存：用来保存一个对象工厂，提供一个匿名内部类，用于创建二级缓存中的对象
  
假设一个简单的循环依赖场景，A、B互相依赖。

![avatar](./image/spring-1.png)

A对象的创建过程：
1. 创建对象A，实例化的时候把A对象工厂放入三级缓存
   ![avatar](./image/spring-2.png)
2. A注入属性时，发现依赖B，转而去实例化B
3. 同样创建对象B，注入属性时发现依赖A，依次从一级到三级缓存查询A，从三级缓存通过对象工厂拿到A，把A放入二级缓存，同时删除三级缓存中的A，此时，B已经实例化并且初始化完成，把B放入一级缓存。
   ![avatar](./image/spring-3.png)
4. 接着继续创建A，顺利从一级缓存拿到实例化且初始化完成的B对象，A对象创建也完成，删除二级缓存中的A，同时把A放入一级缓存
5. 最后，一级缓存中保存着实例化、初始化都完成的A、B对象
   ![avatar](./image/spring-4.png) 

因此，由于把实例化和初始化的流程分开了，所以如果都是用构造器的话，就没法分离这个操作，所以都是构造器的话就无法解决循环依赖的问题了。
#### 为什么要三级缓存？二级不行吗？
- 不可以，主要是为了生成代理对象。因为三级缓存中放的是生成具体对象的匿名内部类，他可以生成代理对象，也可以是普通的实例对象。使用三级缓存主要是为了保证不管什么时候使用的都是一个对象。
- 假设只有二级缓存的情况，往二级缓存中放的是一个普通的Bean对象，BeanPostProcessor去生成代理对象之后，覆盖掉二级缓存中的普通Bean对象，那么多线程环境下可能取到的对象就不一致了。
  ![avatar](./image/spring-5.png) 

### SpringBean 生命周期简单概括为4个阶段：
1. 实例化，创建一个Bean对象
2. 填充属性，为属性赋值
3. 初始化
   - 如果实现了xxxAware接口，通过不同类型的Aware接口拿到Spring容器的资源
   - 如果实现了BeanPostProcessor接口，则会回调该接口的postProcessBeforeInitialzation和postProcessAfterInitialization方法
   - 如果配置了init-method方法，则会执行init-method配置的方法
4. 销毁 
   - 容器关闭后，如果Bean实现了DisposableBean接口，则会回调该接口的destroy方法   
   - 如果配置了destroy-method方法，则会执行destroy-method配置的方法

### 类的作用域
- singleton : bean在每个Spring ioc 容器中只有一个实例。
- prototype：一个bean的定义可以有多个实例。
- request：每次http请求都会创建一个bean，该作用域仅在基于web的Spring ApplicationContext情形下有效。
- session：在一个HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。
- global-session：在一个全局的HTTP Session中，一个bean定义对应一个实例。该作用域仅在基于web的Spring ApplicationContext情形下有效。

### 有状态和无状态
有状态就是有数据存储功能。无状态就是不会保存数据。

### 单例bean是否是线程安全的
不是，spring 框架并没有对单例 bean 进行多线程的封装处理。
想改成线程安全，最简单的就是使用prototype，这样相当与每次都new一个bean

### @Autowired和@Resource之间的区别：
@Autowired默认是按照类型装配注入的，默认情况下它要求依赖对象必须存在（可以设置它required属性为false）
@Resource默认是按照名称来装配注入的，只有当找不到与名称匹配的bean才会按照类型来装配注入

### spring事务在什么情况下不生效
1. 数据库引擎不支持事务
2. 没有被 Spring 管理，如没有加 @Service 注解
3. 方法不是 public 的，@Transactional只能用于public方法上
4. 类内自身调用，默认只有在外部调用事务才会生效，原因是在spring中，当一个方法开启事务时，spring创建这个方法的类的bean对象，实际上创建的是代理对象。在代理bean对象中，一个方法调用本身的另一个方法，实则调用的代理对象的原始对象（不属于 spring bean）的方法，调用方法时不会去判断方法上的注解。解决方案之一就是在的类中注入自己，用注入的对象再调用另外一个方法，或者将被调用方法写到新的类中
5. 异常类型错误，默认回滚的是：RuntimeException，如果你想触发其他异常的回滚，需要在注解上配置一下，@Transactional(rollbackFor = Exception.class)

### spring事务的传播机制
#### 为什么会有传播机制
spring 对事务的控制，是使用 aop 切面实现的，我们不用关心事务的开始，提交，回滚，只需要在方法上加 @Transactional 注解，这时候就有问题了。
- serviceA 方法调用了 serviceB 方法，但两个方法都有事务，这个时候如果 serviceB 方法异常，是让 serviceB 方法提交，还是两个一起回滚。
- serviceA 方法调用了 serviceB 方法，但是只有 serviceA 方法加了事务，是否把 serviceB 也加入 serviceA 的事务，如果 serviceB 异常，是否回滚 serviceA 。
- serviceA 方法调用了 serviceB 方法，两者都有事务，serviceB 已经正常执行完，但 serviceA 异常，是否需要回滚 serviceB 的数据。
#### 传播机制生效条件
因为 spring 是使用 aop 来代理事务控制 ，是针对于接口或类的，所以在同一个 service 类中两个方法的调用，传播机制是不生效的
#### 传播机制类型
PROPAGATION_REQUIRED (默认)
- 支持当前事务，如果当前没有事务，则新建事务
- 如果当前存在事务，则加入当前事务，合并成一个事务

REQUIRES_NEW
- 新建事务，如果当前存在事务，则把当前事务挂起
- 这个方法会独立提交事务，不受调用者的事务影响，父级异常，它也是正常提交

NESTED
- 如果当前存在事务，它将会成为父级事务的一个子事务，方法结束后并没有提交，只有等父事务结束才提交
- 如果当前没有事务，则新建事务
- 如果它异常，父级可以捕获它的异常而不进行回滚，正常提交
- 但如果父级异常，它必然回滚，这就是和 REQUIRES_NEW 的区别

SUPPORTS
- 如果当前存在事务，则加入事务
- 如果当前不存在事务，则以非事务方式运行，这个和不写没区别

NOT_SUPPORTED
- 以非事务方式运行
- 如果当前存在事务，则把当前事务挂起

MANDATORY
- 如果当前存在事务，则运行在当前事务中
- 如果当前无事务，则抛出异常，也即父级方法必须有事务

NEVER
- 以非事务方式运行，如果当前存在事务，则抛出异常，即父级方法必须无事务

### spring 的事务隔离:
- ISOLATION_DEFAULT，用底层数据库的隔离级别
- ISOLATION_READ_UNCOMMITTED， 未提交读，事务未提交前，就可被其他事务读取，会出现幻读、脏读、不可重复读
- ISOLATION_READ_COMMITTED：提交读，一个事务提交后才能被其他事务读取到，会造成幻读、不可重复读，SQL server 的默认级别；
- ISOLATION_REPEATABLE_READ：可重复读，保证多次读取同一个数据时，其值都和事务开始时候的内容是一致，禁止读取到别的事务未提交的数据，可能会造成幻读，MySQL的默认级别；
- ISOLATION_SERIALIZABLE：序列化，代价最高最可靠的隔离级别，该隔离级别能防止脏读、不可重复读、幻读。

### Spring AOP里面的几个定义：
- 切面（Aspect）：切面是通知和切点的结合。通知和切点共同定义了切面的全部内容。 在Spring AOP中，切面可以使用通用类（基于模式的风格） 或者在普通类中以 @Aspect 注解来实现。
- 通知（Advice）：在AOP术语中，切面的工作被称为通知。Spring切面可以应用5种类型的通知：
  1. 前置通知（Before）：在目标方法被调用之前调用通知功能；
  2. 后置通知（After）：在目标方法完成之后调用通知，此时不会关心方法的输出是什么；
  3. 返回通知（After-returning ）：在目标方法成功执行之后调用通知；
  4. 异常通知（After-throwing）：在目标方法抛出异常后调用通知；
  5. 环绕通知（Around）：通知包裹了被通知的方法，在被通知的方法调用之前和调用之后执行自定义的行为。
- 切点（Pointcut）：匹配通知所要织入的一个或多个连接点。我们通常使用明确的类和方法名称，或是利用正则表达式定义所匹配的类和方法名称来指定这些切点。
- 织入（Weaving）：织入是把切面应用到目标对象并创建新的代理对象的过程。在目标对象的生命周期里有多少个点可以进行织入：
  1. 编译期：切面在目标类编译时被织入。AspectJ的织入编译器是以这种方式织入切面的。
  2. 类加载期：切面在目标类加载到JVM时被织入。需要特殊的类加载器，它可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持以这种方式织入切面。
  3. 运行期：切面在应用运行的某个时刻被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。SpringAOP就是以这种方式织入切面。
- 连接点（Join point）：指方法，在Spring AOP中，一个连接点总是代表一个方法的执行。应用可能有数以千计的时机通知。这些时机被称为连接点。连接点是在应用执行过程中能够插入切面的一个点。这个点可以是调用方法时、抛出异常时、甚至修改一个字段时。切面代码可以利用这些点插入到应用的正常流程之中，并添加新的行为。
- 引入（Introduction）：引入允许我们向现有类添加新方法或属性。
- 目标对象（Target Object）： 被一个或者多个切面（aspect）所通知（advise）的对象。它通常是一个代理对象。也有人把它叫做 被通知（adviced） 对象。 既然Spring AOP是通过运行时代理实现的，这个对象永远是一个 被代理（proxied） 对象。

### 同一个aspect，不同advice的执行顺序：
- 没有异常情况下的执行顺序：
  1.  around before advice
  2.  before advice
  3.  target method 执行
  4.  around after advice
  5.  after advice
  6.  afterReturning
- 有异常情况下的执行顺序：
  1.  around before advice
  2.  before advice
  3.  target method 执行
  4.  around after advice
  5.  after advice
  6.  afterThrowing:异常发生
  7.  java.lang.RuntimeException: 异常发生

### 切面配置示例：
```xml
<aop:config>
    <!--统一异常拦截 -->
    <aop:aspect id="exceptionAspect" ref="dtoParamCheckInterceptor">
        <aop:pointcut id="exceptionPointcut" expression="execution(* com.qunar.train.glory.service.impl.*.*(..))"/>
        <aop:around method="process"  pointcut-ref="exceptionPointcut"/>
    </aop:aspect>
</aop:config>
```

### aop底层实现
- JDK动态代理：核心是实现InvocationHandler接口，并且实现接口中的invoke方法
```java
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    // 程序执行前加入逻辑，MethodBeforeAdviceInterceptor
    System.out.println("before-----------------------------");
    // 程序执行
    Object result = method.invoke(target, args);
    // 程序执行后加入逻辑，MethodAfterAdviceInterceptor
    System.out.println("after------------------------------");
    return result;
}
```
使用语法
```java
Service aService = new ServiceImpl();
MyInvocationHandler handler = new MyInvocationHandler(aService);
// Proxy为InvocationHandler实现类动态创建一个符合某一接口的代理实例
Service aServiceProxy = (Service) Proxy.newProxyInstance(aService.getClass().getClassLoader(), aService.getClass().getInterfaces(), handler);
```
- cglib代理：核心是实现MethodInterceptor接口，实现intercept方法。该代理中在add方法前后加入了自定义的切面逻辑
```java
public Object intercept(Object object, Method method, Object[] args, MethodProxy proxy) throws Throwable {
    // 添加切面逻辑（advise），此处是在目标类代码执行之前，即为MethodBeforeAdviceInterceptor。
    System.out.println("before-------------");
    // 执行目标类add方法
    proxy.invokeSuper(object, args);
    // 添加切面逻辑（advise），此处是在目标类代码执行之后，即为MethodAfterAdviceInterceptor。
    System.out.println("after--------------");
    return null;
}
```
还需要获取增强的目标类的工厂Factory，其中增强的方法类对象是由Enhancer来实现的：
```java
public static Base getInstance(CglibProxy proxy) {
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(Base.class);
    //回调方法的参数为代理类对象CglibProxy，最后增强目标类调用的是代理类对象CglibProxy中的intercept方法
    enhancer.setCallback(proxy);
    //此刻，base不是单纯的目标类，而是增强过的目标类
    Base base = (Base) enhancer.create();
    return base;
}
```
使用语法：
```java
CglibProxy proxy = new CglibProxy();
// base为生成的增强过的目标类
Base base = Factory.getInstance(proxy);
base.add();
```

### springMVC工作流程
- 用户发送请求至前端控制器DispatcherServlet；
- DispatcherServlet收到请求后，调用HandlerMapping处理器映射器，请求获取Handler；
- 处理器映射器根据请求url找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet；
- DispatcherServlet 调用 HandlerAdapter处理器适配器；
- HandlerAdapter 经过适配调用 具体处理器(Handler，也叫后端控制器)；
- Handler执行完成返回ModelAndView；
- HandlerAdapter将Handler执行结果ModelAndView返回给DispatcherServlet；
- DispatcherServlet将ModelAndView传给ViewResolver视图解析器进行解析；
- ViewResolver解析后返回具体View；
- DispatcherServlet对View进行渲染视图（即将模型数据填充至视图中）
- DispatcherServlet响应用户。

### 什么是 Spring Boot？
Spring Boot 是 Spring 开源组织下的子项目，是 Spring 组件一站式解决方案，主要是简化了使用 Spring 的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手。

### 注解开发

### mybatis一级缓存
- 一级缓存是SqlSession级别的缓存。在操作数据库时需要构造 sqlSession对象，在对象中有一个数据结构（HashMap）用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互相不影响的。
- 一级缓存的作用域是同一个SqlSession，在同一个sqlSession中两次执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询，从而提高查询效率。当一个sqlSession结束后该sqlSession中的一级缓存也就不存在了。Mybatis默认开启一级缓存。
- 一级缓存只是相对于同一个SqlSession而言。所以在参数和SQL完全一样的情况下，我们使用同一个SqlSession对象调用一个Mapper方法，往往只执行一次SQL，因为使用SelSession第一次查询后，MyBatis会将其放在缓存中，以后再查询的时候，如果没有声明需要刷新，并且缓存没有超时的情况下，SqlSession都会取出当前缓存的数据，而不会再次发送SQL到数据库。
#### 一级缓存的生命周期有多长
- MyBatis在开启一个数据库会话时，会 创建一个新的SqlSession对象，SqlSession对象中会有一个新的Executor对象。Executor对象中持有一个新的PerpetualCache对象；当会话结束时，SqlSession对象及其内部的Executor对象还有PerpetualCache对象也一并释放掉。
- 如果SqlSession调用了close()方法，会释放掉一级缓存PerpetualCache对象，一级缓存将不可用。
- 如果SqlSession调用了clearCache()，会清空PerpetualCache对象中的数据，但是该对象仍可使用。
- SqlSession中执行了任何一个update操作(update()、delete()、insert()) ，都会清空PerpetualCache对象的数据，但是该对象可以继续使用
#### 怎么判断某两次查询是完全相同的查询
如果以下条件都完全一样，那么就认为它们是完全相同的两次查询。
1. 传入的statementId
2. 查询时要求的结果集中的结果范围
3. 这次查询所产生的最终要传递给JDBC java.sql.Preparedstatement的Sql语句字符串（boundSql.getSql() ）
4. 传递给java.sql.Statement要设置的参数值

### mybatis二级缓存
MyBatis的二级缓存是Application级别的缓存，它可以提高对数据库查询的效率，以提高应用的性能。
- 二级缓存是mapper级别的缓存，多个SqlSession去操作同一个Mapper的sql语句，多个SqlSession去操作数据库得到数据会存在二级缓存区域，多个SqlSession可以共用二级缓存
- 二级缓存是多个SqlSession共享的，其作用域是mapper的同一个namespace，不同的sqlSession两次执行相同namespace下的sql语句且向sql中传递参数也相同即认为最终执行相同的sql语句，第一次执行完毕会将数据库中查询的数据写到缓存（内存），第二次会从缓存中获取数据将不再从数据库查询。
- 实现二级缓存的时候，MyBatis要求返回的POJO必须是可序列化的。 也就是要求实现Serializable接口，Mybatis默认没有开启二级缓存，需要在setting全局参数中配置开启二级缓存。配置方法很简单，只需要在映射XML文件配置就可以开启缓存了\<setting name="cacheEnabled" value="true"\/\>
#### 如果我们配置了二级缓存就意味着：
- 映射语句文件中的所有select语句将会被缓存。
- 映射语句文件中的所欲insert、update和delete语句会刷新缓存。
- 缓存会使用默认的Least Recently Used（LRU，最近最少使用的）算法来收回。
- 根据时间表，比如No Flush Interval,（CNFI没有刷新间隔），缓存不会以任何时间顺序来刷新。
- 缓存会存储列表集合或对象(无论查询方法返回什么)的1024个引用
- 缓存会被视为是read/write(可读/可写)的缓存，意味着对象检索不是共享的，而且可以安全的被调用者修改，不干扰其他调用者或线程所做的潜在修改。
