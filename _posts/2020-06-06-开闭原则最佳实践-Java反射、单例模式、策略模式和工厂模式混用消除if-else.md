---
layout:     post
title:      开闭原则最佳实践-Java反射、单例模式、策略模式和工厂模式混用消除if-else
subtitle:   CleanCode-代码整洁之道
date:       2020-06-06
author:     Dream
header-img: img/post-bg-universe.jpg
catalog: true
tags:
- java
- 设计模式
- CleanCode
---

## 开闭原则最佳实践-Java反射、单例模式、策略模式和工厂模式混用消除if-else
> 开闭原则规定“软件中的对象（类，模块，函数等等）应该对于扩展是开放的，但是对于修改是封闭的”

### 背景
一款服务为用户推出优惠套餐，每款套餐又有二级套餐分类，比如：有一款套餐A，可选基础套餐A和高级套餐A。 需要根据二级套餐Id获取二级套餐名，和二级套餐所属的一级套餐。
起初只推出了两款套餐，所以我老老实实的用if-else：

```java
if(packageId == basePackageAId){
   getBasePackageAName;
   getPackageAId;
}else if(packageId == superPackageAId){
   getSuperPackageAName;
   getPackageAId;
}else if(packageId == basePackageBId){
   getBasePackageBName;
   getPackageBId;
}else if(packageId == superPackageBId){
   getSuperPackageBName;
   getPackageBId;
}
```

将来服务又要再推出好几十款套餐，还能继续追加if-else吗？不追求代码质量，肯定不介意继续追加if-else。所以我对代码进行了整改，使代码整洁、易维护 。
### 策略模式应用
将所有的套餐抽象出一个超类:BasePackage,然后每款套餐都继承自BasePackage。
BasePackage定义:

```java
/**
*套餐超类
*/
public abstract class BasePackage {
	// 维护package的id和名称，key：packageId，value：packageName
	Map<String,String> packageMap;
	
	// 初始化packageMap
	BasePackage(Map<String,String> packageMap) {
		this.packageMap = packageMap;
	}
	
	// 根据packageId获取packageName
	public final String getPackageName(String packageId) {
		return packageMap.get(packageId);
	}
	// 获取二级套餐所属的一级套餐Id
	public abstract String getParentPackageId();
}
```

套餐A定义：

```java
/**
*套餐A
*/
public class PackageA extends BasePackage {
	// 套餐A基础套餐
	static final String BASEPACKAGEA_ID = "basePackageAId"; 
	
	// 套餐A高级套餐
	static final String SUPERPACKAGEA_ID = "superPackageAId"; 
	
	// 维护套餐A的二级套餐
	private static final Map<String,String> packageAMap = new HashMap<String,String>();

	// 初始化packageAMap
	static {
		packageAMap.put(BASEPACKAGEA_ID, "basePackageAName");
		packageAMap.put(SUPERPACKAGEA_ID, "superPackageAName");
	}

	// PackageA构造器：初始化BasePackage 的packageMap
	public PackageA() {
		super(packageAMap);
	}
	
	// 重写getParentPackageId,获取二级套餐所属的一级套餐Id
	@Override
	public String getParentPackageId() {
		return "packageAId";
	};
}
```

### 工厂模式应用
提供一个package工厂，可以根据二级套餐Id获取对应的Package实例。
PackageFactory定义：

```java
/**
*package工厂	
*/
public class PackageFactory {
	// 维护各种套餐对应的套餐实例,key:二级套餐Id，value:套餐实例 
	private static Map<String, BasePackage> packageInstanceMap = new HashMap<String, BasePackage>();

	// 初始化packageInstanceMap 
	static {
		packageInstanceMap.put(PackageA.BASEPACKAGEA_ID, new PackageA());
		packageInstanceMap.put(PackageA.SUPERPACKAGEA_ID, new PackageA()); 
	}

	// 根据二级套餐Id获取对应的Package实例
	public static BasePackage createPackage(String packageId) {
		return packageInstanceMap.get(packageId);
	}
}
```

现在可以将前文中一大坨的 if-else用两行代码替代，彻底消除了if-else：

