## SpringBoot Run源码解析

### new SpringApplication

- 配置resourceLoader
- WebApplicationType.deduceFromClasspath() 判断当前应用成程序的类型（REACTIVE、NONE、SERVLET）
- 获取初始化容器的实例对象 setInitializers
  > 1.先从缓存中获取，如果非空直接返回  
  > 2.加载META-INF/spring.factories  
  > 3.加载完成，放到缓存中（cache.put(classLoader, result);） 返回数据Set      
  > 4.createSpringFactoriesInstances 实例化对象 （反射）  
  > 5.AnnotationAwareOrderComparator.sort 排序
- 初始化监听器 setListeners （同上）
- 找到当前应用程序主类 开始执行

![img.png](img.png)

### run

```java
public ConfigurableApplicationContext run(String...args){
        StopWatch stopWatch=new StopWatch();
        stopWatch.start();
        // 设置应用上下文
        DefaultBootstrapContext bootstrapContext=createBootstrapContext();
        // 设置异常报告器
        ConfigurableApplicationContext context=null;
        // 设置java.awt.headless系统参数
        configureHeadlessProperty();
        // 创建监听器对象SpringApplicationRunListener,从配置文件中读取到EventPublishingRunListener
        SpringApplicationRunListeners listeners=getRunListeners(args);
        listeners.starting(bootstrapContext,this.mainApplicationClass);
        try{
        // 创建默认的应用参数
        ApplicationArguments applicationArguments=new DefaultApplicationArguments(args);
        // 加载环境变量
        ConfigurableEnvironment environment=prepareEnvironment(listeners,bootstrapContext,applicationArguments);
        // 配置bean忽略信息
        configureIgnoreBeanInfo(environment);
        // 打印banner
        Banner printedBanner=printBanner(environment);
        // 创建应用上下文
        context=createApplicationContext();
        // 设置启动类
        context.setApplicationStartup(this.applicationStartup);
        // 配置上下文
        prepareContext(bootstrapContext,context,environment,listeners,applicationArguments,printedBanner);
        // 刷新上下文 调用refresh
        refreshContext(context);
        // 刷新之后 空方法
        afterRefresh(context,applicationArguments);
        stopWatch.stop();
        if(this.logStartupInfo){
        new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(),stopWatch);
        }
        listeners.started(context);
        callRunners(context,applicationArguments);
        }
        catch(Throwable ex){
        handleRunFailure(context,ex,listeners);
        throw new IllegalStateException(ex);
        }

        try{
        listeners.running(context);
        }
        catch(Throwable ex){
        handleRunFailure(context,ex,null);
        throw new IllegalStateException(ex);
        }
        return context;
        }
```

- 设置启动时间
- 设置应用上下文
- 设置异常报告器
- 设置java.awt.headless系统参数
- 创建监听器对象SpringApplicationRunListener,从配置文件中读取到EventPublishingRunListener
