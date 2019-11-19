---
layout:     post
title:      Dubbo学习<一>
subtitle:   Dubbo SPI
date:       2019-11-19
author:     Ivan.Ma
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Dubbo
    - SPI
---

# Dubbo学习<一> DUBBO SPI
学习路线参考: 
![学习路线](https://blog.csdn.net/zhonghuixiong/article/details/79351928)

## DUBBO SPI 及其API使用 
- Service Provider Interface简写
- 抽象接口，用一种加载机制实现可插拔的功能。
- JAVA SPI 使用
1. 抽象接口；  
1. 编写实现；  
1. 配置实现/META-INF/services/接口名，内容为：实现类全路径  

```java
public interface Animal {

    void eat();

}
```
```java
public class Cat implements Animal {
    @Override
    public void eat() {
        System.out.println("Cat eat");
    }
}
```
```java
public class Dog implements Animal {
    @Override
    public void eat() {
        System.out.println("Dog eat");
    }
}
```
```java
//路径: src/main/resources/META-INF/services/com.javaleague.dubbo.spi.javaspi.Animal
//内容: 
com.javaleague.dubbo.spi.javaspi.Cat
# com.javaleague.dubbo.spi.javaspi.Dog
```
```java
public class JavaSpiApp {

    public static void main(String[] args) {
        ServiceLoader<Animal> serviceLoader = ServiceLoader.load(Animal.class);
        for (Animal animal : serviceLoader) {
            animal.eat();
            System.out.println(animal);
        }
    }
}
```
```java
//控制台打印如下:
Cat eat
com.javaleague.dubbo.spi.javaspi.Cat@5451c3a8
```

- DUBBO SPI
1. 抽象接口（需要注解@SPI。框架已定义@SPI）； 
1. 编写实现；
1. 配置实现/META-INF/dubbo/接口名，内容为：key=实现类全路径
1. 指定运行时使用的Key  
```java
public class MyFilter implements Filter {
    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        return null;
    }
}
```
```java
//路径: src/main/resources/META-INF/dubbo/com.alibaba.dubbo.rpc.Filter
//内容: 
MyFilter = com.javaleague.dubbo.spi.dubbospi.MyFilter
```
```java
public class DubboSpiDemo {
    public static void main(String[] args) {
        ExtensionLoader<Filter> extensionLoader = ExtensionLoader.getExtensionLoader(Filter.class);
        Filter defaultFilter = extensionLoader.getDefaultExtension();
        System.out.println("Default Filter:" + defaultFilter);

        Filter myFilter = extensionLoader.getExtension("MyFilter");
        System.out.println("MyFilter: " + myFilter);
    }
}
```
```java
//输出如下:
Default Filter:null
consumerAccessLogFilter: com.javaleague.dubbo.spi.dubbospi.MyFilter@50de0926
```

- DUBBO SPI 之默认实现  
注解@SPI("defaultName")

```java
@SPI("default")
public interface IHello {

    void hello(String name);

}
```
```java
public class HelloDefaultImpl implements IHello {
    @Override
    public void hello(String name) {
        System.out.println("This is default impl: " + name);
    }
}
```
```java
public class HelloSecondImpl implements IHello {
    @Override
    public void hello(String name) {
        System.out.println("This is the second impl: " + name);
    }
}
```
```java
//路径: src/main/resources/META-INF/dubbo/com.javaleague.dubbo.spi.dubbospi.defaultdemo
//内容: 
default = com.javaleague.dubbo.spi.dubbospi.defaultdemo.HelloDefaultImpl
second = com.javaleague.dubbo.spi.dubbospi.defaultdemo.HelloSecondImpl
```
```java
public class DubboSpiDefaultDemo {
    public static void main(String[] args) {
        ExtensionLoader<IHello> extensionLoader = ExtensionLoader.getExtensionLoader(IHello.class);
        IHello defaultInstance = extensionLoader.getDefaultExtension();
        System.out.println("Default:" + defaultInstance);

        IHello second = extensionLoader.getExtension("second");
        System.out.println("Second: " + second);
    }
}
```
```java
//控制台输出如下:
Default:com.javaleague.dubbo.spi.dubbospi.defaultdemo.HelloDefaultImpl@4fca772d
Second: com.javaleague.dubbo.spi.dubbospi.defaultdemo.HelloSecondImpl@9807454
```

- DUBBO SPI 之动态实现<一>  
@Adaptive + extensionLoader.getAdaptiveExtension()

```java
@SPI("default")
public interface IHello {

    @Adaptive({"hello"})
    void hello(String name, URL url);

}
```
```java
public class HelloDefaultImpl implements IHello {
    @Override
    public void hello(String name, URL url) {
        System.out.println("This is default impl: " + name);
    }
}
public class HelloSecondImpl implements IHello {
    @Override
    public void hello(String name, URL url) {
        System.out.println("This is the second impl: " + name);
    }
}
//@Adaptive
public class HelloThirdImpl implements IHello {
    @Override
    public void hello(String name, URL url) {
        System.out.println("This is third impl: " + name);
    }
}
```
```java
//路径: META-INF/dubbo/com.javaleague.dubbo.spi.dubbospi.adaptive.IHello
//内容如下:
default = com.javaleague.dubbo.spi.dubbospi.adaptive.HelloDefaultImpl
second = com.javaleague.dubbo.spi.dubbospi.adaptive.HelloSecondImpl
third = com.javaleague.dubbo.spi.dubbospi.adaptive.HelloThirdImpl
```
```java
public class DubboSpiAdaptiveDemo {
    public static void main(String[] args) {
        invoke(URL.valueOf("test://localhost/test"));
        invoke(URL.valueOf("test://localhost/test?hello=second"));
    }

    private static void invoke(URL url) {
        ExtensionLoader<IHello> extensionLoader = ExtensionLoader.getExtensionLoader(IHello.class);
        IHello instance = extensionLoader.getAdaptiveExtension();
        System.out.println(url);
        instance.hello("zhangsan", url);
        System.out.println("---------------------------");
    }
}
```
```java
//控制台输出如下:
test://localhost/test
This is default impl: zhangsan
---------------------------
test://localhost/test?hello=second
This is the second impl: zhangsan
---------------------------
```
```java
//如果HelloThirdImpl类上的注解去掉,控制台输出如下:
test://localhost/test
This is third impl: zhangsan
---------------------------
test://localhost/test?hello=second
This is third impl: zhangsan
---------------------------
```
结论:
- 如果SPI接口有实现类且类上注解@Adaptive，则会默认返回该实现类的实例(硬编码指定,优先级最高)
- 如果SPI接口的方法上有注解@Adaptive且参数不为空，则根据url的参数匹配实现类。
- 如果SPI接口的方法上有注解@Adaptive但参数为空，则参数名=接口使用.分隔，如：IHello分隔后为i.hello,然后根据url的参数匹配。
- 如果url中有这个参数，但根据参数值无法找到扩展点的名称，则抛异常。
- 如果url中无这个参数，则使用默认的扩展点名称。

> dubbo设计@adaptive注解的原因
为什么要设计adaptive？注解在类上和注解在方法上的区别？
adaptive设计的目的是为了识别固定已知类和扩展未知类。  
1.注解在类上：代表人工实现，实现一个装饰类（设计模式中的装饰模式），它主要作用于固定已知类，
目前整个系统只有2个，AdaptiveCompiler、AdaptiveExtensionFactory。  
a.为什么AdaptiveCompiler这个类是固定已知的？因为整个框架仅支持Javassist和JdkCompiler。  
a.为什么AdaptiveExtensionFactory这个类是固定已知的？因为整个框架仅支持2个objFactory,一个是spi,另一个是spring  
2.注解在方法上：代表自动生成和编译一个动态的Adpative类，它主要是用于SPI，因为spi的类是不固定、未知的扩展类，所以设计了动态$Adaptive类.
例如 Protocol的spi类有 injvm dubbo registry filter listener等等 很多扩展未知类，
它设计了Protocol$Adaptive的类，通过ExtensionLoader.getExtensionLoader(Protocol.class).getExtension(spi类);来提取对象  


- DUBBO SPI 之动态实现<二>  
上述方法只返回一个SPI接口的实例，下面讲返回到多个SPI接口的实例。  

```java
@SPI("default")
public interface IHello {

    void hello(String name, URL url);

}
```
```java
public class HelloDefaultImpl implements IHello {
    @Override
    public void hello(String name, URL url) {
        System.out.println("This is default impl: " + name);
    }
}
public class HelloSecondImpl implements IHello {
    @Override
    public void hello(String name, URL url) {
        System.out.println("This is the second impl: " + name);
    }
}
@Activate(group = Constants.CONSUMER)
public class HelloThirdImpl implements IHello {
    @Override
    public void hello(String name, URL url) {
        System.out.println("This is third impl: " + name);
    }
}
```
```java
//spi配置如下:
default = com.javaleague.dubbo.spi.dubbospi.activate.HelloDefaultImpl
first = com.javaleague.dubbo.spi.dubbospi.activate.HelloDefaultImpl
second = com.javaleague.dubbo.spi.dubbospi.activate.HelloSecondImpl
third = com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl
```
```java
public class DubboSpiActivateDemo {
    public static void main(String[] args) {
        System.out.println("------------通过key指定---------------");
        invoke(URL.valueOf("test://localhost/test"), "hello");
        invoke(URL.valueOf("test://localhost/test?hello=default"), "hello");
        //会返回@Activate注解的实现+扩展点名称的实现
        invoke(URL.valueOf("test://localhost/test?hello=first"), "hello");
        invoke(URL.valueOf("test://localhost/test?hello=second"), "hello");
        invoke(URL.valueOf("test://localhost/test?hello=third"), "hello");
        invoke(URL.valueOf("test://localhost/test?hello=second,third"), "hello");
        System.out.println("------------------------------");
        invoke(URL.valueOf("test://localhost/test?hello=second"), "hello", Constants.CONSUMER);
        //如果指定最后一个default,则指定的扩展点排在前面
        invoke(URL.valueOf("test://localhost/test?hello=second,default"), "hello", Constants.CONSUMER);
    }

    private static void invoke(URL url, String key) {
        ExtensionLoader<IHello> extensionLoader = ExtensionLoader.getExtensionLoader(IHello.class);
        List<IHello> list = extensionLoader.getActivateExtension(url, key);
        System.out.println(list);
    }

    private static void invoke(URL url, String key, String group) {
        ExtensionLoader<IHello> extensionLoader = ExtensionLoader.getExtensionLoader(IHello.class);
        List<IHello> list = extensionLoader.getActivateExtension(url, key, group);
        System.out.println(list);
    }
}
```
```java
//控制台输出如下:
[com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl@30dae81]
[com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl@30dae81]
[com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl@30dae81, com.javaleague.dubbo.spi.dubbospi.activate.HelloDefaultImpl@70177ecd]
[com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl@30dae81, com.javaleague.dubbo.spi.dubbospi.activate.HelloSecondImpl@1e80bfe8]
[com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl@30dae81]
[com.javaleague.dubbo.spi.dubbospi.activate.HelloSecondImpl@1e80bfe8, com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl@30dae81]
------------------------------
[com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl@30dae81, com.javaleague.dubbo.spi.dubbospi.activate.HelloSecondImpl@1e80bfe8]
[com.javaleague.dubbo.spi.dubbospi.activate.HelloSecondImpl@1e80bfe8, com.javaleague.dubbo.spi.dubbospi.activate.HelloThirdImpl@30dae81]
```
总结:
- 返回值包含两部分,一部分为用户指定扩展点,一部分为自动激活的扩展点
- 默认情况下是自动激活的扩展点排在前面.如果想用户指定的扩展点在前,则在用户指定的扩展点后加上default
- @Activate中的value属性用于过滤.过滤条件为url中是否包含这些参数



## DUBBO SPI 源码分析
### ExtensionLoader.getDefaultExtension() 和 ExtensionLoader#getExtension(String)
- 程序入口
```java
public class DubboSpiDemo {
    public static void main(String[] args) {
        //获取@SPI接口获取ExtensionLoader, 这里使用了多例模式.1个接口对应一个ExtensionLoader
        ExtensionLoader<Filter> extensionLoader = ExtensionLoader.getExtensionLoader(Filter.class);
        Filter defaultFilter = extensionLoader.getDefaultExtension();
        System.out.println("Default Filter:" + defaultFilter);
    }
}
```
- com.alibaba.dubbo.common.extension.ExtensionLoader#getExtensionLoader
```java
    // 存储@SPI接口对应的ExtensionLoader
    private static final ConcurrentMap<Class<?>, ExtensionLoader<?>> EXTENSION_LOADERS = new ConcurrentHashMap<Class<?>, ExtensionLoader<?>>();
    
    public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
        if (type == null)
            throw new IllegalArgumentException("Extension type == null");
        if(!type.isInterface()) {
            throw new IllegalArgumentException("Extension type(" + type + ") is not interface!");
        }
        if(!withExtensionAnnotation(type)) {
            throw new IllegalArgumentException("Extension type(" + type + 
                    ") is not extension, because WITHOUT @" + SPI.class.getSimpleName() + " Annotation!");
        }
        // 从缓存中获取ExtensionLoader, 如果没有, 则实例化一个
        ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        if (loader == null) {
            EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
            loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        }
        return loader;
    }
```
- com.alibaba.dubbo.common.extension.ExtensionLoader#getExtension
```java
    private final ConcurrentMap<String, Holder<Object>> cachedInstances = new ConcurrentHashMap<String, Holder<Object>>();
    
    /**
     * 返回指定名字的扩展。如果指定名字的扩展不存在，则抛异常 {@link IllegalStateException}.
     *
     * @param name
     * @return
     */
	public T getExtension(String name) {
		if (name == null || name.length() == 0)
		    throw new IllegalArgumentException("Extension name == null");
		if ("true".equals(name)) {
		    return getDefaultExtension();
		}
		// 从缓存中获取Holder, 如果没有, 则实例化一个.
		Holder<Object> holder = cachedInstances.get(name);
		if (holder == null) {
		    cachedInstances.putIfAbsent(name, new Holder<Object>());
		    holder = cachedInstances.get(name);
		}
		// 从holder中获取extension, 如果没有, 则创建一个
		Object instance = holder.get();
		if (instance == null) {
		    synchronized (holder) {
	            instance = holder.get();
	            if (instance == null) {
	                instance = createExtension(name);
	                holder.set(instance);
	            }
	        }
		}
		return (T) instance;
	}
```
- 
```java
    // 根据扩展点名称创建一个该@SPI接口的实例
    private T createExtension(String name) {
        Class<?> clazz = getExtensionClasses().get(name);
        if (clazz == null) {
            throw findException(name);
        }
        try {
            // 从缓存中获取实例, 如果没有, 则实例化一个
            T instance = (T) EXTENSION_INSTANCES.get(clazz);
            if (instance == null) {
                EXTENSION_INSTANCES.putIfAbsent(clazz, (T) clazz.newInstance());
                instance = (T) EXTENSION_INSTANCES.get(clazz);
            }
            // 注入属性 IOC实现埋点. 后面再解析
            injectExtension(instance);
            // wrapperClasses  AOP实现埋点, 后面再解析
            Set<Class<?>> wrapperClasses = cachedWrapperClasses;
            if (wrapperClasses != null && wrapperClasses.size() > 0) {
                // 如果有wrapper, 则使用wrapper增强
                for (Class<?> wrapperClass : wrapperClasses) {
                    instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance));
                }
            }
            return instance;
        } catch (Throwable t) {
            throw new IllegalStateException("Extension instance(name: " + name + ", class: " +
                    type + ")  could not be instantiated: " + t.getMessage(), t);
        }
    }
```
- 这样就根据扩展点名称, 创建了一个该扩展点接口的实例
- 可以从上面的代码中看出使用了大量的缓存. 以及使用双重判断(为空)的方式来保证单例.

### com.alibaba.dubbo.common.extension.ExtensionLoader#getAdaptiveExtension
```java
    // 获取自适配的扩展实例
    public T getAdaptiveExtension() {
        Object instance = cachedAdaptiveInstance.get();
        if (instance == null) {
            if(createAdaptiveInstanceError == null) { // 加载实例时未出错
                synchronized (cachedAdaptiveInstance) {
                    instance = cachedAdaptiveInstance.get();
                    if (instance == null) {
                        try {
                            // 如果缓存中没有, 则创建
                            instance = createAdaptiveExtension();
                            cachedAdaptiveInstance.set(instance);
                        } catch (Throwable t) {
                            createAdaptiveInstanceError = t;
                            throw new IllegalStateException("fail to create adaptive instance: " + t.toString(), t);
                        }
                    }
                }
            }
            else {
                throw new IllegalStateException("fail to create adaptive instance: " + createAdaptiveInstanceError.toString(), createAdaptiveInstanceError);
            }
        }

        return (T) instance;
    }
```
```java
    private T createAdaptiveExtension() {
        try {
            // 获取AdaptiveExtensionClass,并实例化, 然后注入属性
            return injectExtension((T) getAdaptiveExtensionClass().newInstance());
        } catch (Exception e) {
            throw new IllegalStateException("Can not create adaptive extenstion " + type + ", cause: " + e.getMessage(), e);
        }
    }
```
```java
    // 如果有被注解@Adaptive的类, 则返回该Class
    // 如果没有, 则动态生成一个adapter的类
    private Class<?> getAdaptiveExtensionClass() {
        getExtensionClasses();// 加载扩展类
        if (cachedAdaptiveClass != null) {   //该实例存储的是实现类上被注解@Adaptive的class实例
            return cachedAdaptiveClass;
        }
        // 如果没有@Adaptive注解的实现类,则动态创建一个类,并编译
        return cachedAdaptiveClass = createAdaptiveExtensionClass();
    }
    
    private Class<?> createAdaptiveExtensionClass() {
        //生成的适配器的类的java代码, 然后编译
        String code = createAdaptiveExtensionClassCode();
        ClassLoader classLoader = findClassLoader();
        com.alibaba.dubbo.common.compiler.Compiler compiler = ExtensionLoader.getExtensionLoader(com.alibaba.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();
        return compiler.compile(code, classLoader);
    }
```
```java
//动态生成的java如下:
package com.javaleague.dubbo;

import com.alibaba.dubbo.common.extension.ExtensionLoader;

public class IHello$Adpative implements com.javaleague.dubbo.spi.dubbospi.adaptive.IHello {
    
    // 根据url中的参数来获取extension, 并调用方法.
    public void hello(java.lang.String arg0, com.alibaba.dubbo.common.URL arg1) {
        if (arg1 == null) throw new IllegalArgumentException("url == null");
        com.alibaba.dubbo.common.URL url = arg1;
        String extName = url.getParameter("hello", "default");
        if (extName == null)
            throw new IllegalStateException("Fail to get extension(com.javaleague.dubbo.spi.dubbospi.adaptive.IHello) name from url(" + url.toString() + ") use keys([hello])");
        com.javaleague.dubbo.spi.dubbospi.adaptive.IHello extension = (com.javaleague.dubbo.spi.dubbospi.adaptive.IHello) ExtensionLoader.getExtensionLoader(com.javaleague.dubbo.spi.dubbospi.adaptive.IHello.class).getExtension(extName);
        extension.hello(arg0, arg1);
    }
}
```
### ExtensionLoader.getActivateExtension(url, key, group);
```java
    // 根据url获取其key的参数值,call getActivateExtension(url, values, group)
    public List<T> getActivateExtension(URL url, String key, String group) {
        String value = url.getParameter(key);
        return getActivateExtension(url, value == null || value.length() == 0 ? null : Constants.COMMA_SPLIT_PATTERN.split(value), group);
    }
    
    // 根据url, values(扩展点的名称列表), group(消费端还是服务端)
    public List<T> getActivateExtension(URL url, String[] values, String group) {
        List<T> exts = new ArrayList<T>();
        List<String> names = values == null ? new ArrayList<String>(0) : Arrays.asList(values);
        if (! names.contains(Constants.REMOVE_VALUE_PREFIX + Constants.DEFAULT_KEY)) {
            getExtensionClasses();
            // 自动激活包含@Activate的类
            for (Map.Entry<String, Activate> entry : cachedActivates.entrySet()) {
                String name = entry.getKey();
                Activate activate = entry.getValue();
                //两个过滤条件 group和isActive(activate, url)
                if (isMatchGroup(group, activate.group())) {
                    T ext = getExtension(name);
                    if (! names.contains(name)
                            && ! names.contains(Constants.REMOVE_VALUE_PREFIX + name) 
                            && isActive(activate, url)) {
                        exts.add(ext);
                    }
                }
            }
            //支持排序
            Collections.sort(exts, ActivateComparator.COMPARATOR);
        }
        // 用户指定的扩展点处理
        List<T> usrs = new ArrayList<T>();
        for (int i = 0; i < names.size(); i ++) {
        	String name = names.get(i);
            if (! name.startsWith(Constants.REMOVE_VALUE_PREFIX)
            		&& ! names.contains(Constants.REMOVE_VALUE_PREFIX + name)) {
            	// 如果是default的扩展点, 就会将用户指定的扩展前排到最前面
            	if (Constants.DEFAULT_KEY.equals(name)) {
            		if (usrs.size() > 0) {
	            		exts.addAll(0, usrs);
	            		usrs.clear();
            		}
            	} else {
	            	T ext = getExtension(name);
	            	usrs.add(ext);
            	}
            }
        }
        if (usrs.size() > 0) {
            // 加入用户的扩展点
        	exts.addAll(usrs);
        }
        return exts;
    }
```

## DUBBO Ioc源码解析
- 上面有提到达injectExtension, 这里就是DUBBO Ioc的入口
```java
    private T injectExtension(T instance) {
        try {
            if (objectFactory != null) {
                for (Method method : instance.getClass().getMethods()) {
                    //通过setter注入, 并且set方法是public的
                    if (method.getName().startsWith("set")
                            && method.getParameterTypes().length == 1
                            && Modifier.isPublic(method.getModifiers())) {
                        Class<?> pt = method.getParameterTypes()[0];
                        try {
                            String property = method.getName().length() > 3 ? method.getName().substring(3, 4).toLowerCase() + method.getName().substring(4) : "";
                            //最终是通过objectFactory来获取的 pt为类型, property为属性名
                            Object object = objectFactory.getExtension(pt, property);
                            if (object != null) {
                                method.invoke(instance, object);
                            }
                        } catch (Exception e) {
                            logger.error("fail to inject via method " + method.getName()
                                    + " of interface " + type.getName() + ": " + e.getMessage(), e);
                        }
                    }
                }
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
        return instance;
    }
```
- ExtensionLoader的objectFactory解析如下:
```java
    // objectFactory为com.alibaba.dubbo.common.extension.factory.AdaptiveExtensionFactory
    private final ExtensionFactory objectFactory;
    private ExtensionLoader(Class<?> type) {
        this.type = type;
        objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
    }
```
- 解析com.alibaba.dubbo.common.extension.factory.AdaptiveExtensionFactory
```java
@Adaptive
public class AdaptiveExtensionFactory implements ExtensionFactory {
    //factories其实就是dubbou*.jar中声明的
    //spi=com.alibaba.dubbo.common.extension.factory.SpiExtensionFactory
    //spring=com.alibaba.dubbo.config.spring.extension.SpringExtensionFactory
    private final List<ExtensionFactory> factories;
    
    public AdaptiveExtensionFactory() {
        ExtensionLoader<ExtensionFactory> loader = ExtensionLoader.getExtensionLoader(ExtensionFactory.class);
        List<ExtensionFactory> list = new ArrayList<ExtensionFactory>();
        for (String name : loader.getSupportedExtensions()) {
            list.add(loader.getExtension(name));
        }
        factories = Collections.unmodifiableList(list);
    }
    
    //从SpiExtensionFactory和SpringExtensionFactory中获取实例
    public <T> T getExtension(Class<T> type, String name) {
        for (ExtensionFactory factory : factories) {
            T extension = factory.getExtension(type, name);
            if (extension != null) {
                return extension;
            }
        }
        return null;
    }

}
```
- 解析SpiExtensionFactory
```java
public class SpiExtensionFactory implements ExtensionFactory {
    
    public <T> T getExtension(Class<T> type, String name) {
        if (type.isInterface() && type.isAnnotationPresent(SPI.class)) {
            ExtensionLoader<T> loader = ExtensionLoader.getExtensionLoader(type);
            if (loader.getSupportedExtensions().size() > 0) {
                // 这里不用多说, 实际上就是上面讲的@Adaptive相关的内容
                return loader.getAdaptiveExtension();
            }
        }
        return null;
    }

}
```
- 解析SpringExtensionFactory
```java
public class SpringExtensionFactory implements ExtensionFactory {
    
    private static final Set<ApplicationContext> contexts = new ConcurrentHashSet<ApplicationContext>();
    //使用静态方法持有spring的context, 可以加入多个context
    public static void addApplicationContext(ApplicationContext context) {
        contexts.add(context);
    }

    public static void removeApplicationContext(ApplicationContext context) {
        contexts.remove(context);
    }

    @SuppressWarnings("unchecked")
    public <T> T getExtension(Class<T> type, String name) {
        for (ApplicationContext context : contexts) {
            // 而这里实际上就是我们熟悉的从spring context中获取bean了, 是通过bean名称获取的
            if (context.containsBean(name)) {
                Object bean = context.getBean(name);
                if (type.isInstance(bean)) {
                    return (T) bean;
                }
            }
        }
        return null;
    }

}
```
运行的示例代码如下:  
```java
@SPI
public interface IHello {

    void hello(String name);

}
```
```java
public class HelloDefaultImpl implements IHello {
    private String prop1;

    //这里是我们期望注入的属性, 使用setter注入
    public void setProp1(String prop1) {
        this.prop1 = prop1;
    }

    @Override
    public void hello(String name) {
        System.out.println("This is the second impl: " + name);
        System.out.println("prop1:" + prop1);
    }
}
```
```java
//com.javaleague.dubbo.ioc.IHello文件内容如下:
first = com.javaleague.dubbo.ioc.HelloDefaultImpl
```
```java
@Configuration
public class SpringConfig {

    @Bean
    public String prop1() {
        return "age: 18";
    }

}
```
```java
public class DubboIocDemo {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(SpringConfig.class);
        SpringExtensionFactory.addApplicationContext(context);

        ExtensionLoader<IHello> extensionLoader = ExtensionLoader.getExtensionLoader(IHello.class);

        IHello first = extensionLoader.getExtension("first");
        first.hello("zhangsan");
    }
}
```
```java
//控制台输出如下:
This is the second impl: zhangsan
prop1:age: 18
```
## DUBBO Aop源码解析
- 上面有埋点Aop代码, 我们就直接从埋点处开始解析.
```java
    private T createExtension(String name) {
        Class<?> clazz = getExtensionClasses().get(name);
        if (clazz == null) {
            throw findException(name);
        }
        try {
            T instance = (T) EXTENSION_INSTANCES.get(clazz);
            if (instance == null) {
                EXTENSION_INSTANCES.putIfAbsent(clazz, (T) clazz.newInstance());
                instance = (T) EXTENSION_INSTANCES.get(clazz);
            }
            injectExtension(instance);
            // aop 增强 实际上dubbo使用的是静态代理
            Set<Class<?>> wrapperClasses = cachedWrapperClasses;
            if (wrapperClasses != null && wrapperClasses.size() > 0) {
                for (Class<?> wrapperClass : wrapperClasses) {
                    // 这里可以看出wrapperClass的标准
                    // 需要有一个构造方法,且有1个参数,参数类型为@SPI的接口
                    instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance));
                }
            }
            return instance;
        } catch (Throwable t) {
            throw new IllegalStateException("Extension instance(name: " + name + ", class: " +
                    type + ")  could not be instantiated: " + t.getMessage(), t);
        }
    }
```
- 那么cachedWrapperClasses的值是从哪里设置的呢? 我们看ExtensionLoader设置该属性的地方可以发现loadFile
```java
private void loadFile(Map<String, Class<?>> extensionClasses, String dir) {
        String fileName = dir + type.getName();
        try {
            Enumeration<java.net.URL> urls;
            ClassLoader classLoader = findClassLoader();
            if (classLoader != null) {
                urls = classLoader.getResources(fileName);
            } else {
                urls = ClassLoader.getSystemResources(fileName);
            }
            if (urls != null) {
                while (urls.hasMoreElements()) {
                    java.net.URL url = urls.nextElement();
                    try {
                        BufferedReader reader = new BufferedReader(new InputStreamReader(url.openStream(), "utf-8"));
                        try {
                            String line = null;
                            while ((line = reader.readLine()) != null) {
                                final int ci = line.indexOf('#');
                                if (ci >= 0) line = line.substring(0, ci);
                                line = line.trim();
                                if (line.length() > 0) {
                                    try {
                                        String name = null;
                                        int i = line.indexOf('=');
                                        if (i > 0) {
                                            name = line.substring(0, i).trim();
                                            line = line.substring(i + 1).trim();
                                        }
                                        if (line.length() > 0) {
                                            Class<?> clazz = Class.forName(line, true, classLoader);
                                            if (! type.isAssignableFrom(clazz)) {
                                                throw new IllegalStateException("Error when load extension class(interface: " +
                                                        type + ", class line: " + clazz.getName() + "), class " 
                                                        + clazz.getName() + "is not subtype of interface.");
                                            }
                                            if (clazz.isAnnotationPresent(Adaptive.class)) {
                                                if(cachedAdaptiveClass == null) {
                                                    cachedAdaptiveClass = clazz;
                                                } else if (! cachedAdaptiveClass.equals(clazz)) {
                                                    throw new IllegalStateException("More than 1 adaptive class found: "
                                                            + cachedAdaptiveClass.getClass().getName()
                                                            + ", " + clazz.getClass().getName());
                                                }
                                            } else {
                                                try {
                                                    //获取带@SPI接口的构造方法,如果不报错, 则会加入到cachedWrapperClasses
                                                    clazz.getConstructor(type);
                                                    Set<Class<?>> wrappers = cachedWrapperClasses;
                                                    if (wrappers == null) {
                                                        cachedWrapperClasses = new ConcurrentHashSet<Class<?>>();
                                                        wrappers = cachedWrapperClasses;
                                                    }
                                                    wrappers.add(clazz);
                                                } catch (NoSuchMethodException e) {
                                                    clazz.getConstructor();
                                                    if (name == null || name.length() == 0) {
                                                        name = findAnnotationName(clazz);
                                                        if (name == null || name.length() == 0) {
                                                            if (clazz.getSimpleName().length() > type.getSimpleName().length()
                                                                    && clazz.getSimpleName().endsWith(type.getSimpleName())) {
                                                                name = clazz.getSimpleName().substring(0, clazz.getSimpleName().length() - type.getSimpleName().length()).toLowerCase();
                                                            } else {
                                                                throw new IllegalStateException("No such extension name for the class " + clazz.getName() + " in the config " + url);
                                                            }
                                                        }
                                                    }
                                                    String[] names = NAME_SEPARATOR.split(name);
                                                    if (names != null && names.length > 0) {
                                                        Activate activate = clazz.getAnnotation(Activate.class);
                                                        if (activate != null) {
                                                            cachedActivates.put(names[0], activate);
                                                        }
                                                        for (String n : names) {
                                                            if (! cachedNames.containsKey(clazz)) {
                                                                cachedNames.put(clazz, n);
                                                            }
                                                            Class<?> c = extensionClasses.get(n);
                                                            if (c == null) {
                                                                extensionClasses.put(n, clazz);
                                                            } else if (c != clazz) {
                                                                throw new IllegalStateException("Duplicate extension " + type.getName() + " name " + n + " on " + c.getName() + " and " + clazz.getName());
                                                            }
                                                        }
                                                    }
                                                }
                                            }
                                        }
                                    } catch (Throwable t) {
                                        IllegalStateException e = new IllegalStateException("Failed to load extension class(interface: " + type + ", class line: " + line + ") in " + url + ", cause: " + t.getMessage(), t);
                                        exceptions.put(line, e);
                                    }
                                }
                            } // end of while read lines
                        } finally {
                            reader.close();
                        }
                    } catch (Throwable t) {
                        logger.error("Exception when load extension class(interface: " +
                                            type + ", class file: " + url + ") in " + url, t);
                    }
                } // end of while urls
            }
        } catch (Throwable t) {
            logger.error("Exception when load extension class(interface: " +
                    type + ", description file: " + fileName + ").", t);
        }
    }
```
- 下面我们来写个例子
```java
@SPI
public interface IHello {

    void hello(String name);

}
```
```java
public class HelloDefaultImpl implements IHello {

    @Override
    public void hello(String name) {
        System.out.println("This is the second impl: " + name);
    }
}
```
```java
public class HelloWrappedImpl implements IHello {

    private IHello hello;

    public HelloWrappedImpl(IHello hello) {
        this.hello = hello;
    }

    @Override
    public void hello(String name) {
        System.out.println("before");
        hello.hello(name);
        System.out.println("after");
    }
}
```
```java
// com.javaleague.dubbo.aop.IHello内容如下:
first=com.javaleague.dubbo.aop.HelloDefaultImpl
wrapper1 = com.javaleague.dubbo.aop.HelloWrappedImpl
```
```java
public class DubboIocDemo {
    public static void main(String[] args) {
        ExtensionLoader<IHello> extensionLoader = ExtensionLoader.getExtensionLoader(IHello.class);
        IHello first = extensionLoader.getExtension("first");
        first.hello("zhangsan");
    }
}
```
```java
//控制台输出如下:
before
This is the second impl: zhangsan
after
```


## DUBBO 动态编译
参考: https://blog.csdn.net/u012410733/article/details/77926625