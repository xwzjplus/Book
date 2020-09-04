# 1 IOC

IoC（Inverse of Control）即控制反转

## 	1 IoC类型

主要分为构造函数注入，属性注入和接口注入，Spring支持构造函数注入和属性注入。

​		构造函数注入

```java
public class MoAttack {

    private GeLi geLi;

    public MoAttack() {
    }

    public MoAttack(GeLi geLi) {
        this.geLi = geLi;
    }

    void cityGateAsk() {
        geLi.responseAsk();
    }
}
```

​	属性注入

```java
public class MoAttack {

    private GeLi geLi;
    
    void cityGateAsk() {
        geLi.responseAsk();
    }

    public void setGeLi(GeLi geLi) {
        this.geLi = geLi;
    }
}
```

## 	2  反射

Spring能够通过容器完成依赖关系的注入，依靠java本身的反射功能。java允许通过程序化的方式间接对Class进行操作。

### 		 1简单示例

```java
public class Fruit {
    public String color;

    public String name;

    public int price;
    
     public Fruit() {
    }

    public Fruit(String color, String name, int price) {
        this.color = color;
        this.name = name;
        this.price = price;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    @Override
    public String toString() {
        return "Fruit{" +
                "color='" + color + '\'' +
                ", name='" + name + '\'' +
                ", price=" + price +
                '}';
    }
}
```

```java
public class ReflexTest {
    
    public static Fruit initByDefaultContent() throws ClassNotFoundException, NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        // 通过类装载器获取Fruit对象
        ClassLoader contextClassLoader = Thread.currentThread().getContextClassLoader();
        Class aClass = contextClassLoader.loadClass("com.spring.study.IoC.reflex.Fruit");

        // 获取类的默认构造器对象并通过它实例化Fruit
        Constructor constructor = aClass.getDeclaredConstructor((Class[]) null);
        Fruit fruit = (Fruit) constructor.newInstance();
		
        // 通过反射方法设置属性
        Method setColor = aClass.getMethod("setColor", String.class);
        setColor.invoke(fruit, "blue");
        return fruit;
    }

    public static void main(String[] args) throws ClassNotFoundException, NoSuchMethodException, InvocationTargetException, InstantiationException, IllegalAccessException {
        Fruit fruit = initByDefaultContent();
        System.out.println(fruit);
    }
}
```

在ReflectTest中通过获取当前线程的类装载器（**ClassLoader**），然后通过指定的全限定类名装载Fruit对象的反射实例，通过反射对象获取Fruit的构造对象，最后通过构造对象实例化Fruti对象，效果等同于new Fruit()。获取Fruit对象后通过调用setter方法调用方法设置属性。

### 		2 类装载器ClassLoader

类装载器就是寻找类的字节码文件并构造出类在JVM内部表示对象的的组件，在java中，类装载器把把类装入JVM中，需要以下步骤：

​	(1)装载：查找和导入Class文件。

​	(2)连接：执行校验，准备和解析步骤，其中解析步骤可选。

​						校验：检查载入Class文件数据的准确性。

​						准备：给类的静态变量分配存储空间。

​						解析：将符号引用转换成直接引用。				

```
	符号引用：
	以一组符号来描述所引用的目标, 符号可以是任何形式的字面量, 只要使用时能够无歧义的定位到目标即可. 例如, 在Java中, 一个Java类将会编译成一个class文件. 在编译时, Java类并不知道所引用的类的实际地址, 因此只能使用符号引用来代替. 比如org.simple.People类引用了org.simple.Language类, 在编译时People类并不知道Language类的实际内存地址, 因此只能使用符号org.simple.Language来表示Language类的地址；
	直接引用:
		直接指向目标的指针.(个人理解为: 指向方法区中类对象, 类变量和类方法的指针)
		相对偏移量. (指向实例的变量, 方法的指针)
		一个间接定位到对象的句柄.
```

(3)初始化：对类的静态对象，静态代码块执行初始化工作。

​	类装载工作由**ClassLoader**及其子类负责。ClassLoader是Java运行时系统组件，负责在运行时查找和装入Class字节码文件。

​	JVM在运行时会产生3个ClassLoader：**根装载器**、**ExtClassLoader**（扩展类装载器）和**AppClassLoader**（应用类装载器）。其中，根装载器不是ClassLoader的子类，是由C++编写，在JAVA中无法查到。根装载器负责装载JRE的核心类库，如JRE下的rt.jar,charsets.jar等；ExtClassLoader和AppClassLoader都是ClassLoader的子类，其中ExtClassLoader负责装载JRE扩展目录ext中的jar包，AppClassLoader负责装载ClassPath路径下的类包。

​	这3个类装载器之间存在父子层级关系，即根装载器是**ExtClassLoader**的父装载器，**ExtClassLoader**是**AppClassLoader**的父装载器，默认情况下，使用**AppClassLoader**装载应用程序的类。

​	JVM装载类使用“全盘负责委托机制”，“全盘负责”是指当一个**ClassLoader**装载一个类时，除非显式的使用另一个**ClassLoader**，该类所依赖及引用的类也由这个**ClassLoader**载入，“委托机制”是指先委托父装载器寻找目标类，只有找不到的情况下才从自己的类路径下寻找并装载目标类。

​	每个类在JVM中都拥有一个对象的java.lang.Class对象，提供了类结构信息的描述。



## 3 BeanFactory与ApplicationContext

Spring通过一个配置文件描述Bean及Bean之间的依赖关系，利用Java的反射机制实例化Bean并建立Bean之间的依赖关系。Spring的IoC容器在完成这些底层工作的基础上，还提供了Bean实例缓存，生命周期管理，Bean实例代理，事件发布，资源装载等高级服务。

Bean工厂（**org.springframework.beans.factory.BeanFactory**）是Spring框架最核心的接口，它提供了高级IoC的配置机制。**BeanFactory**使管理不同类型的Java对象成为可能。

应用上下文（**org.springframeword.context.ApplicationContext**）建立在**BeanFactory**基础之上，提供更多面向应用的功能。

二者简单区分：**BeanFactory**是Spring框居的基础设施，面向Spring本身；**ApplicationContext**面向使用Spring框架的开发者，几乎所有的应用场合都可以之间使用**ApplicationContext**而非底层的**BeanFactory**。

### 1 BeanFactory

![img](data:image/png;base64,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)

```java
package org.springframework.beans.factory;

public interface BeanFactory {

    /**
     * 用来引用一个实例，或把它和工厂产生的Bean区分开，就是说，如果一个FactoryBean的名字为a，那么，&a会得到那个Factory
     */
    String FACTORY_BEAN_PREFIX = "&";

    /*
     * 四个不同形式的getBean方法，获取实例
     */
    Object getBean(String name) throws BeansException;

    <T> T getBean(String name, Class<T> requiredType) throws BeansException;

    <T> T getBean(Class<T> requiredType) throws BeansException;

    Object getBean(String name, Object... args) throws BeansException;

    boolean containsBean(String name); // 是否存在

    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;// 是否为单实例

    boolean isPrototype(String name) throws NoSuchBeanDefinitionException;// 是否为原型（多实例）

    boolean isTypeMatch(String name, Class<?> targetType)
            throws NoSuchBeanDefinitionException;// 名称、类型是否匹配

    Class<?> getType(String name) throws NoSuchBeanDefinitionException; // 获取类型

    String[] getAliases(String name);// 根据实例的名字获取实例的别名

}
```

