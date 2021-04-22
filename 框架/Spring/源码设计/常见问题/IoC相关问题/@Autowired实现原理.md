>1. 构造函数注入
>
>```java
>public Class Outer {
>    private Inner inner;
>    @Autowired
>    public Outer(Inner inner) {
>        this.inner = inner;
>    }
>}
>```
>
>2. 属性注入
>
>```java
>public Class Outer {
>    @Autowired
>    private Inner inner;
>}
>```
>
>3. 方法注入
>
>```java
>public Class Outer {
>    private Inner inner;
>    public Inner getInner() {
>        return inner;
>    }
>    @Autowired
>    public void setInner(Inner inner) {
>        this.inner = inner;
>    }
>}
>```
>
>第1种在bean实例化时完成；而第2、3种的实现都是在属性填充时完成。

#### AutowiredAnnotationBeanPostProcessor类

>该类是@Autowired的具体实现类。
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gn7ro89cqjj310q02y0sr.jpg" style="zoom:60%">
>
>```java
>@Override
>public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
>    // 寻找bean中所有被@Autowired注解的属性，将其封装成InjectedElement类型
>    InjectionMetadata metadata = findAutowiringMetadata(beanName, beanType, null);
>    metadata.checkConfigMembers(beanDefinition);
>}
>
>@Override
>public PropertyValues postProcessPropertyValues(
>    PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeanCreationException {
>
>    // 寻找通过@Autowired注解的属性或方法
>    InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
>    try {
>        // 注入
>        metadata.inject(bean, beanName, pvs);
>    }
>    catch (BeanCreationException ex) {
>        throw ex;
>    }
>    catch (Throwable ex) {
>        throw new BeanCreationException(beanName, "Injection of autowired dependencies failed", ex);
>    }
>    return pvs;
>}
>```
>
>首先找出所有注解了`@Autowired`的属性或者方法，然后进行注入，当然`postProcessMergedBeanDefinition`后置处理器的调用肯定是在`postProcessPropertyValues`之前的，这里回顾一下spring bean的创建过程：
>
><img src="https://tva1.sinaimg.cn/large/008eGmZEgy1gn7rvmomwdj30tw1r00vt.jpg" style="zoom:60%">

#### 1. 查找所有@Autowired(postProcessMergedBeanDefinition)

