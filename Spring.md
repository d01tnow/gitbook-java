## SpringBoot

Srpingboot 大大简化了开发过程.

开发步骤:

1. 使用 Spring Initializr 创建项目
2. 引入其他依赖
3. 编写配置
4. 编写接口和实现, 添加注解
5. 编写测试

# Spring

参考: Spring in action 4 

## 装配Bean

三种装配方案

* XML 中显示配置
* Java 代码中显示配置
* 隐式的 bean 发现机制和自动装配

### 自动化装配 bean

Spring 从两个角度实现自动装配:

* 组件扫描(component scanning): Spring 会自动发现应用上下文中按照约定创建的 bean.
* 自动装配(autowiring): Spring 自动满足 bean 之间的依赖

方法:

1. 用 @Component 注解标记类
2. 用 @ComponentScan 启用组件扫描
3. 用 @Autowired 注解装配

#### @Component 

* 默认以类名小驼峰为 bean 的ID, 可以通过 @Component("id") 方式重定义 bean id. java 注入规范中@Named 注解实现同样功能. 推荐使用 @Component

#### @ComponentScan

* 默认扫描当前包. 如果想要设置基础包, 可以通过 basePackages 属性配置. 比如: @ComponentScan(basePackages="sound"). 但是这种方式是类型不安全的. 还可以 basePackageClasses={CDPlayer, DVDPlayer}. 

* 推荐建立一个空标记接口, 可以保持对重构友好的接口引用. 

  ``` java
  
  @ComponentScan(basePackageClasses={Marker.class})
  public interface Marker {
    
  }
  ```

#### @Autowired

可以用在类的任何方法或属性上. Spring 会尝试满足方法参数上所声明的依赖. 如果没有匹配的 bean , 那么在创建应用上下文的时候, Spring会抛出异常. 可以通过 @Autowired(required=false) 关闭装配检查. 但是, 如果出现不匹配, 那么该属性处于未装配状态, 默认为 null.

@Autowired 是 Spring 特有注解. Java 注入规范里 @Inject 与之对应. @Autowired 使用的是  byref 方式, 即按类型匹配. 当需要按名称匹配时, 配合 @Qualifier 注解. 

#### @Qualifier

为 bean 定义限定名

#### @Resource

@Resource 不是 Spring 的注解, 是 Java 注解规范内的. 它即可以 by name, 又可以 by ref 方式匹配, 优先使用 by name 方式, 找不到再 by ref 方式扫描.



### 通过 Java 代码装配 bean

有些情况下需要使用Java 代码装配 bean. 比如: 你想要将第三方库中的组件装配到你的应用中. 这时, 你需要显示装配. Java代码或者XML.

JavaConfig 比 XML 方式好在类型安全, 对重构友好.

#### 创建配置类

创建一个类, 为其添加 @Configuration 注解. 该类中包含在 Spring 上下文中如何创建 bean 的细节. 

配置类中编写一个方法, 该方法创建所需类型的实例. 为该方法添加 @Bean 注解. 默认情况下, 返回的 bean 的ID 与带有 @Bean注解的方法名相同. 可以通过 @Bean(name="new_name") 方法重名名 bean的ID.

默认情况下, **Spring 中的 bean 都是单例的, 且不是线程安全的**. Spring 会拦截带有 @Bean 注解的方法的调用, 并确保直接返回该方法创建的 bean, 而不是每次都对其进行实际的调用.

推荐使用参数的方法: public CDPlayer cdPlayer(CompactDisc cd) . cdPlayer 方法能够将 CompactDisc 注入到 CDPlayer构造器中, 而且不用明确引用 CompactDisc 的@Bean 方法.



``` java
@Configuration
public class CDPlayerConfig {
  @Bean
  public CompactDisc sgtPeppers() {
    return new SgtPeppers();
  }
  @Bean
  public CDPlayer cdPlayer(CompactDisc cd) {
    return new CDPlayer(cd)
  }
}
```

#### Bean 生命周期

[参考](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/SpringInterviewQuestions)