```java
PackageFactory.createPackage(packageId).getPackageName(packageId);
PackageFactory.createPackage(packageId).getParentPackageId();
```

现在系统代码变得整洁和可维护了，但是，Package、PackageFactory 类定义的优雅吗？答案是否定的，每个二级套餐对应的套餐实例居然是重新new的。往夸张了想：如果系统中有好几万种套餐，每种套餐下又有好几万种二级套餐。那么一个服务系统，单单就一个套餐业务就存在上亿个套餐对象，系统性能肯定是不理想的。为了解决这个问题，将Package定义成单例的。

###  单例模式应用
进一步优化PackageA和PackageFactory定义。
套餐A单例实现：

```java
/**
*单例套餐A
*/
public class PackageA extends BasePackage {
	// 套餐A基础套餐
	static final String BASEPACKAGEA_ID = "basePackageAId"; 
	
	// 套餐A高级套餐
	static final String SUPERPACKAGEA_ID = "superPackageAId"; 
	
	// 维护套餐A的二级套餐
	private static final Map<String,String> packageAMap = new HashMap<String,String>();

	// 初始化packageAMap
	static {
		packageAMap.put(BASEPACKAGEA_ID, "basePackageAName");
		packageAMap.put(SUPERPACKAGEA_ID, "superPackageAName");
	}

	// PackageA私有构造器：初始化BasePackage 的packageMap
	private PackageA() {
		super(packageAMap);
	}

	// 持有packageA实例的私有静态类
	private static class PackageAHolder {
		private static PackageA packageA = new PackageA();
	}
	
	// 获取Package的单例
	public static PackageA getInstance() {
		return PackageAHolder.packageA;
	}
	
	// 重写getParentPackageId,获取二级套餐所属的一级套餐Id
	@Override
	public String getParentPackageId() {
		return "packageAId";
	};
}
```

PackageFactory持有单例的Package：

```java
/**
*持有单例Package的package工厂	
*/
public class PackageFactory {
	// 维护各种套餐对应的套餐实例,key:二级套餐Id，value:套餐实例 
	private static Map<String, BasePackage> packageInstanceMap = new HashMap<String, BasePackage>();

	// 初始化packageInstanceMap 
	static {
		packageInstanceMap.put(PackageA.BASEPACKAGEA_ID, PackageA.getInstance());
		packageInstanceMap.put(PackageA.SUPERPACKAGEA_ID, PackageA.getInstance()); 
	}

	// 根据二级套餐Id获取对应的Package实例
	public static BasePackage createPackage(String packageId) {
		return packageInstanceMap.get(packageId);
	}
}
```

这份业务代码暂时是不满足开闭原则的，以后每新增一种套餐不可能都去修改PackageFactory 吧。如果只需新增套餐类，而不用修改PackageFactory ，才是满足开闭原则的。借助java反射优化PackageFactory ,不用在PackageFactory手动获取Package实例。
### java反射应用
引入reflections maven包，用于反射获取BasePackage所有子类Package的Class。

```java
<!-- https://mvnrepository.com/artifact/org.reflections/reflections -->
<dependency>
     <groupId>org.reflections</groupId>
     <artifactId>reflections</artifactId>
     <version>0.9.11</version>
 </dependency>
```

PackageFactory反射初始化packageInstanceMap ：

```java
/**
*持有单例Package的package工厂	
*/
public class PackageFactory {
	// Package字段修饰符
	private final static String MODIFIER = "static final";

	// 获取Package单例的方法名
    private final static String GETINSTANCE = "getInstance";

	// 维护各种套餐对应的套餐实例,key:二级套餐Id，value:套餐实例 
	private static Map<String, BasePackage> packageInstanceMap = new HashMap<String, BasePackage>();

	// 反射初始化packageInstanceMap 
	static {
		final String packageName = PackageFactory.class.getPackage().getName();
		Reflections reflections = new Reflections(packageName, new SubTypesScanner());
		Set<Class<? extends BasePackage>> subTypes = reflections.getSubTypesOf(BasePackage.class);
		subTypes.forEach(subtype -> {
			try {
				Method getInstance = subtype.getMethod(GETINSTANCE);
				BasePackage package = (BasePackage) getInstance.invoke(null);
				Field[] declaredFields = subtype.getDeclaredFields();
				for (Field field : declaredFields) {
					if (StringUtils.equals(MODIFIER, Modifier.toString(field.getModifiers()))) {
						packageInstanceMap.put(field.get(subtype).toString(), package);
					}
				}
			} catch(Exception e) {
			}
		});
	}

	// 根据二级套餐Id获取对应的Package实例
	public static BasePackage createPackage(String packageId) {
		return packageInstanceMap.get(packageId);
	}
}
```