​	**BeanFactory**接口位于类结构树顶端，最主要的方法是getBean(String beanName)，该方法从容器中返回特定名称的Bean。**BeanFactory**的功能通过其他接口得到不断扩展。

​	**ListableBeanFactory**：该接口定义了访问容器中Bean基本信息的若干方法，如查看Bean个数，获取某一类型Bean的配置名，查看容器中是否包含某一Bean等

​	**HierarchicalBeanFactory**：父子级联IoC容器的接口，子容器可以通过接口方法访问父容器

​	**ConfigurableBeanFactory**：定义了设置类装载器，属性编辑器，容器初始化后置处理器等方法

​	**AutowireCapableBeanFactory**:定义了将容器中的Bean按某种规则（如按名字匹配，按类型匹配等）进行自动装配的方法

​	**SingletonBeanRegistry**：定义了允许在运行期间向容器注册单例Bean的方法

​	**BeanDefinitionRegistry**：提供向容器手动注册BeanDefinition对象的方法

​	初始化**BeanFactory**时，必须为其提供一种日志框架。

### 	2 ApplicationContext

​	**ApplicationContext**主要实现类是**ClassPathXmlApplication**和**FileSystemXmlApplicationContext**，前者默认从类路径加载配置文件，后者默认从文件夹系统装载配置文件。

![img](C:\Users\93776\Desktop\岚\spring.assets\10198961-15b4f9aa41f4ff8a.png)

**ApplicationContext**继承了**HierarchicalBeanFactory**和**ListableBeanFactory**接口，在此基础上通过其他接口扩展了**BeanFactory**的功能。

**ApplicationEventPublisher：**让容器拥有发布应用上下文事件的功能，包括容器启动事件，容器关闭事件等，实现了**ApplicationListener**事件监听接口的Bean可以接收到容器事件，并对事件进行响应处理。在**ApplicationContext**抽象实现类**AbstractApplicationContext**中存在一个**ApplicationEventMulticaster**，它负责保存所有的监听器，以便在容器产生上下文事件时通知这些事件监听者。

**MessageSource：**为应用提供i18n国际化消息访问的功能。

**ResourcePatternResolver：**为所有**ApplicationContext**实现类都实现了类似于**PathMatchingResourcePatternResolver**的功能，可以通过带前缀的Ant风格的资源文件装载Spring的配置文件。

**LifeCyle：**该接口提供start()和stop()两个方法，主要用于控制异步处理过程，在具体使用时，该接口同时呗ApplicationContext实现及具体Bean实现，**ApplicationContext**会将start/stop的信息传递给容器中所有实现了该接口的Bean，以达到管理和控制JMX、任务调度等目的。

**ConfigurableApplicationContext**：扩展于**ApplicationContext**，新增两个主要方法：refresh()和close()，让ApplicationContext具有启动，刷新和关闭应用上下文的能力。在应用上下文关闭的情况下调用refresh()即可启动应用上下文，在已经启动的情况下调用refresh()则可清除缓存并重新装载配置信息，而调用close()则可关闭应用上下文。

**ApplicationContext**初始化和**BeanFactory**初始化区别：**BeanFactory初始化容器时，并未实例化Bean，直到第一次访问某个Bean时才实例化目标Bean；而ApplicationContext则在初始化应用上下文时就实例化所有单实例Bean。**

**WebApplicationContext**：

​	**WebApplicationContext**专门为Web应用准备，允许从Web根目录的路径中装载配置文件完成初始化工作，从**WebApplicationContext**中可以获取**ServletContext**引用，整个Web应用上下文对象将作为属性放置到**ServletContext**中，以便Web应用环境可以访问Spring应用上下文。Spring专门提供工具类**WebApplicationContextUtils**，通过该类的getWebApplicationContext(ServletContext sc)方法，可以从**ServletContext**中获取**WebApplicationContext**实例。

![image-20200902143507920](C:\Users\93776\Desktop\岚\spring.assets\image-20200902143507920.png)

**ConfigurableWebApplicationContext**扩展了**WebApplicationContext**，允许通过配置的方式实例化**WebApplicationContext**，同时定义了两个重要方法：

setServletContext(ServletContext servletContext) 为Spring设置Web上下文，以便二者结合

setConfigLocations(String[] configLocations) 设置Spring配置文件地址

**WebApplicationContext**初始化需要**ServletContext**实例，即需要在拥有Web容器的前提下才能启动

### 3 父子容器

通过**HierarchicalBeanFactory**接口，Spring的IoC容器可以建立父子层级关联的容器体系，子容器可以访问父容器中的Bean，但父容器不能访问子容器的Bean，在容器内，Bean的id必须是唯一的，但子容器可以拥有一个和父容器id相同的Bean。

## 4 Bean生命周期  

在Spring中，可以从两个层面定义Bean的生命周期：第一个层面是Bean的作用范围；第二个层面是实例化Bean所经历的一系列阶段。

### 1 BeanFactory中Bean的生命周期

#### 	1 生命周期流程图

![image-20200902162345623](C:\Users\93776\Desktop\岚\spring.assets\image-20200902162345623.png)



#### 	2 各种接口方法分类

Bean的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类：

1、Bean自身的方法：这个包括了Bean本身调用的方法和通过配置文件中<bean>的init-method和destroy-method指定的方法。

2、Bean级生命周期接口方法：这个包括了**BeanNameAware**、**BeanFactoryAware**、**InitializingBean**和**DiposableBean**这些接口的方法，这些接口方法由Bean类直接实现。

3、容器级生命周期接口方法：这个包括了**InstantiationAwareBeanPostProcessor** 和 **BeanPostProcessor** 这两个接口实现，一般称它们的实现类为“后处理器”。后处理器接口一般不由Bean本身实现，他们独立于Bean，实现类以容器附加装置的形式注册到Spring容器中，并通过接口反射为Spring容器扫描识别。当Spring容器创建任何Bean的时候，这些后处理器都会发生作用，所以这些后处理器的影响是**全局性**的。

4、工厂后处理器接口方法：这个包括了**BeanFactoryPostProcessor** 等等非常有用的工厂后处理器接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。

Bean级生命周期接口和容器级生命周期接口是个性与共性辩证统一思想的体现，前者解决Bean个性化处理的问题，而后者解决容器中某些Bean共性化处理的问题。

若注册多个后处理器，可以通过实现（**org.springframework.core.Ordered**）接口，容器将按特定的顺序依次调用这些后处理器。

#### 	3 示例

首先是一个简单的Spring Bean，调用Bean自身的方法和Bean级生命周期接口方法，为了方便演示，它实现了**BeanNameAware**、**BeanFactoryAware**、**InitializingBean**和**DiposableBean**这4个接口，同时有2个方法，对应配置文件中<bean>的init-method和destroy-method。如下：

```java
public class People implements BeanFactoryAware, BeanNameAware,
        InitializingBean, DisposableBean {
    String name;
    String sex;
    int age;

    private BeanFactory beanFactory;

    private String beanName;

    public People() {
        System.out.println("构造函数初始化People对象");
    }

    public People(String name, String sex, int age) {
        this.name = name;
        this.sex = sex;
        this.age = age;
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()");
        this.beanFactory = beanFactory;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("【BeanNameAware接口】调用BeanNameAware.setBeanName()");
        this.beanName = name;
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("【DisposableBean接口】调用DisposableBean.destroy()");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("【InitializingBean接口】调用InitializingBean.afterPropertiesSet()");
    }


    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("注入属性Name");
        this.name = name;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        System.out.println("注入属性Sex");
        this.sex = sex;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        System.out.println("注入属性Age");
        this.age = age;
    }

    @Override
    public String toString() {
        return "People{" +
                "name='" + name + '\'' +
                ", sex='" + sex + '\'' +
                ", age=" + age +
                '}';
    }
}
```

