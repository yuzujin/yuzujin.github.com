---
layout: post
title: "Spring AOP介绍"
description: ""
category: [Java Web]
tags: [AOP]
---
{% include JB/setup %}

AOP（Aspect Oriented Programming），即面向切面编程，可以说是OOP（Object Oriented Programming，面向对象编程）的补充和完善。OOP引入封装、继承、多态等概念来建立一种对象层次结构，用于模拟公共行为的一个集合。不过OOP允许开发者定义纵向的关系，但并不适合定义横向的关系，例如日志功能。日志代码往往横向地散布在所有对象层次中，而与它对应的对象的核心功能毫无关系对于其他类型的代码，如安全性、异常处理和透明的持续性也都是如此，这种散布在各处的无关的代码被称为横切（cross cutting），在OOP设计中，它导致了大量代码的重复，而不利于各个模块的重用。

AOP技术恰恰相反，它利用一种称为"横切"的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其命名为"Aspect"，即切面。所谓"切面"，简单说就是那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，并有利于未来的可操作性和可维护性。

使用"横切"技术，AOP把软件系统分为两个部分：核心关注点和横切关注点。业务处理的主要流程是核心关注点，与之关系不大的部分是横切关注点。横切关注点的一个特点是，他们经常发生在核心关注点的多处，而各处基本相似，比如权限认证、日志、事物。AOP的作用在于分离系统中的各种关注点，将核心关注点和横切关注点分离开来。

下面引用逗逼网友的通俗解释：
	
	aop切面编程就是在常规的执行java类中方法前或执行后加入自定义的方法。比如你本来每天都去打酱油，去，
	打酱油，回。现在我每天在你打酱油路上等着，你去打酱油的时候我打你一顿，回来的时候给你点糖果吃。
	你根本不知道为什么我会在路上拦住打你。所以在切面中插入你自定义的方法，这个方法的执行和本身要执行的
	类方法无关系，也就是不是这个类的方法来调用你写的方法的，你写的方法什么时候执行都是要通过在配置指定。
	我打完你，你该打酱油还是去打酱油，当然我如果是拦住你让你酱油打少点，你打酱油的时候还是会打那么多，
	但是在你打完酱油回来的时候我可以把你的酱油倒些出去，所以嵌入的自定义方法对要调用的类方法本身没有影响，
	但是可以操纵这个方法的返结果或者处理结果。

**AOP核心概念**

1、横切关注点

对哪些方法进行拦截，拦截后怎么处理，这些关注点称之为横切关注点

2、切面（aspect）

类是对物体特征的抽象，切面就是对横切关注点的抽象

3、连接点（joinpoint）

被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的就是被拦截到的方法，实际上连接点还可以是字段或者构造器

4、切入点（pointcut）

对连接点进行拦截的定义

5、通知（advice）

所谓通知指的就是指拦截到连接点之后要执行的代码，通知分为前置、后置、异常、最终、环绕通知五类

6、目标对象

代理的目标对象

7、织入（weave）

将切面应用到目标对象并导致代理对象创建的过程

8、引入（introduction）

在不修改代码的前提下，引入可以在运行期为类动态地添加一些方法或字段

**AOP使用场景**

AOP用来封装横切关注点，具体可以在下面的场景中使用:
 
1. Authentication 权限
2. Caching 缓存
3. Context passing 内容传递
4. Error handling 错误处理
5. Lazy loading　懒加载
6. Debugging　　调试
7. logging, tracing, profiling and monitoring　记录跟踪　优化　校准
8. Performance optimization　性能优化
9. Persistence　　持久化
10. Resource pooling　资源池
11. Synchronization　同步
12. Transactions 事务

实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。

**如何使用Spring AOP**

可以通过配置文件或者编程的方式来使用Spring AOP。
 
配置可以通过xml文件来进行，大概有四种方式：

	1. 基于代理的AOP(配置ProxyFactoryBean，显式地设置advisors, advice, target等)
	或AutoProxyCreator(这种方式下，还是如以前一样使用定义的bean，但是从容器中获得的其实已经是代理对象)
	2. 纯POJO切面(通过<aop:config>来配置)
	3. @AspectJ注解驱动的切面(通过<aop:aspectj-autoproxy>来配置，使用AspectJ的注解来标识通知及切入点)
	4. 注入式AspectJ切面
   
**示例**

*基于代理的AOP*

Spring支持五种类型的通知：

- Before(前)  org.apringframework.aop.MethodBeforeAdvice
- After-returning(返回后) org.springframework.aop.AfterReturningAdvice
- After-throwing(抛出后) org.springframework.aop.ThrowsAdvice
- Arround(周围) org.aopaliance.intercept.MethodInterceptor
- Introduction(引入) org.springframework.aop.IntroductionInterceptor

具体步骤：

1. 创建通知：实现这几个接口，把其中的方法实现了
2. 定义切点和通知者：在Spring配制文件中配置这些信息
3. 使用ProxyFactoryBean来生成代理

