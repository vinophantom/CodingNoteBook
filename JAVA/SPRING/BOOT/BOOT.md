# SpringBoot

## 注解

### @Value



> spring提供，`@Value` 就相当于传统 xml 配置文件中的 value 字段。

假设存在代码：

```java
@Component public class Person { 
@Value("i am name") private String name; 
} 
```

上面代码等价于的配置文件：

```xml
<bean class="Person"> <property name ="name" value="i am name"></property></bean> 
```

配置文件中的 value 的取值可以是：

- 字面量
- 通过 `${key}` 方式从环境变量中获取值
- 通过 `${key}` 方式全局配置文件中获取值
- `#{EL}`

> 所以，我们就可以通过 `@Value(${key})` 的方式获取全局配置文件中的指定配置项。



### **@ConfigurationProperties** 

如果我们需要取 N 个配置项，通过 @Value 的方式去配置项需要一个一个去取，这就显得有点 low 了。我们可以使用 `@ConfigurationProperties` 。

> SpringBoot提供，标有 `@ConfigurationProperties` 的类的所有属性和配置文件中相关的配置项进行绑定。（默认从全局配置文件中获取配置值），绑定之后我们就可以通过这个类去访问全局配置文件中的属性值了。

下面看一个实例：

1. 在主配置文件

```properties
person.name=kundy 
person.age=13 
person.sex=male 
```

 2. 创建配置类

```java
@Component @ConfigurationProperties(prefix = "person") public class Person { 
private String name; private Integer age; private String sex; 
} 
```

> 这里 @ConfigurationProperties 有一个 `prefix` 参数，主要是用来指定该配置项在配置文件中的前缀。

### @Import

> `@Import` 注解支持导入普通 java 类，并将其声明成一个bean。主要用于将多个分散的 java config 配置类融合成一个更大的 config 类。

- `@Import` 注解在 4.2 之前只支持导入配置类。
- 在4.2之后 `@Import` 注解支持导入普通的 java 类,并将其声明成一个 bean。

**@Import 三种使用方式 **

- 直接导入普通的 Java 类。
- 配合自定义的 ImportSelector 使用。
- 配合 ImportBeanDefinitionRegistrar 使用。

#### **1. 直接导入普通的 Java 类**

1. 创建一个普通的 Java 类。

```java
public class Circle { 
public void sayHi() { System.out.println("Circle sayHi()"); } 
} 
```

1. 创建一个配置类，里面没有显式声明任何的 Bean，然后将刚才创建的 Circle 导入。

```java
@Import({Circle.class}) @Configuration public class MainConfig { 
} 
```

1. 创建测试类。

```java
public static void main(String[] args) { 
ApplicationContext context = new AnnotationConfigApplicationContext(MainConfig.class); Circle circle = context.getBean(Circle.class); circle.sayHi(); 
} 
```

1. 运行结果：

> Circle sayHi()

可以看到我们顺利的从 IOC 容器中获取到了 Circle 对象，证明我们在配置类中导入的 Circle 类，确实被声明为了一个 Bean。

#### **2. 配合自定义的 ImportSelector 使用**

> `ImportSelector` 是一个接口，该接口中只有一个 selectImports 方法，用于返回全类名数组。所以利用该特性我们可以给容器**动态导入 N 个** Bean。

1. 创建普通 Java 类 Triangle。

```java
public class Triangle { 
public void sayHi(){ System.out.println("Triangle sayHi()"); } 
}
```

1. 创建 ImportSelector 实现类，selectImports 返回 Triangle 的全类名。



```java
public class MyImportSelector implements ImportSelector { 
@Override public String[] selectImports(AnnotationMetadata annotationMetadata) { return new String[]{"annotation.importannotation.waytwo.Triangle"}; } 
} 
```

1. 创建配置类，在原来的基础上还导入了 MyImportSelector。

```java
@Import({Circle.class,MyImportSelector.class}) @Configuration public class MainConfigTwo { 
} 
```

1. 创建测试类

```java
public static void main(String[] args) { 
ApplicationContext context = new AnnotationConfigApplicationContext(MainConfigTwo.class); Circle circle = context.getBean(Circle.class); Triangle triangle = context.getBean(Triangle.class); circle.sayHi(); triangle.sayHi(); 
} 
```

1. 运行结果：

> Circle sayHi()
>
> Triangle sayHi()

可以看到 Triangle 对象也被 IOC 容器成功的实例化出来了。

#### **3. 配合 ImportBeanDefinitionRegistrar 使用**