演示工厂后处理器接口方法：

```java
public class PeopleBenFactoryPostProcessor implements BeanFactoryPostProcessor {

    public PeopleBenFactoryPostProcessor() {
        System.out.println("自定义【BeanFactoryPostProcessor】实现类构造器！！！");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanDefinition people = beanFactory.getBeanDefinition("people");
        people.getPropertyValues().addPropertyValue("name", "Wei");
        System.out.println("调用自定义【BeanFactoryPostProcessor】实现类的【postProcessBeanFactory】方法");
    }

}
```

**InstantiationAwareBeanPostProcessor**接口本质是**BeanPostProcessor**的子接口，一般我们继承Spring为其提供的适配器类**InstantiationAwareBeanPostProcessorAdapter**来使用它，这个有3个方法，其中第二个方法postProcessAfterInitialization就是重写了**BeanPostProcessor**的方法。第三个方法postProcessPropertyValues用来操作属性，返回值也应该是PropertyValues对象：

```java
public class PeopleInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessorAdapter {

    public PeopleInstantiationAwareBeanPostProcessor() {
        System.out.println("自定义【InstantiationAwareBeanPostProcessorAdapter】实现类构造器！！");
    }

    // 接口方法、实例化Bean之前调用
    @Override
    public Object postProcessBeforeInstantiation(Class beanClass,
                                                 String beanName) throws BeansException {
        if (beanName.equals("people")) {
            System.out.println("【InstantiationAwareBeanPostProcessor】调用【postProcessBeforeInstantiation】方法");
        }
        return null;
    }

    // 接口方法、实例化Bean之后调用
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("people")) {
            System.out.println("【InstantiationAwareBeanPostProcessor】调用【postProcessAfterInitialization方法】");
        }
        return bean;
    }

    // 接口方法、设置某个属性时调用
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("people")) {
            System.out.println("【InstantiationAwareBeanPostProcessor】调用【postProcessPropertyValues】方法");
        }
        return pvs;
    }

}
```

**BeanPostProcessor**接口包括2个方法postProcessAfterInitialization和postProcessBeforeInitialization，这两个方法的第一个参数都是要处理的Bean对象，第二个参数都是Bean的name，返回值也都是要处理的Bean对象，主要用于“补缺补漏”操作：

```java
public class PeoplePostProcessor implements BeanPostProcessor {

    public PeoplePostProcessor() {
        super();
        System.out.println("自定义【BeanPostProcessor】实现类构造器！！");
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("people")) {
            System.out.println("【BeanPostProcessor】接口方法【postProcessAfterInitialization】对属性进行更改！");
        }

        return bean;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("people")) {
            System.out.println("【BeanPostProcessor】接口方法【postProcessBeforeInitialization】对属性进行更改！");
        }
        return bean;
    }
}
```

使用xml配置将Bean注入到spring中：

```xml-dtd
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="people" class="com.spring.study.bean.initialization.People" init-method="initPeople" destroy-method="destroyPeople">
    </bean>
    
</beans>
```

测试类

```java
public class BeanLifeCycle {

    private static void lifeCycleInBeanFactory() {
        Resource resource = new ClassPathResource("beans.xml");

        DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();

        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);

        reader.loadBeanDefinitions(resource);

        new PeopleBenFactoryPostProcessor().postProcessBeanFactory(beanFactory);

        beanFactory.addBeanPostProcessor(new PeoplePostProcessor());
        beanFactory.addBeanPostProcessor(new PeopleInstantiationAwareBeanPostProcessor());


        People people = (People) beanFactory.getBean("people");
        System.out.println(people);

        beanFactory.destroySingletons();
    }

    public static void main(String[] args) {
        lifeCycleInBeanFactory();
    }

}
```

调用结果如下，与流程图一致：

```
 自定义【BeanFactoryPostProcessor】实现类构造器！！！
 调用自定义【BeanFactoryPostProcessor】实现类的【postProcessBeanFactory】方法
 自定义【InstantiationAwareBeanPostProcessorAdapter】实现类构造器！！
 自定义【BeanPostProcessor】实现类构造器！！
【InstantiationAwareBeanPostProcessor】调用【postProcessBeforeInstantiation】方法
 构造函数初始化People对象
【InstantiationAwareBeanPostProcessor】调用【postProcessPropertyValues】方法
 注入属性Name
【BeanNameAware】调用【setBeanName】
【BeanFactoryAware】调用【setBeanFactory】
【BeanPostProcessor】调用方法【postProcessBeforeInitialization】对属性进行更改！
【InitializingBean】调用【afterPropertiesSet】
【InstantiationAwareBeanPostProcessor】调用【postProcessAfterInitialization方法】
【BeanPostProcessor】调用方法【postProcessAfterInitialization】对属性进行更改！
【DisposableBean】调用【destroy()】
```

生命周期讨论：

**BeanFactoryPostProcessor**是用来处理修改bean定义信息的后置处理器，这个时候bean还没有初始化，只是定好了**BeanDefinition**。**BeanFactoryPostProcessor**的主要作用是让我们能接触到BeanDefinitions，对BeanDefinitions进行一定hack，但是也仅此而已了。**绝对不允许在BeanFactoryPostProcessor中触发到bean的实例化**。如果在这里getBean(""),就会把bean的初始化信息给提前了，这破坏了spring的封装特性。

**BeanPostProcessor**则是bean初始化前后对bean的一些操作，意思就是说bean在调用构造之后，初始化方法前后进行一些操作。

如果Bean是非单例的，则aop无法拦截，因为创建非单例作用域的bean，获取一次就创建一次，生命周期不交给Spring托管。

### 2 ApplicationContext中Bean生命周期

Bean在应用上下文中的生命周期与在**BeanFactory**中的生命周期类似，不同的是，如果Bean实现了（**org.springframework.context.ApplicationContextAware**）接口，则会增加调用该接口方法**setApplicationContext**() 的步骤。

**ApplicationContext**和**BeanFactory**另一个最大的不同之处在于：前者会利用Java反射机制自动识别出配置文件中定义的**BeanPostProcessor**、**InstantiationAwareBeanPostProcessor**和**BeanFactoryPostProcessor**，并自动将它们注册到应用上下文中，而后者需要在代码中通过手工调用addBeanPostProcessor()方法进行注册。

示例：

首先是一个简单的Spring Bean，调用Bean自身的方法和Bean级生命周期接口方法，为了方便演示，它实现了**BeanNameAware**、**BeanFactoryAware**、**InitializingBean**、**ApplicationContextAware**和**DiposableBean**

