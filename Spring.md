## spring mvc

调用handlerMapping获取HandlerExecutionChain

通过getHandlerAdapter获取适配器 以处理不同的controller 比如注解@controller 、实现Controller接口、实现HttpServletHandle接口的

执行applyPreHandle

执行对应handle方法，反射获取参数数组,创建arg数组遍历通过List<HandlerMethodArgumentResolver> argumentResolvers resolveArgument得到对应arg值，执行method

执行applyPostHandle

根据返回的modelAndView 以及tryCache捕获的异常处理结果

执行triggerAfterCompletion











## spring

### IOC 执行流程

```java
ClassPathXmlApplicationContext applicationContext = new ClassPathXmlApplicationContext("/d");
        applicationContext.getBean("");
```

```java
public ClassPathXmlApplicationContext(
			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
			throws BeansException {
		// 父类构造
		super(parent);
    	// 设置 springapplication.xml
		setConfigLocations(configLocations);
		if (refresh) {
            // 初始化
			refresh();
		}
	}
```

```java
@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
            // 初始化准备，记录启动时间、closed=false、active=true等预备操作
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
            // 初始化beanFactory并加载loadBeanDefinitions，交由BeanDefinitionsReader
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
            // 对beanFactory的一些参数设置准备工作
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				// 空实现，子类实现的钩子方法
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
                // 执行beanFactory后置处理器
                // @PriorityOrdered > @Ordered > 无注解 排序执行
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
                // 注册beanPostProcesses
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
                // 初始化messageSource - i18n所需
				initMessageSource();

				// Initialize event multicaster for this context.
                // 初始化事件广播器
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
                // 子类实现空方法
				onRefresh();

				// Check for listener beans and register them.
                // 注册监听器
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
                // 实例化所有非懒加载的单例对象
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
                // 完成 推送ContextRefreshedEvent事件
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```



1. setConfigLocation
2. refresh()



## 三级缓存

```java
// 实例化bean 提前暴露 先加入三级缓存，使用时会调用getEarlyBeanReference
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean)
    protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
            Object exposedObject = bean;
            if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
                for (BeanPostProcessor bp : getBeanPostProcessors()) {
                    if (bp instanceof SmartInstantiationAwareBeanPostProcessor) {
                        SmartInstantiationAwareBeanPostProcessor ibp = (SmartInstantiationAwareBeanPostProcessor) bp;
                        exposedObject = ibp.getEarlyBeanReference(exposedObject, beanName);
                    }
                }
            }
            return exposedObject;
    }
// 执行 invokeAwareMethods 
// 执行initializeBean
	// 执行beanPostprocessBefore，此处返回null则使用就的bean 否则使用的是此处加工处理后的bean
	// 执行 init 方法
	// 执行beanPostprocessAfter，此处返回null则使用就的bean 否则使用的是此处加工处理后的bean
// 从三级缓存种取出放入二级缓存 Object earlySingletonReference = getSingleton(beanName, false);
                        
                        
 当执行init前后埋点方法后和二级缓存中的不是一个bean对象
```





## Springboot 启动流程自动装配

```java
public static void main(String[] args) throws InterruptedException {
        SpringApplication.run(TestgroundApplication.class, args);
}

// 1. 创建SpringApplication对象 new SpringApplication(primarySources).run(args)
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
    	// primarySource 为传入的 TestgroundApplication.class
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
    	// 根据类存在推断是什么类型应用 REACTIVE、NONE、SERVLET 此处为 SERVLET
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		setInitializers((Collection) getSpringFactoriesInstances(ApplicationContextInitializer.class));
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
        // 根据栈信息找到“main”入口，返回主启动类对象
		this.mainApplicationClass = deduceMainApplicationClass();
	}

// 2. 执行 run 方法
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		ConfigurableApplicationContext context = null;
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting();
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			prepareContext(context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