> `ImportBeanDefinitionRegistrar` 也是一个接口，它可以**手动注册bean到容器中**，从而我们可以对类进行个性化的定制。(需要搭配 @Import 与 @Configuration 一起使用。）

1. 创建普通 Java 类 Rectangle。

```java
public class Rectangle { 
public void sayHi() { System.out.println("Rectangle sayHi()"); } 
}
```

1. 创建 ImportBeanDefinitionRegistrar 实现类，实现方法直接手动注册一个名叫 rectangle 的 Bean 到 IOC 容器中。

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar { 
@Override public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) { 
RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Rectangle.class); 
beanDefinitionRegistry.registerBeanDefinition("rectangle", rootBeanDefinition); } 
} 
```

1. 创建配置类，导入 MyImportBeanDefinitionRegistrar 类。

```java
@Import({Circle.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class}) @Configuration 
public class MainConfigThree { 
} 
```

1. 创建测试类。

```java
public static void main(String[] args) { 
ApplicationContext context = new AnnotationConfigApplicationContext(MainConfigThree.class); Circle circle = context.getBean(Circle.class); Triangle triangle = context.getBean(Triangle.class); Rectangle rectangle = context.getBean(Rectangle.class); circle.sayHi(); triangle.sayHi(); rectangle.sayHi(); 
} 
```

1. 运行结果

> Circle sayHi()
>
> Triangle sayHi()
>
> Rectangle sayHi()

嗯对，Rectangle 对象也被注册进来了。



### **@Conditional **

Spring提供

\> `@Conditional` 注释可以实现只有在特定条件满足时才启用一些配置。

下面看一个简单的例子：

1. 创建普通 Java 类 ConditionBean，该类主要用来验证 Bean 是否成功加载。

```java
public class ConditionBean { 
public void sayHi() { System.out.println("ConditionBean sayHi()"); } 
} 
```

1. 创建 Condition 实现类，@Conditional 注解只有一个 Condition 类型的参数，Condition 是一个接口，该接口只有一个返回布尔值的 matches() 方法，该方法返回 true 则条件成立，配置类生效。反之，则不生效。在该例子中我们直接返回 true。

```java
public class MyCondition implements Condition { 
@Override 
public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) { return true; } 
} 
```

创建配置类，可以看到该配置的 @Conditional 传了我们刚才创建的 Condition 实现类进去，用作条件判断。

```java
@Configuration 
@Conditional(MyCondition.class) 
public class ConditionConfig { 

    @Bean 
    public ConditionBean conditionBean(){ return new ConditionBean(); } 
} 
```

测试。

```java
public static void main(String[] args) { 
ApplicationContext context = new AnnotationConfigApplicationContext(ConditionConfig.class); ConditionBean conditionBean = context.getBean(ConditionBean.class); conditionBean.sayHi(); 
} 
```

1. 结果分析

因为 Condition 的 matches 方法直接返回了 true，配置类会生效，我们可以把 matches 改成返回 false，则配置类就不会生效了。

除了自定义 Condition，Spring 还为我们扩展了一些常用的 Condition。

| 扩展注解                    | 作用                             |
| --------------------------- | -------------------------------- |
| ConditionalOnBean           | 容器中存在指定 Bean，则生效。    |
| ConditionalOnMissingBean    | 容器中不存在指定 Bean，则生效。  |
| ConditionalOnClass          | 系统中有指定的类，则生效。       |
| ConditionalOnMissingClass   | 系统中没有指定的类，则生效。     |
| ConditionalOnProperty       | 系统中指定的属性是否有指定的值。 |
| ConditionalOnWebApplication | 当前是web环境，则生效。          |

 



## 启动过程

### SpringApplication 实例的初始化

对照代码来看：

![SpringApplication 实例的初始化代码](https://user-gold-cdn.xitu.io/2018/9/5/165a6ae37c345171?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

四个关键的步骤已标注在图中，分别解释如下：

- **①** 推断应用的类型：创建的是 REACTIVE应用、SERVLET应用、NONE 三种中的某一种

![推断应用的类型](https://user-gold-cdn.xitu.io/2018/9/5/165a6ae2e8305552?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- **②** 使用 `SpringFactoriesLoader`查找并加载 classpath下 `META-INF/spring.factories`文件中所有可用的 `ApplicationContextInitializer`

![ApplicationContextInitializer 加载](https://user-gold-cdn.xitu.io/2018/9/5/165a6ae2e6cd12b5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- **③** 使用 `SpringFactoriesLoader`查找并加载 classpath下 `META-INF/spring.factories`文件中的所有可用的 `ApplicationListener`

![ApplicationListener 加载](https://user-gold-cdn.xitu.io/2018/9/5/165a6ae2e6b68aeb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

- **④** 推断并设置 main方法的定义类

![推断并设置main方法的定义类](https://user-gold-cdn.xitu.io/2018/9/5/165a6ae2e8047512?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

### SpringApplication 的run()方法探秘

先看看代码长啥样子：

![SpringApplication 的run()方法背后的奥秘](https://user-gold-cdn.xitu.io/2018/9/5/165a6ae37e01211c?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

各个主要步骤我已经标注在上图之中了，除此之外，我也按照自己的理解画了一个流程图如下所示，可以对照数字标示看一下：

![SpringBoot 应用启动流程图](https://user-gold-cdn.xitu.io/2018/9/5/165a6ae37ed44681?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们将各步骤总结精炼如下：

1. 通过 `SpringFactoriesLoader` 加载 `META-INF/spring.factories` 文件，获取并创建 `SpringApplicationRunListener` 对象
2. 然后由 `SpringApplicationRunListener` 来发出 starting 消息
3. 创建参数，并配置当前 SpringBoot 应用将要使用的 Environment
4. 完成之后，依然由 `SpringApplicationRunListener` 来发出 environmentPrepared 消息
5. 创建 `ApplicationContext`
6. 初始化 `ApplicationContext`，并设置 Environment，加载相关配置等
7. 由 `SpringApplicationRunListener` 来发出 `contextPrepared` 消息，告知SpringBoot 应用使用的 `ApplicationContext` 已准备OK
8. 将各种 beans 装载入 `ApplicationContext`，继续由 `SpringApplicationRunListener` 来发出 contextLoaded 消息，告知 SpringBoot 应用使用的 `ApplicationContext` 已装填OK
9. refresh ApplicationContext，完成IoC容器可用的最后一步
10. 由 `SpringApplicationRunListener` 来发出 started 消息
11. 完成最终的程序的启动
12. 由 `SpringApplicationRunListener` 来发出 running 消息，告知程序已运行起来了

 

 

 

 