```java
public class Fruit implements BeanNameAware, BeanFactoryAware, InitializingBean, DisposableBean, ApplicationContextAware {

    private String name;

    private String color;

    private BeanFactory beanFactory;

    private String beanName;

    private ApplicationContext applicationContext;

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("【BeanFactoryAware接口】调用【setBeanFactory()】");
        this.beanFactory = beanFactory;
    }

    @Override
    public void setBeanName(String name) {
        System.out.println("【BeanNameAware接口】调用【setBeanName()】");
        this.beanName = name;
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("【DisposableBean接口】调用【destroy()】");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("【InitializingBean接口】调用【afterPropertiesSet()】");
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("【ApplicationContextAware】调用【setApplicationContext()】");
        this.applicationContext = applicationContext;
    }

    /**
     * 获取applicationContext
     *
     * @return
     */
    public ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    /**
     * 通过name获取 Bean.
     *
     * @param name
     * @return
     */
    public Object getBean(String name) {
        return getApplicationContext().getBean(name);
    }

    /**
     * 通过class获取Bean.
     *
     * @param clazz
     * @param <T>
     * @return
     */
    public <T> T getBean(Class<T> clazz) {
        return getApplicationContext().getBean(clazz);
    }

    /**
     * 通过name,以及Clazz返回指定的Bean
     *
     * @param name
     * @param clazz
     * @param <T>
     * @return
     */
    public <T> T getBean(String name, Class<T> clazz) {
        return getApplicationContext().getBean(name, clazz);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public BeanFactory getBeanFactory() {
        return beanFactory;
    }

    public String getBeanName() {
        return beanName;
    }
}
```

演示工厂后处理器接口方法：

```java
public class FruitBeanFactoryPostProcess implements BeanFactoryPostProcessor {

    public FruitBeanFactoryPostProcess() {
        System.out.println("自定义【BeanFactoryPostProcessor】实现类构造器！！！");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        System.out.println("调用自定义【BeanFactoryPostProcessor】实现类的【postProcessBeanFactory】方法");
        BeanDefinition people = beanFactory.getBeanDefinition("fruit");
        people.getPropertyValues().addPropertyValue("name", "Banana");
    }
}
```

实现**InstantiationAwareBeanPostProcessor**接口

```java
public class FruitInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessorAdapter {

    public FruitInstantiationAwareBeanPostProcessor() {
        System.out.println("自定义【InstantiationAwareBeanPostProcessorAdapter】实现类构造器！！");
    }

    // 接口方法、实例化Bean之前调用
    @Override
    public Object postProcessBeforeInstantiation(Class beanClass,
                                                 String beanName) throws BeansException {
        if (beanName.equals("fruit")) {
            System.out.println("【InstantiationAwareBeanPostProcessor】调用【postProcessBeforeInstantiation】方法");
        }
        return null;
    }

    // 接口方法、实例化Bean之后调用
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("fruit")) {
            System.out.println("【InstantiationAwareBeanPostProcessor】调用【postProcessAfterInitialization方法】");
        }
        return bean;
    }

    // 接口方法、设置某个属性时调用
    @Override
    public PropertyValues postProcessPropertyValues(PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("fruit")) {
            System.out.println("【InstantiationAwareBeanPostProcessor】调用【postProcessPropertyValues】方法");
        }
        return pvs;
    }

}
```

实现**BeanPostProcessor**接口

```java
public class FruitPostProcessor implements BeanPostProcessor {

    public FruitPostProcessor() {
        super();
        System.out.println("自定义【BeanPostProcessor】实现类构造器！！");
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("fruit")) {
            System.out.println("【BeanPostProcessor】接口方法【postProcessAfterInitialization】对属性进行更改！");
        }
        return bean;
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName)
            throws BeansException {
        if (beanName.equals("fruit")) {
            System.out.println("【BeanPostProcessor】接口方法【postProcessBeforeInitialization】对属性进行更改！");
        }
        return bean;
    }
}
```

以JavaConfig的方式注册Bean

```java
@Configuration
public class FruitConfig {


    @Bean
    public Fruit fruit() {
        return new Fruit();
    }

    @Bean
    public FruitBeanFactoryPostProcess fruitBeanFactoryPostProcess() {
        return new FruitBeanFactoryPostProcess();
    }


    @Bean
    public FruitInstantiationAwareBeanPostProcessor fruitInstantiationAwareBeanPostProcessor() {
        return new FruitInstantiationAwareBeanPostProcessor();
    }

    @Bean
    public FruitPostProcessor fruitPostProcessor() {
        return new FruitPostProcessor();
    }


}
```

编写测试类，注册Bean到容器中

```java
public class FruitTest {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(FruitConfig.class);
        Fruit fruit = applicationContext.getBean(Fruit.class);
        System.out.println(fruit);
    }
}
```

调用结果如下，与**BeanFactory**加载流程基本一致：**不清楚为何未触发销毁方法，后续补充**

```
自定义【BeanFactoryPostProcessor】实现类构造器！！！
调用自定义【BeanFactoryPostProcessor】实现类的【postProcessBeanFactory】方法
自定义【InstantiationAwareBeanPostProcessorAdapter】实现类构造器！！
自定义【BeanPostProcessor】实现类构造器！！
【InstantiationAwareBeanPostProcessor】调用【postProcessBeforeInstantiation】方法
【InstantiationAwareBeanPostProcessor】调用【postProcessPropertyValues】方法
【BeanNameAware接口】调用【setBeanName()】
【BeanFactoryAware接口】调用【setBeanFactory()】
【ApplicationContextAware】调用【setApplicationContext()】
【BeanPostProcessor】接口方法【postProcessBeforeInitialization】对属性进行更改！
【InitializingBean接口】调用【afterPropertiesSet()】
【InstantiationAwareBeanPostProcessor】调用【postProcessAfterInitialization方法】
【BeanPostProcessor】接口方法【postProcessAfterInitialization】对属性进行更改！
```

### 3 Bean初始化及销毁补充

Spring 允许 Bean 在**初始化完成后**以及**销毁前**执行特定的操作。下面是常用的三种指定特定操作的方法：

  	通过实现 InitializingBean / DisposableBean 接口；
  			通过<bean> 元素的 init-method / destroy-method属性；
  			通过@PostConstruct或@PreDestroy注解。

执行顺序

​	Bean在实例化的过程中：**Constructor > @PostConstruct >InitializingBean > init-method**

​	Bean在销毁的过程中：   **@PreDestroy > DisposableBean > destroy-method**

### 4 Bean名称重复问题

**DefaultListableBeanFactory**的registerBeanDefinition方法，名称重复的bean会覆盖

```java
@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}

		BeanDefinition existingDefinition = this.beanDefinitionMap.get(beanName);
		if (existingDefinition != null) {
			if (!isAllowBeanDefinitionOverriding()) {
				throw new BeanDefinitionOverrideException(beanName, beanDefinition, existingDefinition);
			}
			else if (existingDefinition.getRole() < beanDefinition.getRole()) {
				// e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
				if (logger.isInfoEnabled()) {
					logger.info("Overriding user-defined bean definition for bean '" + beanName +
							"' with a framework-generated bean definition: replacing [" +
							existingDefinition + "] with [" + beanDefinition + "]");
				}
			}
			else if (!beanDefinition.equals(existingDefinition)) {
				if (logger.isDebugEnabled()) {
					logger.debug("Overriding bean definition for bean '" + beanName +
							"' with a different definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
			else {
				if (logger.isTraceEnabled()) {
					logger.trace("Overriding bean definition for bean '" + beanName +
							"' with an equivalent definition: replacing [" + existingDefinition +
							"] with [" + beanDefinition + "]");
				}
			}
            // 覆盖同名Bean
			this.beanDefinitionMap.put(beanName, beanDefinition);
		}
		else {
			if (hasBeanCreationStarted()) {
				// Cannot modify startup-time collection elements anymore (for stable iteration)
				synchronized (this.beanDefinitionMap) {
					this.beanDefinitionMap.put(beanName, beanDefinition);
					List<String> updatedDefinitions = new ArrayList<>(this.beanDefinitionNames.size() + 1);
					updatedDefinitions.addAll(this.beanDefinitionNames);
					updatedDefinitions.add(beanName);
					this.beanDefinitionNames = updatedDefinitions;
					removeManualSingletonName(beanName);
				}
			}
			else {
				// Still in startup registration phase
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
			}
			this.frozenBeanDefinitionNames = null;
		}

		if (existingDefinition != null || containsSingleton(beanName)) {
			resetBeanDefinition(beanName);
		}
		else if (isConfigurationFrozen()) {
			clearByTypeCache();
		}
	}
```

