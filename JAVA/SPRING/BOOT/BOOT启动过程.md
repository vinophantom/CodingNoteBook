## 主程序类

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        //run()传入的类必须是@SpringBootAplication注解的类
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

@**SpringBootApplication** ： 表示标注的类是主程序类。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
      @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
      @Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication {
```

@**SpringBootConfiguration**：标注在某个类上，表示这是一个Spring Boot 的配置类。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration
public @interface SpringBootConfiguration {
}
```

@**Configuration**：Spring 定义的注解，配置类也是一个组件。



@**EnableAutoConfiguration**： 开启自动配置功能。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)//将所有需要导入的组件以全类名的方式返回
public @interface EnableAutoConfiguration {
```



@**AutoConfigurationPackage**：自动配置包。由@Import 实现。

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import(AutoConfigurationPackages.Registrar.class)
public @interface AutoConfigurationPackage {
```