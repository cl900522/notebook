Spring Bean
=================
# 生命周期
>Spring作为当前Java最流行. 最强大的轻量级框架，受到了程序员的热烈欢迎。准确的了解Spring Bean的生命周期是非常必要的。我们通常使用ApplicationContext作为Spring容器。这里，我们讲的也是 ApplicationContext中Bean的生命周期。而实际上BeanFactory也是差不多的，只不过处理器需要手动注册。

## 生命周期流程图
>若容器注册了以上各种接口，程序那么将会按照以上的流程进行

![](/images/2017/09/181453414212066.png)


## 各种接口方法分类
Bean的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类：

1. Bean自身的方法
    这个包括了Bean本身调用的方法和通过配置文件中<bean>的init-method和destroy-method指定的方法

2. Bean级生命周期接口方法
    这个包括了BeanNameAware, BeanFactoryAware,InitializingBean和DiposableBean这些接口的方法

3. 容器级生命周期接口方法
    这个包括了InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现，一般称它们的实现类为“后处理器”。

4. 工厂后处理器接口方法
    这个包括了AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer等等非常有用的工厂后处理器接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。

# 作用域
>当在 Spring 中定义一个 时，你必须声明该 bean 的作用域的选项。例如，为了强制 Spring 在每次需要时都产生一个新的 bean 实例，你应该声明 bean 的作用域的属性为 prototype。同理，如果你想让 Spring 在每次需要时都返回同一个bean实例，你应该声明 bean 的作用域的属性为 singleton。

Spring 框架支持以下五个作用域，如果你使用 web-aware ApplicationContext 时，其中三个是可用的。

作用域  |  描述
--|--
singleton	  |  该作用域将 bean 的定义的限制在每一个 Spring IoC 容器中的一个单一实例(默认)。
prototype  |  该作用域将单一 bean 的定义限制在任意数量的对象实例。
request  |  该作用域将 bean 的定义限制为 HTTP 请求。只在 web-aware Spring ApplicationContext 的上下文中有效。
session  |  该作用域将 bean 的定义限制为 HTTP 会话。 只在web-aware Spring ApplicationContext的上下文中有效。
global-session  |  该作用域将 bean 的定义限制为全局 HTTP 会话。只在 web-aware Spring ApplicationContext 的上下文中有效。