## 5 装配Bean

### 1 Spring配置概述

要使应用程序中的Spring容器成功启动，需要同时具备以下三方面的条件：

​	（1）Spring框架的类包都已经放到应用程序的路径下

​	（2）应用程序为Spring提供了完备的Bean配置信息

​	（3）Bean的类都已经放到应用程序的类路径下

Spring启动时读取应用程序提供的Bean配置信息，并在Spring容器中生成一份相应的Bean配置注册表，如何根据注册表实例化Bean，装配好Bean之间的依赖关系，为上层应用提供准备就绪的运行环境。

Bean配置信息包含：Bean的实现类；Bean的属性信息；Bean的依赖关系；Bean的行为配置。

Bean元数据信息在Spring容器中的内部对应物是由一个个**BeanDefinition**形成的Bean注册表。

![img](C:\Users\93776\Desktop\岚\spring.assets\u=1344543480,3375101353&fm=26&gp=0.jpg)

Bean配置信息在容器内部建立Bean定义注册表，如何根据注册表加载、实例化Bean，并建立Bean和Bean之间的依赖关系，最好将这些准备就绪的Bean放到缓冲池中，以供外层应用程序进行调用。

Spring支持基于注解、XML、Java类以及Groovy的配置方式。

### 2 Bean基本配置

Bean的id必须是唯一的，而且id的命名要满足XML对id的命名规范：必须字母开头，后面可以是字母、数字、连字符、下划线、句号、冒号等完整结束的符号，逗号和空格等非完整结束符是非法的。name属性没用字符上的限制，几乎可以使用任何字符。id和name都可以指定多个名字，名字自建用逗号，分号或者空格分隔。name是可以重复的，但如果有多个name相同的Bean，那么通过getbean(beanName)获取的Bean为后加载的Bean，因为同名Bean会覆盖。

### 3 依赖注入

注入方式：属性注入，构造函数注入

循环依赖问题

​	Spring容器能对构造函数配置的Bean进行实例化有一个前提，即Bean构造函数入参引用的对象必须以及准备就绪。因此，如果两个Bean都采用构造函数注册，并且都要通过构造函数入参引用对方，就会产生类似于线程死锁的循环依赖问题。

### 4 自动装配

Spring提供4种自动装配类型的属性，byName、byType、constructor、autodetect。

@Autowired自动注入

### 5 方法注入

无状态Bean的作用域一般可以配置为singleton(单例模式)，如果我们往singleton的Bean中注入prototype的Bean，并且希望每次调单例对象都能返回新的Bean，使用传统的注入方式将无法实现这样的要求，因为单例Bean注入关联Bean的动作只有一次，虽然注入的Bean并非是单例，但返回的每次都是最开始注入的Bean。

解决方式：1.实现BeanFactoryAware接口  2.lookup方法注入

方法替换：实现MethodReplacer接口

### 6 Bean之间关系

继承，依赖，引用

### 7 Bean作用域

![Spring bean作用域和生命周期.png](C:\Users\93776\Desktop\岚\spring.assets\bVbJSdX)

###   8 FactoryBean

​	一般情况下，Spring通过反射机制利用bean的class属性指定实现类来实例化bean 。在某些情况下，实例化bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息，配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个**org.Springframework.bean.factory.FactoryBean**的工厂类接口，用户可以通过实现该接口定制实例化bean的逻辑

  实例：

​		首先编写一个简单的Bean

```java
public class Car {

    private String name;

    private String type;

    public Car() {
        System.out.println("实例化Car");
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Car{" +
                "name='" + name + '\'' +
                ", type='" + type + '\'' +
                '}';
    }
}
```

编写实现FactoryBean的类

```java
@Component
public class CarFactoryBean implements FactoryBean<Car> {

    @Override
    public Car getObject() throws Exception {
        Car car = new Car();
        car.setName("redFlag");
        return car;
    }

    @Override
    public Class<?> getObjectType() {
        return Car.class;
    }
}
```

编写测试类

```java
public class FactoryBeanTest {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(CarFactoryBean.class);
        Car bean = applicationContext.getBean(Car.class);
        System.out.println(bean);
    }
}
```

结果为

```
实例化Car
Car{name='redFlag', type='null'}
```

从上面的代码中，我们从Spring容器中获取了Car类型的Bean。我们先从getBean这个方法看起，因为在Spring的**AbstractApplicationContext**中有很多重载的getBean方法，这里我们调用的是根据Type(指Class类型)来获取的Bean信息。我们传入的type是Car类型。

**AbstractApplicationContext#getBean(java.lang.Class)**

```java
	@Override
	public <T> T getBean(Class<T> requiredType) throws BeansException {
        // 检测BeanFactory的激活状态
		assertBeanFactoryActive();
        // getBeanFactory()获取到的是一个DefaultListableBeanFactory的实例
		return getBeanFactory().getBean(requiredType);
	}
```

**DefaultListableBeanFactory#getBean(java.lang.Class)**

```java
	@Override
	public <T> T getBean(Class<T> requiredType) throws BeansException {
		return getBean(requiredType, (Object[]) null);
	}

	@SuppressWarnings("unchecked")
	@Override
	public <T> T getBean(Class<T> requiredType, @Nullable Object... args) throws BeansException {
		Assert.notNull(requiredType, "Required type must not be null");
		Object resolved = resolveBean(ResolvableType.forRawClass(requiredType), args, false);
        //如果都没有获取到，则抛出异常
		if (resolved == null) {
			throw new NoSuchBeanDefinitionException(requiredType);
		}
		return (T) resolved;
	}


	@Nullable
	private <T> T resolveBean(ResolvableType requiredType, @Nullable Object[] args, boolean nonUniqueAsNull) {
         // 解析Bean
		NamedBeanHolder<T> namedBean = resolveNamedBean(requiredType, args, nonUniqueAsNull);
		if (namedBean != null) {
			return namedBean.getBeanInstance();
		}
        //如果当前Spring容器中没有获取到相应的Bean信息，则从父容器中获取
		BeanFactory parent = getParentBeanFactory();
		if (parent instanceof DefaultListableBeanFactory) {
			return ((DefaultListableBeanFactory) parent).resolveBean(requiredType, args, nonUniqueAsNull);
		}
		else if (parent != null) {
            //一个重复的调用过程，只不过BeanFactory的实例变了
			ObjectProvider<T> parentProvider = parent.getBeanProvider(requiredType);
			if (args != null) {
				return parentProvider.getObject(args);
			}
			else {
				return (nonUniqueAsNull ? parentProvider.getIfUnique() : parentProvider.getIfAvailable());
			}
		}
		return null;
	}
	
```

