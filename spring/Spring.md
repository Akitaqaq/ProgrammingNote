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



