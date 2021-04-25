BeanDefinition只是一个最小的接口：主要目的是==允许 `BeanFactoryPostProcessor` 内省和修改属性值和其他bean的元数据==。

BeanDefinition抽象了Bean的定义，保存了Bean的必要信息，如在xml配置中，BeanDefinition就保存了与`<bean>`相关的id，name，aliases等属性，封装了很多与Bean相关的基本数据，是容器实现依赖反转功能的核心数据结构。

```java
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
  String SCOPE_SINGLETON = ConfigurableBeanFactory.SCOPE_SINGLETON;
  String SCOPE_PROTOTYPE = ConfigurableBeanFactory.SCOPE_PROTOTYPE;
  int ROLE_APPLICATION = 0;
  int ROLE_SUPPORT = 1;
  int ROLE_INFRASTRUCTURE = 2;
  String getParentName();
  void setParentName(String parentName);
  String getBeanClassName();
  // 类名
  void setBeanClassName(String beanClassName);
  String getFactoryBeanName();
  void setFactoryBeanName(String factoryBeanName);
  String getFactoryMethodName();
  void setFactoryMethodName(String factoryMethodName);
  String getScope();
  // scope
  void setScope(String scope);
  // 是否是懒加载
  boolean isLazyInit();
  void setLazyInit(boolean lazyInit);
  String[] getDependsOn();
  // 依赖的bean
  void setDependsOn(String... dependsOn);
  boolean isAutowireCandidate();
  void setAutowireCandidate(boolean autowireCandidate);
  boolean isPrimary();
  void setPrimary(boolean primary);
  // 构造函数参数列表
  ConstructorArgumentValues getConstructorArgumentValues();
  MutablePropertyValues getPropertyValues();
  // 是否是单例
  boolean isSingleton();
  boolean isPrototype();
  boolean isAbstract();
  int getRole();
  String getDescription();
  String getResourceDescription();
  BeanDefinition getOriginatingBeanDefinition();
}
```

>- BeanDefinition接口继承了AttributeAccessor接口，说明它拥有处理属性的能力；
>- BeanDefinition接口继承了BeanMetadataElement接口，说明它拥有bean元素的属性，即可以持有XML文件的一个bean标签对应的Object。
>
>BeanDefinition常用的实现类有：
>
>- `ChildBeanDefinition`
>- `RootBeanDefinition`
>- `GenericBeanDefinition`