重点查看resolveNamedBean方法

```java
@SuppressWarnings("unchecked")
	@Nullable
	private <T> NamedBeanHolder<T> resolveNamedBean(
			ResolvableType requiredType, @Nullable Object[] args, boolean nonUniqueAsNull) throws BeansException {

		Assert.notNull(requiredType, "Required type must not be null");
         // 这个方法是根据传入的Class类型来获取BeanName，因为我们有一个接口有多个实现类的情况(多态)，所以这里返回的是一个String数组。这个过程也比较复杂。
        // 这里需要注意的是，我们调用getBean方法传入的type为Car.Class类型，但是我们没有在Spring容器中注入Car类型的Bean
        // 正常来说我们在这里是获取不到beanName,但是实际上却获取到了
		String[] candidateNames = getBeanNamesForType(requiredType);
 		//如果有多个BeanName，则挑选合适的BeanName
		if (candidateNames.length > 1) {
			List<String> autowireCandidates = new ArrayList<>(candidateNames.length);
			for (String beanName : candidateNames) {
				if (!containsBeanDefinition(beanName) || getBeanDefinition(beanName).isAutowireCandidate()) {
					autowireCandidates.add(beanName);
				}
			}
			if (!autowireCandidates.isEmpty()) {
				candidateNames = StringUtils.toStringArray(autowireCandidates);
			}
		}
 		//如果只有一个BeanName 我们调用getBean方法来获取Bean实例来放入到NamedBeanHolder中
        //这里获取bean是根据beanName，beanType和args来获取bean
		if (candidateNames.length == 1) {
			String beanName = candidateNames[0];
			return new NamedBeanHolder<>(beanName, (T) getBean(beanName, requiredType.toClass(), args));
		}
        // 如果合适的BeanName还是有多个的话
		else if (candidateNames.length > 1) {
			Map<String, Object> candidates = new LinkedHashMap<>(candidateNames.length);
			for (String beanName : candidateNames) {
                // 看看是不是已经创建多的单例Bean
				if (containsSingleton(beanName) && args == null) {
					Object beanInstance = getBean(beanName);
					candidates.put(beanName, (beanInstance instanceof NullBean ? null : beanInstance));
				}
				else {
                    //调用getType方法继续获取Bean实例
					candidates.put(beanName, getType(beanName));
				}
			}
            //有多个Bean实例的话 则取带有Primary注解或者带有Primary信息的Bean
			String candidateName = determinePrimaryCandidate(candidates, requiredType.toClass());
			if (candidateName == null) {
                 //如果没有Primary注解或者Primary相关的信息，则去优先级高的Bean实例
				candidateName = determineHighestPriorityCandidate(candidates, requiredType.toClass());
			}
			if (candidateName != null) {
				Object beanInstance = candidates.get(candidateName);
                // Class类型的话 继续调用getBean方法获取Bean实例
				if (beanInstance == null || beanInstance instanceof Class) {
					beanInstance = getBean(candidateName, requiredType.toClass(), args);
				}
				return new NamedBeanHolder<>(candidateName, (T) beanInstance);
			}
			if (!nonUniqueAsNull) {
                //都没有获取到 抛出异常
				throw new NoUniqueBeanDefinitionException(requiredType, candidates.keySet());
			}
		}

		return null;
	}
```

getBeanNamesForType的分析

```java
@Override
	public String[] getBeanNamesForType(ResolvableType type) {
		return getBeanNamesForType(type, true, true);
	}

	@Override
	public String[] getBeanNamesForType(ResolvableType type, boolean includeNonSingletons, boolean allowEagerInit) {
		Class<?> resolved = type.resolve();
		if (resolved != null && !type.hasGenerics()) {
			return getBeanNamesForType(resolved, includeNonSingletons, allowEagerInit);
		}
		else {
			return doGetBeanNamesForType(type, includeNonSingletons, allowEagerInit);
		}
	}

	@Override
	public String[] getBeanNamesForType(@Nullable Class<?> type) {
		return getBeanNamesForType(type, true, true);
	}

	@Override
	public String[] getBeanNamesForType(@Nullable Class<?> type, boolean includeNonSingletons, boolean allowEagerInit) {
		if (!isConfigurationFrozen() || type == null || !allowEagerInit) {
			return doGetBeanNamesForType(ResolvableType.forRawClass(type), includeNonSingletons, allowEagerInit);
		}
        //先从缓存中获取
		Map<Class<?>, String[]> cache =
				(includeNonSingletons ? this.allBeanNamesByType : this.singletonBeanNamesByType);
		String[] resolvedBeanNames = cache.get(type);
		if (resolvedBeanNames != null) {
			return resolvedBeanNames;
		}
         // 调用doGetBeanNamesForType方法获取beanName
		resolvedBeanNames = doGetBeanNamesForType(ResolvableType.forRawClass(type), includeNonSingletons, true);
        //所传入的类能不能被当前类加载加载
		if (ClassUtils.isCacheSafe(type, getBeanClassLoader())) {
            //放入到缓存中，解析一次以后从缓存中获取
            //这里对应到我们这里 key是Car Value是CarBeanFactory
			cache.put(type, resolvedBeanNames);
		}
		return resolvedBeanNames;
	}
```

doGetBeanNamesForType的分析

```java
private String[] doGetBeanNamesForType(ResolvableType type, boolean includeNonSingletons, boolean allowEagerInit) {
		List<String> result = new ArrayList<>();

		// Check all bean definitions.
   		//循环所有的beanName 这个是在Spring容器启动解析Bean的时候放入到这个List中的
		for (String beanName : this.beanDefinitionNames) {
			// Only consider bean as eligible if the bean name
			// is not defined as alias for some other bean.
			if (!isAlias(beanName)) {
				try {
                     //根据beanName获取RootBeanDefinition
					RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
					// Only check bean definition if it is complete.
                    // RootBeanDefinition中的Bean不是抽象类、非延迟初始化
					if (!mbd.isAbstract() && (allowEagerInit ||
							(mbd.hasBeanClass() || !mbd.isLazyInit() || isAllowEagerClassLoading()) &&
									!requiresEagerInitForType(mbd.getFactoryBeanName()))) {
                        //是不是FactoryBean的子类
						boolean isFactoryBean = isFactoryBean(beanName, mbd);
						BeanDefinitionHolder dbd = mbd.getDecoratedDefinition();
						boolean matchFound = false;
						boolean allowFactoryBeanInit = allowEagerInit || containsSingleton(beanName);
						boolean isNonLazyDecorated = dbd != null && !mbd.isLazyInit();
                        //如果isTypeMatch这个方法返回true的话，我们会把这个beanName即carFactoryBean 放入到result中返回
						if (!isFactoryBean) {
							if (includeNonSingletons || isSingleton(beanName, mbd, dbd)) {
								matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
							}
						}
						else  {
							if (includeNonSingletons || isNonLazyDecorated ||
									(allowFactoryBeanInit && isSingleton(beanName, mbd, dbd))) {
								matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
							}
							if (!matchFound) {
								// In case of FactoryBean, try to match FactoryBean instance itself next.
                                // 如果不匹配，还是FactoryBean的子类 这里会把beanName变为 &beanName
								beanName = FACTORY_BEAN_PREFIX + beanName;
                                // 判断类型匹配不匹配
								matchFound = isTypeMatch(beanName, type, allowFactoryBeanInit);
							}
						}
						if (matchFound) {
							result.add(beanName);
						}
					}
				}
				catch (CannotLoadBeanClassException | BeanDefinitionStoreException ex) {
					if (allowEagerInit) {
						throw ex;
					}
					// Probably a placeholder: let's ignore it for type matching purposes.
					LogMessage message = (ex instanceof CannotLoadBeanClassException) ?
							LogMessage.format("Ignoring bean class loading failure for bean '%s'", beanName) :
							LogMessage.format("Ignoring unresolvable metadata in bean definition '%s'", beanName);
					logger.trace(message, ex);
					onSuppressedException(ex);
				}
			}
		}


		// Check manually registered singletons too.
    	//这里的Bean是Spring容器创建的特殊的几种类型的Bean 像Environment
		for (String beanName : this.manualSingletonNames) {
			try {
				// In case of FactoryBean, match object created by FactoryBean.
				if (isFactoryBean(beanName)) {
					if ((includeNonSingletons || isSingleton(beanName)) && isTypeMatch(beanName, type)) {
						result.add(beanName);
						// Match found for this bean: do not match FactoryBean itself anymore.
						continue;
					}
					// In case of FactoryBean, try to match FactoryBean itself next.
					beanName = FACTORY_BEAN_PREFIX + beanName;
				}
				// Match raw bean instance (might be raw FactoryBean).
				if (isTypeMatch(beanName, type)) {
					result.add(beanName);
				}
			}
			catch (NoSuchBeanDefinitionException ex) {
				// Shouldn't happen - probably a result of circular reference resolution...
				logger.trace(LogMessage.format("Failed to check manually registered singleton with name '%s'", beanName), ex);
			}
		}

		return StringUtils.toStringArray(result);
	}
```