代码：

	接口
	public interface Eatable {
    	void eat(); 
	}
		
	目标类
	public Human implements Eatable {
		public void eat(){
      		System.out.println("吃饭");
      	}
    }
    applicationContext.xml中配置目标类
    <bean id="human" class="com.test.aop.bean.Human">
      	
    通知类
    public class EatHelper implements MethodBeforeAdvice,AfterReturningAdvice {
		public void before(Method mtd, Object[] arg1, Object arg2)
        	throws Throwable {
        		System.out.println("饭前洗手");
    	}

    	public void afterReturning(Object arg0, Method arg1, Object[] arg2,
            Object arg3) throws Throwable {
        		System.out.println("饭后漱口");
    	}
    }
    	
    applicationContext.xml中配置通知bean
    <bean id="eatHelper" class="com.test.aop.bean.EatHelper"></bean>
    	
    配置切点
    <bean id="eatPointcut" class="org.springframework.aop.support.JdkRegexpMethodPointcut">
    	<property name="pattern" value=".*eat"/>
	</bean>
		
	结合通知和切点
	<bean id="eatHelperAdvisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
    	<property name="advice" ref="eatHelper"/>
     	<property name="pointcut" ref="eatPointcut"/>
    </bean>
     	
    调用ProxyFactoryBean产生代理对象
    <bean id="humanProxy" class="org.springframework.aop.framework.ProxyFactoryBean">
     	<property name="target" ref="human"/>
     	<property name="interceptorNames" value="eatHelperAdvisor" />
     	<property name="proxyInterfaces" value="com.test.aop.bean.Eatable" />
	</bean>
	ProxyFactoryBean是一个代理，我们可以把它转换为proxyInterfaces中指定的实现该interface的代理对象
		
	测试
	public class Test {
		public static void main(String[] args){
       		ApplicationContext appCtx = new ClassPathXmlApplicationContext("applicationContext.xml");
       		Eatable eater = (Eatable)appCtx.getBean("humanProxy");
       		eater.eat();
    	}
	}
		
	结果：
		饭前洗手
		吃饭
		饭后漱口
      	
      	
配置切点跟通知的地方,Spring提供了一种自动代理的功能，能让切点跟通知自动进行匹配，修改配置文件如下:

	 <bean id="eatHelper" class="com.test.aop.bean.EatHelper"></bean>
	 <bean id="human" class="com.test.aop.bean.Human">
	 <bean id="eatHelperAdvisor" class="org.springframework.aop.support.RegexpMethodPointcutAdvisor">
    	<property name="advice" ref="eatHelper"/>
     	<property name="pattern" value=".*eat"/>
     </bean>
     <bean id="human" class="com.test.aop.bean.Human"></bean>
     <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"/>
    
     声明了DefaultAdvisorAutoProxyCreator就能为方法匹配的bean自动创建代理。
     
**@AspectJ提供的注解**

	@Aspect   // 标识切面
	public class EatHelper {
    	public EatHelper(){}
    
    	@Pointcut("execution(* *.eat())")  // 指定切点
    	public void eatPoint(){}
    
    	@Before("eatPoint()")   // 指定运行时通知
    	public void beforeEat(){
        	System.out.println("饭前洗手!");
    	}
    
    	@AfterReturning("eatPoint()")  // 指定运行时通知
    	public void afterEat(){
        	System.out.println("饭后漱口!");
    	}
	}
	
	// 配置applicationContext.xml
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/aop
						http://www.springframework.org/schema/aop/spring-aop-2.0.xsd"
	

	然后加上这个标签:
	<aop:aspectj-autoproxy/> 
	
**纯粹的POJO切面(最常用的方法)**

Spring在aop的命名空间里面还提供的配置元素:

- <aop:aspectj-autoproxy/><aop:advisor> 定义一个AOP通知者
- <aop:after> 后通知
- <aop:after-returning> 返回后通知
- <aop:after-throwing> 抛出后通知
- <aop:around> 周围通知
- <aop:aspect>定义一个切面
- <aop:before>前通知
- <aop:config>顶级配置元素，类似于<beans>这种东西
- <aop:pointcut>定义一个切点

代码不变，只是修改配置文件,加入AOP配置即可:
	
	<aop:config>
    	<aop:aspect ref="eatHelper">
    		<aop:before method="beforeEat" pointcut="execution(* *.eat(..))"/>
    		<aop:after method="afterEat" pointcut="execution(* *.eat(..))"/>
    	</aop:aspect>
	</aop:config>

假如增加一个打印日志的横切关注点

	public class LogHandler {
    	public void LogBefore() {
        	System.out.println("Log before method");
    	}
    	
    	public void LogAfter() {
        	System.out.println("Log after method");
    	}
	}
	
	applicationContext.xml配置文件需要修改
	
	<bean id="logHandler" class="com.test.aop.LogHandler" />
        
    <aop:config>
    	<aop:aspect id="eat" ref="eatHelper" order="1">
    		<aop:before method="beforeEat" pointcut="execution(* *.eat(..))"/>
    		<aop:after method="afterEat" pointcut="execution(* *.eat(..))"/>
    	</aop:aspect>
        <aop:aspect id="log" ref="logHandler" order="2">
        	<aop:pointcut id="printLog" expression="execution(* com.test.aop.Eatable.*(..))" />
            <aop:before method="LogBefore" pointcut-ref="printLog" />
            <aop:after method="LogAfter" pointcut-ref="printLog" />
        </aop:aspect>
    </aop:config>
    
 要想让logHandler在eatHelper前使用有两个办法：

（1）aspect里面有一个order属性，order属性的数字就是横切关注点的顺序

（2）把logHandler定义在eatHelper前面，Spring默认以aspect的定义顺序作为织入顺序
      
