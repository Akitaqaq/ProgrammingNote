 # Spring

## 一、IOC容器

### 基于注解的容器入口

<img src="C:\Users\cm\Desktop\ProgrammingNote\spring\Spring.assets\image-20230505220344579.png" alt="image-20230505220344579" style="zoom:150%;" />

spring上下文对象，基于注解方式启动：`ApplicationContext content=new  AnnotationConfigApplicationContext(SpringConfig.class);`方法属性如下图：

![image-20230505222212690](C:\Users\cm\Desktop\ProgrammingNote\spring\Spring.assets\image-20230505222212690.png)

````java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
		this();
		register(componentClasses);
		refresh();
	}
````

**this**:调用自身无参构造方法，初始化reader和scanner，reader内部初始化了4个处理器并保存在：`BeanDefinitionRegistry`：

![image-20230505223655635](C:\Users\cm\Desktop\ProgrammingNote\spring\Spring.assets\image-20230505223655635.png)

scanner初始化了注解过滤器：

![image-20230505225100739](C:\Users\cm\Desktop\ProgrammingNote\spring\Spring.assets\image-20230505225100739.png)

**register**：

1.首先需要把传入的组件类转为`beanDefinition`描述

2.`beanDefinition`包装到`BeanDefinitionHolder`中

3.`BeanDefinitionHolder`调用方法，放入到`DefaultListableBeanFactory`的成员变量`Map<String, BeanDefinition> beanDefinitionMap `缓存中

**refresh**：