在上面的代码中，我们可以知道的是我们的CarFactoryBean是一个**FactoryBean**类型的类。所以在上面的代码中会调用isTypeMatch这个方法来判断CarFactoryBean是不是和我们传入的类型相匹配，即Car.class。

isTypeMatch方法：

```java
protected boolean isTypeMatch(String name, ResolvableType typeToMatch, boolean allowFactoryBeanInit)
			throws NoSuchBeanDefinitionException {
//转换beanName   这里我们可以知道我们的beanName为carFactoryBean 因为上面是循环了Spring容器中的所有的Bean
		String beanName = transformedBeanName(name);
		boolean isFactoryDereference = BeanFactoryUtils.isFactoryDereference(name);
 		//因为我们这里是用的AbstractApplicationContext的子类来从Spring容器中获取Bean
        //获取beanName为carFactoryBean的Bean实例 这里是可以获取到Bean实例的
		// Check manually registered singletons.
		Object beanInstance = getSingleton(beanName, false);
		if (beanInstance != null && beanInstance.getClass() != NullBean.class) {
            // carFactoryBeanLearn是FactoryBean的一个实现类
			if (beanInstance instanceof FactoryBean) {
				if (!isFactoryDereference) {
                    // 这里就是从carFactoryBean中获type类型 
					Class<?> type = getTypeForFactoryBean((FactoryBean<?>) beanInstance);
                    //从carFactoryBean中获取到的type类型和我们传入的类型是不是同一种类型 是的话直接返回
					return (type != null && typeToMatch.isAssignableFrom(type));
				}
				else {
					return typeToMatch.isInstance(beanInstance);
				}
			}
			else if (!isFactoryDereference) {
				if (typeToMatch.isInstance(beanInstance)) {
					// Direct match for exposed instance?
					return true;
				}
				else if (typeToMatch.hasGenerics() && containsBeanDefinition(beanName)) {
					// Generics potentially only match on the target class, not on the proxy...
					RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
					Class<?> targetType = mbd.getTargetType();
					if (targetType != null && targetType != ClassUtils.getUserClass(beanInstance)) {
						// Check raw class match as well, making sure it's exposed on the proxy.
						Class<?> classToMatch = typeToMatch.resolve();
						if (classToMatch != null && !classToMatch.isInstance(beanInstance)) {
							return false;
						}
						if (typeToMatch.isAssignableFrom(targetType)) {
							return true;
						}
					}
					ResolvableType resolvableType = mbd.targetType;
					if (resolvableType == null) {
						resolvableType = mbd.factoryMethodReturnType;
					}
					return (resolvableType != null && typeToMatch.isAssignableFrom(resolvableType));
				}
			}
			return false;
		}
		else if (containsSingleton(beanName) && !containsBeanDefinition(beanName)) {
			// null instance registered
			return false;
		}

		// No singleton instance found -> check bean definition.
		BeanFactory parentBeanFactory = getParentBeanFactory();
		if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
			// No bean definition found in this factory -> delegate to parent.
			return parentBeanFactory.isTypeMatch(originalBeanName(name), typeToMatch);
		}

		// Retrieve corresponding bean definition.
		RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
		BeanDefinitionHolder dbd = mbd.getDecoratedDefinition();

		// Setup the types that we want to match against
		Class<?> classToMatch = typeToMatch.resolve();
		if (classToMatch == null) {
			classToMatch = FactoryBean.class;
		}
		Class<?>[] typesToMatch = (FactoryBean.class == classToMatch ?
				new Class<?>[] {classToMatch} : new Class<?>[] {FactoryBean.class, classToMatch});


		// Attempt to predict the bean type
		Class<?> predictedType = null;

		// We're looking for a regular reference but we're a factory bean that has
		// a decorated bean definition. The target bean should be the same type
		// as FactoryBean would ultimately return.
		if (!isFactoryDereference && dbd != null && isFactoryBean(beanName, mbd)) {
			// We should only attempt if the user explicitly set lazy-init to true
			// and we know the merged bean definition is for a factory bean.
			if (!mbd.isLazyInit() || allowFactoryBeanInit) {
				RootBeanDefinition tbd = getMergedBeanDefinition(dbd.getBeanName(), dbd.getBeanDefinition(), mbd);
				Class<?> targetType = predictBeanType(dbd.getBeanName(), tbd, typesToMatch);
				if (targetType != null && !FactoryBean.class.isAssignableFrom(targetType)) {
					predictedType = targetType;
				}
			}
		}

		// If we couldn't use the target type, try regular prediction.
		if (predictedType == null) {
			predictedType = predictBeanType(beanName, mbd, typesToMatch);
			if (predictedType == null) {
				return false;
			}
		}

		// Attempt to get the actual ResolvableType for the bean.
		ResolvableType beanType = null;

		// If it's a FactoryBean, we want to look at what it creates, not the factory class.
		if (FactoryBean.class.isAssignableFrom(predictedType)) {
			if (beanInstance == null && !isFactoryDereference) {
				beanType = getTypeForFactoryBean(beanName, mbd, allowFactoryBeanInit);
				predictedType = beanType.resolve();
				if (predictedType == null) {
					return false;
				}
			}
		}
		else if (isFactoryDereference) {
			// Special case: A SmartInstantiationAwareBeanPostProcessor returned a non-FactoryBean
			// type but we nevertheless are being asked to dereference a FactoryBean...
			// Let's check the original bean class and proceed with it if it is a FactoryBean.
			predictedType = predictBeanType(beanName, mbd, FactoryBean.class);
			if (predictedType == null || !FactoryBean.class.isAssignableFrom(predictedType)) {
				return false;
			}
		}

		// We don't have an exact type but if bean definition target type or the factory
		// method return type matches the predicted type then we can use that.
		if (beanType == null) {
			ResolvableType definedType = mbd.targetType;
			if (definedType == null) {
				definedType = mbd.factoryMethodReturnType;
			}
			if (definedType != null && definedType.resolve() == predictedType) {
				beanType = definedType;
			}
		}

		// If we have a bean type use it so that generics are considered
		if (beanType != null) {
			return typeToMatch.isAssignableFrom(beanType);
		}

		// If we don't have a bean type, fallback to the predicted type
		return typeToMatch.isAssignableFrom(predictedType);
	}
```

