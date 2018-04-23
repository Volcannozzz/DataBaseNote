#### Beans工程

##### 对org.springframework.beans.factory.support包内的接口分析

**1. AutowireCandidateResolver**

策略接口，用来决定一个特定的bean definition能否作为特定依赖的候选bean。

**2. BeanDefinitionReader**

bean definition的读取器。AbstractBeanDefinitionReader是其直接实现类。AbstractBeanDefinitionReader内部含有一个registry，用于注册。

**3. BeanDefinitionRegistry**

注册器，继承了AliasRegistry接口，包含对指定的bean definition以特定的名字进行注册的功能。

**4. BeanDefinitionRegistryPostProcessor**

对SPI  BeanFactoryPostProcessor的一个扩展，在实例化bean之前进行一些额外操作（例如注册更多的bean definition）等。

**5. BeanNameGenerator**

bean名称的组织接口。Generate a bean name for the given bean definition.

**6. InstantiationStrategy**

负责创建与根bean定义相对应的实例的接口。

**7. MergedBeanDefinitionPostProcessor**

对被合并的bean定义的后处理接口。

**8. MethodReplacer**

实现该接口的类，可以以方法注入的形式，对IoC容器管理的对象的方法进行replace.

**9. SecurityContextProvider**

在bean工厂内运行的代码的安全上下文的提供者。