![Spring Bean 生命周期](http://my-blog-to-use.oss-cn-beijing.aliyuncs.com/18-9-17/5496407.jpg)

### @Component VS @Bean

两者的区别是什么? [参考](https://snailclimb.gitee.io/javaguide/#/docs/system-design/framework/spring/SpringInterviewQuestions)

1. 作用对象不同. @Component 作用于类. @Bean 作用于方法.
2. @Component 通常是通过路径扫描来侦测, 自动装配到Spring 容器中. @Bean 告诉 Spring 这个方法返回某个类的实例, 当我需要的时候把它给我.
3. @Bean 比 @Component 的自定义性更强. 比如: 我们引用第三方库的类, 需要装配到 Spring 容器中时, 只能使用 @Bean 来实现.

## 高级装配

* Spring profile: 环境相关的配置
* 条件化的 bean 声明
* 自动装配与歧义性
* bean 的作用域
* Spring 表达式语言

### 环境与profile

通过在配置类或方法上使用 @Profile("env_flag"), 来标注该类用于某个环境下. 只有在环境 profile 激活时才会创建对应的 bean.

Spring 优先激活 spring.profiles.active 指定的 profile, 没有设置 active 时, 使用 spring.profiles.default 的值. 都没有指定的话, 那 spring 只创建没有定义在 profile 中的 bean.

多种方式设置这两个属性

* 作为 DispatcherServlet 的初始化参数
* 作为 Web 应用的上下文参数
* 作为环境变量
* 作为 JVM 的系统属性
* 在集成测试类上, 使用 @ActiveProfiles 注解设置

### 条件化的 bean

使用方法:

1. 用 @Conditional 注解标注带有 @Bean 的注解的方法上, 如果给定的条件为 true 则创建bean
2. @Conditional 参数可以是任意实现了 Condition 接口的类型.

例子:

```java
// 某个类中创建 bean 的方法
@Bean
@Conditional(MagicExistsCondition.class)
public MagicBean magicBean() {
  return new MagicBean();
}

// 实现 Condition 接口的类
public class MagicExistsCondition implements Condition {
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    Environment env = context.getEnvironment();
    return env.containsProperty("magic");
  }
}

// ConditionContext 是一个接口
public interface ConditionContext {
  BeanDefinitionRegistry get Registry();
  ConfigurableListableBeanFactory get BeanFactory();
  Environment getEnvironment();
  ResourceLoader getResourceLoader();
  ClassLoader getClassLoader();
}
// AnnotatedTypeMetadata 也是一个接口. 它能让我们检查带有@Bean的方法上还有哪些其他注解
```

### 处理装配歧义

当 Spring 试图自动装配时, 发现没有唯一, 无歧义的可选值, 它会抛出异常 NoUniqueBeanDefinitionException.

* 可以通过 @Primary 注解配合 @Component 标注一个首选的类, 也可以与@Bean 组合在Java 配置的 bean 声明中. @Primary 只能有一个.
* @Qualifier 注解是使用限定符的主要方法. 它可以和 @Autowired 和 @Inject 配合使用, 在注入时指定想要注入哪个 Bean. 它也用来定义一个限定符.
* 通过自定义注解来处理可能出现多处定义相同的@Qualifier的情况. 

```java
// 创建一个 @Cold 注解
@Target({ElementType.CONSTRUCTOR,ElementType.FIELD,ElementType.METHOD,ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Qualifier
public @interface Cold{}
```

### bean 的作用域

默认情况下, Spring 应用上下文中所有的bean都是单例的. 多次注入的都是同一实例.

有些时候, 你所使用的类是易变的, 它们会持有状态, 因此, 重用是不安全的.

Spring 通过作用域解决这个问题. 作用域包括:

* 单例: 在整个应用中, 只创建一个bean实例. 默认的.
* 原型: 每次注入或者通过Spring 上下文获取的时候, 都会创建一个新的 bean 实例.
* 会话: 在 Web 应用中, 为每个会话创建一个bean 实例.
* 请求: 在 Web 应用中, 为每个请求创建一个 bean 实例.

作用域通过 @Scope 注解实现.  ConfigurableBeanFactory.SCOPE_PROTOTYPE 定义了原型作用域. WebApplicationContext.SCOPE_SESSION 定义了会话作用域. 

@Scope 默认属性是 value, 指定作用域类型. 还有一个关键的 proxyMode 属性. 用于指定代理模式, 可以通过延迟加载解决session作用域在服务刚启动时不存在, 直到某个用户进入系统才会创建的问题.

```java
@Component
@Scope(
  value=WebApplicationContext.SCOPE_SESSION,
  proxyMode=ScopeProxyMode.INTERFACES)
public ShoppingCart cart() {}
```

### 运行时注入

* 属性占位符(Property placeholder)
* Spring表达式语言(SpEL)

#### 注入外部的值

* 最简单的方法就是声明属性源并通过Spring的Environment来检索属性.

```java
// 通过Spring 的 Environment 检索属性的例子
@Configuration
@PropertySource("classpath:/com/soundsystem/app.properties")
public class ExpressiveConfig {
  @Autowired
  Environment evn;
  @Bean
  public BlankDisc disc() {
    return new BlankDisc(
    	env.getPropterty("disc.title");
    );
  }
}
// 类路径中的 app.properties 中定义了属性
// disc.title=Sgt. Peppers Lonely Hearts Club Band
```

#### Spring Environment

它的 getProperty有4中重载

* String getProperty(String key)
* String getProperty(String key, String defaultvalue)
* T getProperty(String key, Class<T> type)
* T getProperty(String key, Class<T> type, T defaultvalue)

常见的方法如下:

* 如果属性为必需, 那么使用 getRequiredProperty 方法. 当属性不存在时, 将会抛出 IllegalStateException 异常.

* containsProperty(String key)方法用于检查属性是否存在.

* 如果想将属性解析为类的话, 可以使用 getPropertyAsClass(String key, Class<T> type) 方法.
* String[] getActiveProfiles() 返回激活的 profile 名称的数组
* String[[ getDefaultProfiles() 返回默认的 profile 名称的数组
* boolean acceptsProfiles(String... profiles) 判断环境是否支持给定的 profile

### 解析属性占位符

Spring 支持将属性定义在外部的属性文件中, 并使用占位符将其插入到 bean 中. 占位符形式为 "${...}". 这种方法用在 XML方式装配 bean 中.

如果依赖组件扫描和自动装配. 那么就没有指定占位符的配置文件和类了. 这种情况使用 @Value 注解.

```java
// 使用@Value 注解的例子
public BlankDisc(
	@Value("${disc.title}") String title
) {
  this.title=title;
}

// 为了使用占位符, 我们必须配置一个PropertySourcesPlaceholderConfigurer bean. 
// 如下的@Bean方法在Java 中配置了 PropertySourcesPlaceholderConfigurer
@Bean
public static PropertySourcesPlaceholderConfigurer placeholerConfigurer() {
  return new PropertySourcesPlaceholderConfigurer();
}
```

### 使用 SpEL 进行装配

SpEL 很强大, 可以完成其他装配技术难以做到的效果.

它的特性:

* 使用 bean ID 来引用 bean
* 调用方法和访问对象的属性
* 对值进行算术, 关系和逻辑运算
* 正则表达式匹配
* 集合操作

SpEL 表达式要放到 #{..} 之中.

```java
// SpEL 的简单例子
// 表达式为常量
#{1}
// 调用静态方法. 下面是调用 T()表达式求值的 java.lang.System 的类型的 currentTimeMillis() 方法
// 
#{T(System).currentTimeMillis()}
// 引用其他 bean或者 bean 的属性
#{sgtPeppers.artist}
// 引用系统属性
#{systemProperties['disc.title']}
// 数字相等用 ==
// 或者使用文本型的 eq 运算符
#{counter.total == 100}
#{counter.total eq 100}
// 三元表达式
#{scoreboard.score > 1000 ? "Winner" : "Loser"}
// 计算正则表达式
#{email matches '[abc]+@[def]+\\.com'}
 
```

## 面向切面的Spring

切面的常用术语有通知(advise), 切点(pointcut), 连接点(join point)

* 通知定义了切面是什么, 以及何时使用.
* 连接点定义了在应用执行过程中能够插入切面的一个点
* 切点定义了匹配通知所要织入的一个或者多个连接点. 指明"何处".
* 切面是通知和切点的结合. 描述切面是什么,何时何处完成其功能.
* 引入(Introduction) 允许我们向现有的类添加新方法或者属性.
* 织入(Weaving)是把切面应用到目标对象并创建新的代理对象的过程. 可以在目标对象的生命周期里进行织入: AspectJ的织入编译器可以完成编译期织入. 还可以在类加载期织入, 也可以在运行期织入.

SpringAOP 目前只支持方法级别的连接点. AspectJ 和 JBoss 支持字段和构造器的接入点. 可以通过 Aspect 来补充 Spring AOP 功能.

## Spring MVC

### 最小可用的 Spring MVC 的配置

```java
@Configuration
@EnableWebMvc
@ComponentScan
public class WebConfig extends WebMvcConfigurerAdapter {
  // 配置 JSP 视图解析器
  @Bean
  public ViewResolver viewResolver() {
    
  }
  // 配置静态资源的处理
  @Override
  public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    
  }
}
```

### 最简单的 Spring MVC Controller

```java
@Controller
public class HomeController {
  // 处理对 "/" 的 GET 请求
  // 路径映射到方法上.
	@RequestMapping(value="/", method=GET)  
  public String home() {
    return "home";	// 返回视图名称. DispatcherServlet 会要求视图解析器将这个名称解析为实际的视图.
  }
}
```

路径还可以映射到类上

```java
@Controller

// RequestMapping 可以映射一个 String 类型的数组.
@RequestMapping({"/", "/homepage"})
public class HomeController {
  // 现在 "/", "/homepage" 的 GET 请求都映射到 home() 方法上
  // 此处的 RequestMapping 注解是对类级别 RequestMapping 的补充
  @RequestMapping(method=GET)
  public String home() {
    return "home";
  }
}
```

### 传递 Model 数据到视图中

```java
// 一般定义模型访问接口.
public interface SpittleReposigory {
  // Spittle 是一个 POJO 
  List<Spittle> findSpittles(long max, int count);
}

// SpittleController
@Controller
@RequestMapping("/spittles")
public class SpittleController {
  private SpittleRepository spittleRepository;
  // 注入 SpittleRepository 实例
  @Autowired
  public SpittleController(SpittleRepository spittleRepository) {
    this.spittleRepository = spittleRepository;
  }
  @RequestMapping(method=RequestMethod.GET)
  public String spittles(Model model) {
    // 将返回数据添加到模型中
    // Model 对象实际上是一个 Map(key-value的集合)
    // 下面调用中没有指定 key, key值会根据对象类型推导确定. 本例子中findSpittles返回的类型是 List<Spittle>, 所以 key 为: spittleList
    // 等同于 model.addAttribute("spittleList", spittleRepository.findSpittles(Long.MAX_VALUE, 20));
    model.addAttribute(
    	spittleRepository.findSpittles(Long.MAX_VALUE, 20)
    );
    return "spittles"; // 返回视图名称
  }
  // 上面的方法返回的是视图名称
  // 还可以有替代方法, 比如
  @RequestMapping(method=RequestMethod.GET)
  public List<Spittles> spittles() {
    // 这个值会放到模型中, 模型的 key 会根据其类型推断得出. 此处返回的 key 是 "spittleList"
    // 而视图名称会根据请求路径推断得出. 因为这个方法处理 "/spittles" 的 GET 请求, 因此视图的名称是 "spittles"
    return spttileRepository.findSpittles(Long.MAX_VALUE, 20));  
  }
}


```

### Mock 测试

```java
// 针对 "/spittles" 的 test
@Test
public void shouldShowRecentSpittles() throws Exception {
  List<Spittle> expectedSpittles = createSpittleList(20);
  SpittleRepository mockRepository = mock(SpittleRepository.class);
  when(mockRepository.findSpittles(Long.MAX_VALUES, 20)).thenReturn(expectedSpittles);
  SpittleController controller = new SpittleController(mockRepository);
  MockMvc mockMvc = standaloneSetup(controller).setSingleView(
    new InternalResourceView("/WEB-INF/views/spittles.jsp")).build();
  mockMvc.perform(get("/spittles"))
    .andExpect(view().name("spittles"))
    .andExpect(model().attributeExists("spittleList"))
    .andExpect(model().attribute("spittleList", hasItems(expectedSpittles.toArray())));
}
// ...
private List<Spittle> createSpittleList(int count) {
  List<Spittle> spittles = new ArrayList<Spittle>();
  IntStream.range(0, count).forEach(i -> new Spittle("Spittle"+i, new Date()));
  return spittles;
}
```

### 接受请求的输入

将客户端数据传到控制器的方法:

* 查询参数

* 路径变量

* 表单参数

  

#### 处理查询参数

```java
// SpittleController 中的新方法
// RequestParam 注解可以标注请求中的参数的名称和默认值. @RequestParam(value="max", defaultValue=MAX_LONG_AS_STRING)
// 虽然 defaultValue 指定的是 String 类型的值, 但是当绑定到方法的 max 参数时, 它被转换为 Long 类型.
@RequestMapping(method=RequestMethod.GET)
public List<Spittle> spittles(
  @RequestParam("max") long max,
  @RequestParam("count") int count) {
  return spittleRepository.findSpittles(max, count);
}

// ...
// 用来测试分页 Spittle 列表的新方法
@Test
public void shoudShowPagedSpittles() throws Exception {
  List<Spittle> expectedSpittles = createSpittleList(50);
  SpittleRepository mockRepository = mock(SpittleRepository.class);
  when(mockRepository.findSpittles(238900, 50))
    .thenReturn(expectedSpittles);
  MockMvc mockMvc = standaloneSetup(controller)
    .setSingleView(new InternalResourceView("/WEB-INF/views/spittles.jsp")).build();
  mockMvc.perform(get("/spittles?max=238900&count=50"))
    .andExpect(view().name("spittles"))
    .andExpect(model().attributeExists("spittleList"))
    .andExpect(model().attribute("spittleList", hasItems(expectedSpittles.toArray())));
}
```

#### 处理路径变量

"/spittles/show?spittle_id=123" 这样的请求本质上是通过 HTTP 发起的 RPC 请求. 从面向资源的角度来看这不是很理想. "/spittles/123" 是以 RESTful API 方式表达, 能够很好的识别出要查询的资源. 下面的处理器方法使用了占位符, 将 spittle_id 作为路径的一部分:

```java
// 处理 "/spittles/123" 的 GET 请求
@RequestMapping(value="/{spittleId}", method=RequestMethod.GET)
public String spittle(
  // 如果 @PathVariable 缺失 value 属性, 则它会假设占位符的名称与方法的参数名称相同.
  // 就如下面, 可以省略掉 ("spittleId")
  @PathVariable("spittleId") long spittleId,
  Model model
) {
  model.addAttribute(spittleRepository.findOne(spittleId));
  return "spittle";
} 
```

#### 处理表单

```java

@Controller
@RequestMapping("/spitter")
public class SpitterController {
  // 下面展示处理 "/spitter/register" 的 GET请求
  @RequestMapping(value="/register", method=GET)
  public String showRegistrationForm() {
    return "registerForm"; // 按照配置的的InternalResourceViewResolver 的方式, 会使用 "/WEB-INF/views/registerForm.jsp" 来渲染注册表单.
  }
  // 处理 "/spitter/register" 的 POST 请求
  @RequestMapping(value="/register", method=POST)
  public String processRegistration(Spitter spitter) {
    spitterRepository.save(spitter);
    return "redirect:/spitter/"+spitter.getUsername(); // 重定向基本信息界面
  }
  
}

```

```java
// 测试处理表单的控制器方法
@Test
public void shouldProcessRegistration() throws Exception {
  SpitterRepository mockRepository = mock(SpitterRepository.class);
  Spitter unsaved = new Spitter("jbauer", "24hours", "Jack", "Bauer");
  Spitter saved = new Spitter(24L, "jbauer", "24hours", "Jack", "Bauer");
  when(mockRepository.save(unsaved)).thenReturn(saved);
  SpitterController controller = new SpitterController(mockRepository);
  MockMvc mockMvc = standaloneSetup(controller).build();
  mockMvc.perform(post("/spitter/register")
                 .param("firstName", "Jack")
                 .param("lastName", "Bauer")
                 .param("username", "jbauer")
                 .param("password", "24hours")
                 ).andExpect(redirectedUrl("/spitter/jbauer")); // 重定向可以避免浏览器刷新重复提交表单.
  verify(mockRepository, atLeastOnce()).save(unsaved); // 验证保存情况
}
```

#### 校验表单

通过 Java 校验 API 来校验表单. Java 校验 API 定义了多个注解, 这些注解可以放到属性上, 从而校验这些属性的值.

```java
// 校验注解的的例子
public class Spitter {
  private Long id;
  @NotNull
  @Size(min=5, max=16)
  private String username;
  @NotNull
  @Size(min=6, max=30)
  private String password;
}
```

| 注解                       | 含义                                                         |
| -------------------------- | ------------------------------------------------------------ |
| @AssertFalse               | 被注解的元素是 Boolean 类型,并且为 false                     |
| @AssertTrue                | 被注解的元素是 Boolean 类型,并且为 true                      |
| @DecimalMax                | 被注解的元素是 数字 类型,并且它的值**最大**为给定的 BigDecimalString 的值(可以等于) |
| @DecimalMin                | 被注解的元素是 数字 类型,并且它的值**最小**为给定的 BigDecimalString 的值(可以等于) |
| @Null                      | 被注解的元素是任意类型, 它的值必须为 null                    |
| @NotNull                   | 被注解的元素是任意类型, 它的值必须不为 null                  |
| @NotBlank                  | 被注解的元素是 CharSequence 的子类型(CharBuffer, String, StringBuffer, StringBuilder), 值不为 null, 去除首尾空格后长度不为 0 |
| @NotEmpty                  | CharSequence 子类型, Collection, Map, Array. 值不为 null, 字符串长度不为0 ,集合长度不为 0 |
| @Size(min,max)             | 字符串,Collection, Map, 数组. 字符串的长度或集合大小在 [min, max] 之间 |
| @Length(min,max)           | CharSequence 子类型. 长度在 [min, max]                       |
| @Past                      | Date, Calendar 等日期类型. 值要比当前时间**早**              |
| @Future                    | Date, Calendar 等日期类型. 值要比当前时间**晚**              |
| @PastOrPresent             | Date, Calendar 等日期类型. 比当前早或者当前时间.             |
| @FutureOrPresent           | Date, Calendar 等日期类型. 比当前晚或者当前时间.             |
| @Min(value)                | BigDecimal, BigInteger, byte, short, int, long 等. 值 >= value |
| @Max(value)                | BigDecimal, BigInteger, byte, short, int, long 等. 值 <= value |
| @Email(regexp,flag)        | Email字符串. 也可以通过 regexp 和 flag 指定 email 格式       |
| @Pattern(regexp,flag)      | 字符串类型. 值要能被正则表达式匹配.                          |
| @ScriptAssert(lang,script) | 业务类. 校验复杂的业务逻辑.                                  |

## Mybatis Plus

mybatis plus 是 mybatis 的增强实现.

按官方的"快速入门"中的例子, 我遇到的问题:

1. @Autowired 注解 UserMapper 接口变量 , 会报创建 Bean 失败. 使用 @Resource 注解可以.
2. 插入操作中, 不设置 User 的 id(自增的), 会报错. user.setId(0) 即可以解决.

## SpringBoot & Junit5



  

  