getTypeForFactoryBean：

```java
	@Nullable
	protected Class<?> getTypeForFactoryBean(final FactoryBean<?> factoryBean) {
		try {
			if (System.getSecurityManager() != null) {
				return AccessController.doPrivileged((PrivilegedAction<Class<?>>)
						factoryBean::getObjectType, getAccessControlContext());
			}
			else {
                // 调用FactoryBean实例的getObjectType()方法
				return factoryBean.getObjectType();
			}
		}
		catch (Throwable ex) {
			// Thrown from the FactoryBean's getObjectType implementation.
			logger.info("FactoryBean threw exception from getObjectType, despite the contract saying " +
					"that it should return null if the type of its object cannot be determined yet", ex);
			return null;
		}
	}
```

我们调用getBean(Class requiredType)方法根据类型来获取容器中的bean的时候，根据类型Car来从Spring容器中获取Bean，(首先明确的一点是在Spring容器中没有Car类型的**BeanDefinition**。但是却有一个Bean和Car这个类型有一些关系)。**Spring在根据type去获取Bean的时候，会先获取到beanName**。获取beanName的过程是：先循环Spring容器中的所有的beanName，然后根据beanName获取对应的**BeanDefinition**，如果当前bean是**FactoryBean**的类型，则会从Spring容器中根据beanName获取对应的Bean实例，接着调用获取到的Bean实例的getObjectType方法获取到Class类型，判断此Class类型和我们传入的Class是否是同一类型。如果是则返回测beanName，对应到我们这里就是：根据carFactoryBean获取到CarFactoryBean实例，调用CarFactoryBean的getObjectType方法获取到返回值Car.class。和我们传入的类型一致，**所以这里获取的beanName为carfactoryBean**。换句话说这里我们把carfactoryBean这个beanName映射为了：Car类型。**即Car类型对应的beanName为carfactoryBean。**

## 6 Spring 容器高级主题

### 1 内部工作机制

Spring的**AbstractApplicationContext**是**ApplicationContext**的抽象实现类，该抽象类的refresh()方法定义了Spring容器在加载配置文件后的各种处理过程。

**AbstractApplicationContext#refresh()**

```java
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
            // 一些准备工作，主要是一些状态的设置事件容器的初始化
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
            // 初始化BeanFactory
            // 这个方法里面调用了一个抽象的refreshBeanFactory方法刷新BeanFactory
            // 然后调用getBeanFactory()获取BeanFactory,
            // 在这一步里Spring将配置文件的信息装入容器的Bean定义注册表（BeanDefinitionRegistry）中
            // 但此时Bean还未初始化
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
            // 把拿到的beanFactory做一些准备，但是这个方法是一个protected的方法，
       	 	// 也就是说我们如果实现自己的spring启动类/或者spring团队需要写一个新的spring启动类的时候
       	 	// 是可以在beanFactory获取之后做一些事情的，算是一个钩子
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
                // 这也是一个钩子，在处理beanFactory前允许子类做一些事情
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
                // 根据反射机制从BeanDefinitionRegistry中找出所有实现了BeanFactoryPostProcessor接口的方法
                // 并且调用其postProcessorBeanFactory()的方法，
           		// 我们@Compoment等注解的收集处理主要就是在这里做的
            	// 有一个ConfigurationClassPostProcessor专门用来做这些注解支撑的工作
            	// 到这里为止，我们的beanDefinition的收集（注解/xml/其他来源...），注册（注册到beanFactory的beanDefinitionMap、beanDefinitionNames）工作基本就全部完成了
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
                // 从这里开始，我们就要专注bean的实例化了
           		// 根据反射机制从BeanDefinitionRegisty中找到所有实现了BeanPostProcess接口的Bean
                // 并将它们注册到容器Bean后处理器的注册表中
            	// 因为beanPostProcessor主要就是在bean实例化过程中，做一些附加操作的（埋点）
           		// 这个流程基本跟BeanFactoryPostProcessor的初始化是一样的，
            	// 排序，创建实例，然后放入一个list --> AbstractBeanFactory#beanPostProcessors
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
                // 初始化一些国际化相关的组件
				initMessageSource();

				// Initialize event multicaster for this context.
                // 初始化事件多播器
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
                // 也是个钩子方法，给子类创建一下特殊的bean
                // 子类可以借助这个方法执行一些特殊的操作如AbstarctRefreshableWebApplicationContext使用该方法执行初始化ThemeSource的操作
				onRefresh();

				// Check for listener beans and register them.
                // 注册事件监听器
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
                // 实例化所有的、非懒加载的单例bean
                // 初始化Bean后将他们放入Spring容器的缓冲池中
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
                // 初始化结束，清理资源，发送事件
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
                // 销毁已经注册的单例bean
				destroyBeans();

				// Reset 'active' flag.
                // 修改容器状态
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```

### 2 IOC流水线

![img](C:\Users\93776\Desktop\岚\spring.assets\u=426097473,3631386939&fm=26&gp=0.jpg)

（1）**ResourceLoader**从存储介质中加载Spring配置信息，并使用Resource表示这个配置文件资源

（2）**BeanDefinitionReader**读取Resource所指向的资源信息，将资源内的每个Bean解析成一个**BeanDefinition**对象，并保存到**BeanDefinitionRegistry**中

（3）容器扫描**BeanDefinitionRegistry**中的**BeanDefinition**，使用Java反射机制自动识别出Bean工程后处理器（实现**BeanFactoryPostProcessor**接口的Bean），然后调用这些Bean工厂后处理器对**BeanDefinitionRegistry**中的**BeanDefinition**进行加工处理，主要完成以下两个工作：

​		1.对一些半成品式的**BeanDefinition**对象进行加工处理并得到成品的**BeanDefinition**对象

​		2.对**BeanDefinitionRegistry**中的**BeanDefinition**进行扫描，通过Java反射机制找到所有属性编辑器的Bean（实现java.beans.PropertyEditor接口的Bean），并自动将它们注册到Spring容器的属性编辑器注册表中（**PropertyEditorRegistry**）

（4）Spring容器从**BeanDefinitionRegistry**中取出加工后的**BeanDefinition**，并调用**InstantiationStrategy**进行Bean实例化工作

（5）实例化Bean时，Spring容器使用**BeanWrapper**对Bean进行封装。**BeanWrapper**提供了以Java反射机制操作Bean的方法，结合Bean的**BeanDefinition**及容器中的属性编辑器完成Bean属性注入工作

（6）利用容器中注册的Bean后处理器（实现**BeanPostProcess**接口的Bean）对已经完成属性设置工作的Bean进行后续加工，之间装配出一个准备就绪的Bean

### 3 Spring组件