> 注：JDK8，在static代码块中，并行流并行执行lambada表达式时，会出现死锁问题。

自此，套餐业务代码满足开闭原则。下文是代码基于Spring框架的进一步精简，感兴趣的童鞋可以继续阅读。
### 注入到Spring IOC容器实现单例
基于Spring框架开发系统，Package中编写具体业务代码时，避免不了自动装配注入到Spring IOC容器中的bean，自然的将Package实例也注入到Spring IOC容器中。由于注入到Spring IOC容器的实例**默认是单例**，所以Package单例不需要手动实现。
Package实例注入到Spring IOC容器中：

```java
/**
*单例注入到Spring IOC容器的套餐A
*/
Component
public class PackageA extends BasePackage {
    // 套餐A基础套餐
    static final String BASEPACKAGEA_ID = "basePackageAId"; 

    // 套餐A高级套餐
    static final String SUPERPACKAGEA_ID = "superPackageAId"; 

    // 维护套餐A的二级套餐
    private static final Map<String,String> packageAMap = new HashMap<String,String>();

    // 初始化packageAMap
    static {
        packageAMap.put(BASEPACKAGEA_ID, "basePackageAName");
        packageAMap.put(SUPERPACKAGEA_ID, "superPackageAName");
    }

    // PackageA私有构造器：初始化BasePackage 的packageMap
    private PackageA() {
        super(packageAMap);
    }

    // 重写getParentPackageId,获取二级套餐所属的一级套餐Id
    @Override
    public String getParentPackageId() {
        return "packageAId";
    };
}
```

### 实现ApplicationContextAware接口获取Spring IOC容器中的bean
实现ApplicationContextAware接口的setApplicationContext(ApplicationContext applicationContext)方法，通过ApplicationContext 获取注入到Spring IOC容器中的bean，ApplicationContext的getBeansOfType方法可以获取指定类型的bean(包括子类)，故可以将上文引入的reflections maven包给移除。
实现ApplicationContextAware的PackageFactory：

```java
/**
*持有单例Package的package工厂	
*/
Component
public class PackageFactory implements ApplicationContextAware {
	// Package字段修饰符
	private final static String MODIFIER = "static final";

	// 维护各种套餐对应的套餐实例,key:二级套餐Id，value:套餐实例 
	private static Map<String, BasePackage> packageInstanceMap = new HashMap<String, BasePackage>();

	// 反射初始化packageInstanceMap 
	@Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		applicationContext.getBeansOfType(BasePackage.class).values().forEach(package -> {
			Class<? extends BasePackage> packageClass = package.getClass();
			Field[] declaredFields = packageClass.getDeclaredFields();
			for (Field field : declaredFields) {
				if (StringUtils.equals(MODIFIER, Modifier.toString(field.getModifiers()))) {
					try {
						packageInstanceMap.put(field.get(packageClass).toString(), package);
					} catch (IllegalAccessException e) {
					   
					}
				}
			}
		});
	}
	
	// 根据二级套餐Id获取对应的Package实例
	public static BasePackage createPackage(String packageId) {
		return packageInstanceMap.get(packageId);
	}
}
```

> Spring初始化Spring IOC容器中的bean时，PackageFactory实现的setApplicationContext方法被执行，故串行初始化packageInstanceMap 。

通过引入Spring框架，Package 和PackageFactory 被进一步精简优化。


---------
