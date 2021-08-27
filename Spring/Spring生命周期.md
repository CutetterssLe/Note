- [1、IOC容器加载过程](#1ioc容器加载过程)
  - [1.1、this()构造函数](#11this构造函数)
  - [1.2、register（annotatedClasses）：注册成bean定义，BeanDefinitionMap](#12registerannotatedclasses注册成bean定义beandefinitionmap)
  - [1.3、refresh（）：刷新IOC容器接口，也就是SpringBean的生命周期](#13refresh刷新ioc容器接口也就是springbean的生命周期)
- [2、SpringBean的生命周期](#2springbean的生命周期)
- [2.1 准备刷新上下文环境](#21-准备刷新上下文环境)
- [2.2 获取告诉子类初始化bean工厂 不同工厂不同实现](#22-获取告诉子类初始化bean工厂-不同工厂不同实现)
- [2.3 对bean工厂进行属性填充](#23-对bean工厂进行属性填充)
- [2.4 留个子类实现postProcessBeanFactory](#24-留个子类实现postprocessbeanfactory)
- [2.5 调用bean工厂的后置处理器](#25-调用bean工厂的后置处理器)
- [2.6 注册我们自定义的bean后置处理器](#26-注册我们自定义的bean后置处理器)
- [2.7 初始化国际资源处理器](#27-初始化国际资源处理器)
- [2.8 创建事件多播器](#28-创建事件多播器)
- [2.9](#29)
# 1、IOC容器加载过程
## 1.1、this()构造函数
读取Bean工厂，在父类GenericApplicationContext，beanFactory = new DefaultListableBeanFactory，这个类实现了很多工厂，拥有了很多父工厂的能力。AnnotationConfigApplicationContext中创建了BeanDefinitionReader，创建了很多内置bean定义，用来解析实例化过程中的外置bean，扫描bean：ClassPathBeanDefinitionScanner
- 注册BeanFactory
- 注册BeanDefinitionReader
- 注册BeanDefinitionScanner，可以手动context.scan（）进行扫描
## 1.2、register（annotatedClasses）：注册成bean定义，BeanDefinitionMap
## 1.3、refresh（）：刷新IOC容器接口，也就是SpringBean的生命周期
# 2、SpringBean的生命周期
# 2.1 准备刷新上下文环境
# 2.2 获取告诉子类初始化bean工厂 不同工厂不同实现
# 2.3 对bean工厂进行属性填充
# 2.4 留个子类实现postProcessBeanFactory
# 2.5 调用bean工厂的后置处理器
# 2.6 注册我们自定义的bean后置处理器
# 2.7 初始化国际资源处理器
# 2.8 创建事件多播器
# 2.9 
