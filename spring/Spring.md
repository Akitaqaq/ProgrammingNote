### Spring

#### ConfigurableApplicationContext 类图

![image-20220809153521420](https://gitee.com/amiaosixsix/md-img/raw/master/%20tyImg/image-20220809153521420.png)

- `MessageSource`国际化支持
- `ResourcePatternResolver` 资源通配符
- `ApplicationEventPublisher` 事件发布器
- `EnvironmentCapable` 系统运行环境相关

`BeanFactory`的默认实现`DefaultListableBeanFactory`

![image-20220809205859038](https://gitee.com/amiaosixsix/md-img/raw/master/%20tyImg/image-20220809205859038.png)

````java
   DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
        AbstractBeanDefinition beanDefinition = BeanDefinitionBuilder.genericBeanDefinition(Config.class).setScope("singleton")
                .getBeanDefinition();
        beanFactory.registerBeanDefinition("config", beanDefinition);
        AnnotationConfigUtils.registerAnnotationConfigProcessors(beanFactory);
        //给beanFactory加上一些后处理器，补充一些bean定义
 beanFactory.getBeansOfType(BeanFactoryPostProcessor.class).values().forEach(beanFactoryPostProcessor -> {
            beanFactoryPostProcessor.postProcessBeanFactory(beanFactory);
        });
        //bean 后处理器，针对bean的生命周期各个阶段提供扩展，例如@autowired @resource
beanFactory.getBeansOfType(BeanPostProcessor.class).values().forEach(beanFactory::addBeanPostProcessor);
        String[] beanDefinitionNames = beanFactory.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            System.out.println(beanDefinitionName);
        }
        beanFactory.preInstantiateSingletons();
        System.out.println("==============================");
        System.out.println(beanFactory.getBean(bean1.class).getBean2());
````



`DefaultListableBeanFactory`特点：

1. beanFactory不会做那些事：
   1. 不会主动调用BeanFactory后置处理器
   2. 不会主动添加Bean后处理器
   3. 不会主动初始化单例
   4. 不会解析BeanFactory,不会解析${} #{}
2. bean后处理器会有排序的逻辑

#### Bean的生命周期

六个扩展点。。。待写

##### bean后置处理器作用

1. `AutowiredAnnotationBeanPostProcessor`:解析`  @Autowired`、`@Value`依赖注入注解，在依赖注入阶段执行。

   运行分析：

   ````java
     DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
           beanFactory.registerSingleton("bean2", new Bean2());
           beanFactory.registerSingleton("bean3", new Bean3());
           beanFactory.setAutowireCandidateResolver(new ContextAnnotationAutowireCandidateResolver());//@value
           beanFactory.addEmbeddedValueResolver(new StandardEnvironment()::resolvePlaceholders);//${}解析器
           AutowiredAnnotationBeanPostProcessor processor = new AutowiredAnnotationBeanPostProcessor();
           processor.setBeanFactory(beanFactory);
           Bean1 bean1 = new Bean1();
           System.out.println(bean1);
   //        processor.postProcessProperties(null, bean1, "bean1");
           // 1.processor.postProcessProperties：如何进行依赖注入
           // 通过方法名和参数找到对应的方法
           Method findAutowiringMetadata = processor.getClass().getDeclaredMethod("findAutowiringMetadata", String.class, Class.class, PropertyValues.class);
           findAutowiringMetadata.setAccessible(true);
           //传入对象示例和参数调用方法
           InjectionMetadata injectionMetadata = (InjectionMetadata) findAutowiringMetadata.invoke(processor, "bean1", bean1.getClass(), null);
           System.out.println(injectionMetadata);
           //2、调用injectionMetadata进行依赖注入，注入时按类型查找值
           injectionMetadata.inject(bean1, "bean1", null);
           System.out.println(bean1);
           //3、injectionMetadata.inject:如何按类型查找值进行注入
           //3.1 字段注入
           Field bean3 = bean1.getClass().getDeclaredField("bean3");
           DependencyDescriptor dependencyDescriptor = new DependencyDescriptor(bean3, false);
           Object o = beanFactory.doResolveDependency(dependencyDescriptor, null, null, null);
           System.out.println(o);
           //3.2 方法注入
           Method setBean2 = bean1.getClass().getDeclaredMethod("setBean2", Bean2.class);
           DependencyDescriptor dependencyDescriptor1 = new DependencyDescriptor(new MethodParameter(setBean2, 0), false);
           Object o1 = beanFactory.doResolveDependency(dependencyDescriptor1, null, null, null);
           System.out.println(o1);
           //3.3 @value注入
           Method setHome = bean1.getClass().getDeclaredMethod("setHome", String.class);
           DependencyDescriptor dependencyDescriptor2 = new DependencyDescriptor(new MethodParameter(setHome, 0), false);
           Object o2 = beanFactory.doResolveDependency(dependencyDescriptor2, null, null, null);
           System.out.println(o2);
   ````

   

   

2. `CommonAnnotationBeanPostProcessor`:解析`@Resource`、`@PostConstruct`、`@PreDestroy` 注解

3. `ConfigurationPropertiesBindingPostProcessor`:解析 `@ConfigurationProperties`注解

#### Bean工厂的后置处理器

1. `ConfigurationClassPostProcessor`:解析`@ComponentScans`、`@Bean`、`@Import`、`@ImportResource`注解
2. `MapperScannerConfigurer`:解析MyBatis mapper接口

````java
    GenericApplicationContext genericApplicationContext = new GenericApplicationContext();
        genericApplicationContext.registerBean("config", Config.class);
        genericApplicationContext.registerBean(ConfigurationClassPostProcessor.class);
        genericApplicationContext.registerBean(MapperScannerConfigurer.class, bd -> {
            bd.getPropertyValues().add("basePackage", "com.shsnc.kbs.a05.mapper");
        });
        genericApplicationContext.refresh();

        for (String beanDefinitionName : genericApplicationContext.getBeanDefinitionNames()) {
            System.out.println(beanDefinitionName);
        }

        genericApplicationContext.close();
````

#### Aware接口及InitializingBean

1. Aware用于注入与容器相关的信息

   1. `BeanNameAware` 注入bean的名字
   2. `BeanFactoryAware` 注入BeanFactory容器
   3. `ApplicationContextAware `注入 ApplicationContextAware 容器
   4. `EmbeddedValueResolverAware` ${}

   ![image-20220814095830202](https://gitee.com/amiaosixsix/md-img/raw/master/%20tyImg/image-20220814095830202.png)

   其中b、c、d可以使用`@Autowire` 实现。aware属于内置功能，`@Autowir`属于扩展功能，某些情况下会失效，而内置功能不会

   失效举例：

   - 例1：比如没有把解析`@Autowired`和`@PostStruct`注解的`Bean`的后处理器加到`Bean`工厂中，你会发现用 `Aware` 注入 `ApplicationContext` 成功， 而 `@Autowired` 注入 `ApplicationContext` 失败

   - 例2：定义两个Java Config类（类上加@Configuration注解），名字分别叫MyConfig1和MyConfig2，都实现注入ApplicationContext容器和初始化功能，MyConfig1用@Autowired和@PostConstruct注解实现，MyConfig2用实现Aware和InitializingBean接口的方式实现，另外，两个Config类中都通过@Bean注解的方式注入一个BeanFactoryPostProcessor，代码如下：
     ````java
     @Slf4j
     public class MyConfig1 {
         @Autowired
         public void setApplicationContext(ApplicationContext applicationContext) {
             log.debug("注入 ApplicationContext");
         }
     
         @PostConstruct
         public void init() {
             log.debug("初始化");
         }
     
         @Bean
         public BeanFactoryPostProcessor processor1() {
             return beanFactory -> {
                 log.debug("执行 processor1");
             };
         }
     }
     
     ````

   测试代码：

   ````java
   @Slf4j
   public class TestAwareAndInitializingBean {
       @Test
       public void testAware_MyConfig1() {
           GenericApplicationContext context = new GenericApplicationContext();
           // MyConfig1没有加上@
           context.registerBean("myConfig1", MyConfig1.class);
           // 解析 @Autowired 注解的Bean后处理器
           context.registerBean(AutowiredAnnotationBeanPostProcessor.class);
           // 解析 @PostConstruct 注解的Bean后处理器
           context.registerBean(CommonAnnotationBeanPostProcessor.class);
           // 解析@ComponentScan、@Bean、@Import、@ImportResource注解的后处理器
           // 这个后处理器不加出不来效果
           context.registerBean(ConfigurationClassPostProcessor.class);
   
           // 1. 添加beanfactory后处理器；2. 添加bean后处理器；3. 初始化单例。
           context.refresh();
   
           context.close();
   }
   ````

   运行结果：

   ![image-20220331090537999](https://img-blog.csdnimg.cn/img_convert/ba4a53092e03eceef2e6172ba845b791.png)

Java配置类在添加了 bean 工厂后处理器后，你会发现用传统接口方式的注入和初始化依然成功，而 @Autowired 和 @PostConstruct 的注入和初始化失败。

那是什么原因导致的呢？

配置类 @Autowired 注解失效分析

- Java 配置类不包含 BeanFactoryPostProcessor 的情况

- ````mermaid
  sequenceDiagram 
  participant ac as ApplicationContext
  participant bfpp as BeanFactoryPostProcessor
  participant bpp as BeanPostProcessor
  participant config as Java配置类
  ac ->> bfpp : 1. 执行 BeanFactoryPostProcessor
  ac ->> bpp : 2. 注册 BeanPostProcessor
  ac ->> +config : 3. 创建和初始化
  bpp ->> config : 3.1 依赖注入扩展(如 @Value 和 @Autowired)
  bpp ->> config : 3.2 初始化扩展(如 @PostConstruct)
  ac ->> config : 3.3 执行 Aware 及 InitializingBean
  config -->> -ac : 3.4 创建成功
  ````

- 
