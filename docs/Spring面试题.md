## Spring

### 什么是三级缓存

1. 第一级缓存：单例缓存池singletonObjects。
2. 第二级缓存：早期提前暴露的对象缓存earlySingletonObjects。（属性还没有值对象也没有被初始化）
3. 第三级缓存：singletonFactories单例对象工厂缓存。

三级缓存详解：[根据 Spring 源码写一个带有三级缓存的 IOC](https://zhuanlan.zhihu.com/p/144627581)

### Spring如何解决循环依赖问题

Spring使用了三级缓存解决了循环依赖的问题。在populateBean()给属性赋值阶段里面Spring会解析你的属性，并且赋值，当发现，A对象里面依赖了B，此时又会走getBean方法，但这个时候，你去缓存中是可以拿的到的。因为我们在对createBeanInstance对象创建完成以后已经放入了缓存当中，所以创建B的时候发现依赖A，直接就从缓存中去拿，此时B创建完，A也创建完，一共执行了4次。至此Bean的创建完成，最后将创建好的Bean放入单例缓存池中。（非单例的实例作用域是不允许出现循环依赖）

## BeanFactory和ApplicationContext的区别

1. BeanFactory是Spring里面最低层的接口，提供了最简单的容器的功能，只提供了实例化对象和拿对象的功能。
2. ApplicationContext应用上下文，继承BeanFactory接口，它是Spring的一各更高级的容器，提供了更多的有用的功能。如国际化，访问资源，载入多个（有继承关系）上下文 ，使得每一个上下文都专注于一个特定的层次，消息发送、响应机制，AOP等。
3. BeanFactory在启动的时候不会去实例化Bean，中有从容器中拿Bean的时候才会去实例化。ApplicationContext在启动的时候就把所有的Bean全部实例化了。它还可以为Bean配置lazy-init=true来让Bean延迟实例化

#### BeanFactory
- BeanFactory是一个Bean工厂，使用简单工厂模式，是Spring IoC容器顶级接口，可以理解为含有Bean集合的工厂类，
- 作用是管理Bean，包括实例化、定位、配置对象及建立这些对象间的依赖。
- BeanFactory实例化后并不会自动实例化Bean，只有当Bean被使用时才实例化与装配依赖关系，属于延迟加载，适合多例模式。

#### FactoryBean
- FactoryBean是一个工厂Bean，使用了工厂方法模式，
- 作用是生产其他Bean实例，可以通过实现该接口，提供一个工厂方法来自定义实例化Bean的逻辑。
- FactoryBean 接口由BeanFactory中配置的对象实现，这些对象本身就是用于创建对象的工厂，如果一个Bean实现了这个接口，那么它就是创建对象的工厂Bean， 而不是Bean实例本身。
#### ApplicationConext
- ApplicationConext是BeanFactory的子接口，扩展了BeanFactory的功能，提供了支持国际化的文本消息，统一的资源文件读取方式，事件传播以及应用层的特别配置等。
- 容器会在初始化时对配置的Bean进行预实例化，Bean的依赖注入在容器初始化时就已经完成，属于立即加载，适合单例模式，一般推荐使用。



### 动态代理的实现方式，AOP的实现方式

1. JDK动态代理：利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。
2. CGlib动态代理：利用ASM（开源的Java字节码编辑库，操作字节码）开源包，将代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。
3. 区别：JDK代理只能对实现接口的类生成代理；CGlib是针对类实现代理，对指定的类生成一个子类，并覆盖其中的方法，这种通过继承类的实现方式，不能代理final修饰的类。

### @Transactional错误使用失效场景

1. @Transactional 在private上：当标记在protected、private、package-visible方法上时，不会产生错误，但也不会表现出为它指定的事务配置。可以认为它作为一个普通的方法参与到一个public方法的事务中。
2. @Transactional 的事务传播方式配置错误。
3. @Transactional 注解属性 rollbackFor 设置错误：Spring默认抛出了未检查unchecked异常（继承自 RuntimeException 的异常）或者 Error才回滚事务；其他异常不会触发回滚事务。
4. 同一个类中方法调用，导致@Transactional失效：由于使用Spring AOP代理造成的，因为只有当事务方法被当前类以外的代码调用时，才会由Spring生成的代理对象来管理。
5. 异常被 catch 捕获导致@Transactional失效。
6. 数据库引擎不支持事务。

### Spring中的事务传播机制

1. REQUIRED（默认，常用）：支持使用当前事务，如果当前事务不存在，创建一个新事务。eg:方法B用REQUIRED修饰，方法A调用方法B，如果方法A当前没有事务，方法B就新建一个事务（若还有C则B和C在各自的事务中独立执行），如果方法A有事务，方法B就加入到这个事务中，当成一个事务。
2. SUPPORTS：支持使用当前事务，如果当前事务不存在，则不使用事务。
3. MANDATORY：强制，支持使用当前事务，如果当前事务不存在，则抛出Exception。
4. REQUIRES_NEW（常用）：创建一个新事务，如果当前事务存在，把当前事务挂起。eg:方法B用REQUIRES_NEW修饰，方法A调用方法B，不管方法A上有没有事务方法B都新建一个事务，在该事务执行。
5. NOT_SUPPORTED：无事务执行，如果当前事务存在，把当前事务挂起。
6. NEVER：无事务执行，如果当前有事务则抛出Exception。
7. NESTED：嵌套事务，如果当前事务存在，那么在嵌套的事务中执行。如果当前事务不存在，则表现跟REQUIRED一样。

### Spring中Bean的生命周期

1. 实例化 Instantiation
2. 属性赋值 Populate
3. 初始化 Initialization
4. 销毁 Destruction

### Spring的后置处理器

1. BeanPostProcessor：Bean的后置处理器，主要在bean初始化前后工作。（before和after两个回调中间只处理了init-method）
2. InstantiationAwareBeanPostProcessor：继承于BeanPostProcessor，主要在实例化bean前后工作（TargetSource的AOP创建代理对象就是通过该接口实现）
3. BeanFactoryPostProcessor：Bean工厂的后置处理器，在bean定义(bean definitions)加载完成后，bean尚未初始化前执行。
4. BeanDefinitionRegistryPostProcessor：继承于BeanFactoryPostProcessor。其自定义的方法postProcessBeanDefinitionRegistry会在bean定义(bean definitions)将要加载，bean尚未初始化前真执行，即在BeanFactoryPostProcessor的postProcessBeanFactory方法前被调用。

## Spring Boot的核心注解
@SpringBootApplication注解包含
- @Configuration：允许在上下文中注册额外的 bean 或导入其他配置类
- @EnableAutoConfiguration：启用 SpringBoot 的自动配置机制
- @ComponentScan：扫描被@Component (@Service,@Controller)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean

## Spring、SpringMVC、Spring Boot、Spring Cloud
- Spring是一个轻量级的控制反转(IoC)和面向切面(AOP)的容器框架。Spring使你能够编写更干净、更可管理、并且更易于测试的代码。

- Spring MVC是Spring的一个模块，一个web框架。通过Dispatcher Servlet, ModelAndView 和 View Resolver，开发web应用变得很容易。主要针对的是网站应用程序或者服务开发——URL路由、Session、模板引擎、静态Web资源等等。

- Spring配置复杂，繁琐，所以推出了Spring boot，约定优于配置，简化了spring的配置流程。

- Spring Cloud构建于Spring Boot之上，是一个关注全局的服务治理框架。
### 总结
- Spring是核心，提供了基础功能；
- Spring MVC 是基于Spring的一个 MVC 框架 ；
- Spring Boot 是为简化Spring配置的快速开发整合包；
- Spring Cloud是构建在Spring Boot之上的服务治理框架。


### Spring MVC的工作流程（源码层面）

参考文章：[自己写个Spring MVC](https://zhuanlan.zhihu.com/p/139751932)