>```java
>private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, @Nullable PropertyValues pvs) {
>    // 获取缓存的key值，一般以beanName做key
>    String cacheKey = (StringUtils.hasLength(beanName) ? beanName : clazz.getName());
>    // 从缓存中获取metadata
>    InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
>    // 检测metadata是否需要更新
>    if (InjectionMetadata.needsRefresh(metadata, clazz)) {
>        synchronized (this.injectionMetadataCache) {
>            metadata = this.injectionMetadataCache.get(cacheKey);
>            if (InjectionMetadata.needsRefresh(metadata, clazz)) {
>                if (metadata != null) {
>                    metadata.clear(pvs);
>                }
>                // 通过clazz类，查找所有@Autowired的属性或者方法，并封装成InjectionMetadata类型
>                metadata = buildAutowiringMetadata(clazz);
>                // 将metadata加入缓存
>                this.injectionMetadataCache.put(cacheKey, metadata);
>            }
>        }
>    }
>    return metadata;
>}
>```
>
>可以看到spring依然在用缓存的方式提高性能，继续跟踪核心代码`buildAutowiringMetadata(clazz)`
>
>```java
>private InjectionMetadata buildAutowiringMetadata(final Class<?> clazz) {
>    // 查看clazz是否有Autowired注解
>    if (!AnnotationUtils.isCandidateClass(clazz, this.autowiredAnnotationTypes)) {
>        return InjectionMetadata.EMPTY;
>    }
>    // 这里需要注意AutowiredFieldElement，AutowiredMethodElement均继承了InjectionMetadata.InjectedElement
>    // 因此这个列表是可以保存注解的属性和被注解的方法的
>    List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
>    Class<?> targetClass = clazz;
>
>    // 1. 通过do while循环，递归的往直接继承的父类寻找@Autowired
>    do {
>        final List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();
>
>        // 2. 通过反射，获取所有属性，doWithLocalFields则是循环的对每个属性应用以下匿名方法
>        ReflectionUtils.doWithLocalFields(targetClass, field -> {
>            // 判断当前field属性是否含有@Autowired的注解
>            MergedAnnotation<?> ann = findAutowiredAnnotation(field);
>            if (ann != null) {
>                // 返回该属性在类中的修饰符，如果等于static常量，则抛出异常，@Autowired不允许注解在静态属性上
>                if (Modifier.isStatic(field.getModifiers())) {
>                    if (logger.isInfoEnabled()) {
>                        logger.info("Autowired annotation is not supported on static fields: " + field);
>                    }
>                    return;
>                }
>                // @Autowired有required属性，获取required的值，默认为true
>                boolean required = determineRequiredStatus(ann);
>                // 3. 将field封装成InjectedElement，并添加到集合中，这里用的是AutowiredFieldElement
>                currElements.add(new AutowiredFieldElement(field, required));
>            }
>        });
>
>        // 4. @Autowired可以注解在方法上
>        ReflectionUtils.doWithLocalMethods(targetClass, method -> {
>            Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
>            if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
>                return;
>            }
>            MergedAnnotation<?> ann = findAutowiredAnnotation(bridgedMethod);
>            if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
>                if (Modifier.isStatic(method.getModifiers())) {
>                    if (logger.isInfoEnabled()) {
>                        logger.info("Autowired annotation is not supported on static methods: " + method);
>                    }
>                    return;
>                }
>                if (method.getParameterCount() == 0) {
>                    if (logger.isInfoEnabled()) {
>                        logger.info("Autowired annotation should only be used on methods with parameters: " +
>                                    method);
>                    }
>                }
>                boolean required = determineRequiredStatus(ann);
>                PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
>                // 5. 将方法封装成InjectedElement，并添加到集合中，这里用的是AutowiredMethodElement
>                currElements.add(new AutowiredMethodElement(method, required, pd));
>            }
>        });
>
>        elements.addAll(0, currElements);
>        // 返回直接继承的父类
>        targetClass = targetClass.getSuperclass();
>    }
>    // 如果父类不为空则需要把父类的@Autowired属性或方法也找出
>    while (targetClass != null && targetClass != Object.class);
>    // 6. new InjectionMetadata(clazz, elements)，将找到的所有的待注入属性或方法生成metadata返回
>    return InjectionMetadata.forElements(elements, clazz);
>}
>```
>
>1. 外层 do … while … 的循环被用于递归的查找父类的`@Autowired`属性或方法
>2. 通过反射的方式获取到所有属性并循环验证每一个属性是否被`@Autowired注解`
>3. 将查找到包含@Autowired注解的`filed`封装成`AutowiredFieldElement`，加入到列表中
>4. 循环查找在方法上的注解
>5. 将找到的方法封装成`AutowiredMethodElement`，并加入列表
>   这里需要特别强调一点，InjectedElement被`AutowiredFieldElement`、`AutowiredMethodElement`所继承，他们都有各自的`inject`函数，实现各自的注入。因此改ArrayList elements是拥有2种类型的属性。
>6. 将找到的所有元素列表和clazz作为参数生成metadata数据返回。

#### 2. 注入(postProcessPropertyValues)

>```java
>public void inject(Object target, @Nullable String beanName, @Nullable PropertyValues pvs) throws Throwable {
>    Collection<InjectedElement> checkedElements = this.checkedElements;
>    Collection<InjectedElement> elementsToIterate =
>        (checkedElements != null ? checkedElements : this.injectedElements);
>    if (!elementsToIterate.isEmpty()) {
>        for (InjectedElement element : elementsToIterate) {
>            if (logger.isDebugEnabled()) {
>                logger.debug("Processing injected element of bean '" + beanName + "': " + element);
>            }
>            element.inject(target, beanName, pvs);
>        }
>    }
>}
>```
>
>利用for循环，遍历刚刚我们查到到的`elements列表`，进行注入。在上面有特别提醒，这里的element有可能是`AutowiredFieldElement`类型、或`AutowiredMethodElement`类型。各自代表`@Autowired注解`在属性上、以及注解在方法上的2种不同元素。因此他们调用的`element.inject(target, beanName, pvs);`也是不一样的。

##### 2.1 字段注入(AutowiredFieldElement)



##### 2.2 方法注入(AutowiredMethodElement)



