---
layout:     post
title:      SpringBoot工程 yaml配置文件映射到pojo对象
subtitle:   SpringBoot工程项目配置文件的那些事
date:       2018-11-1
author:     Dream
header-img: img/post-bg-swift.jpg
catalog: true
tags:
    - Java
    - SpringBoot
    - YAML
---

#### YAML简介

> **YAML**是一个可读性高，用来表达资料序列的编程语言。yml 文件在写的时候层次感强，而且少写了代码。所以现在很多人都使用YAML配置文件。

**YAML** 支持的数据结构有三种。
- **对象**：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
- **数组**：一组按次序排列的值，又称为序列（sequence） / 列表（list）
- **纯量（scalars）**：单个的、不可再分的值

#### YAML配置文件对比properties配置文件的优势
>**YAML**可以代替传统的xx.properties文件，但是它支持声明map,数组，list，字符串，boolean值，数值，NULL，日期，基本满足开发过程中的所有配置。而properties配置文件只能配置简单的键值对配置项。

#### YAML配置文件编写及命名
>**SpringBoot工程下自动加载src\main\resources\application.yml应用配置文件，默认读取application.yml的配置项映射到POJO对象，也可以通过实现yaml配置文件加载工厂，以使用@PropertySource注解加载指定yaml文件的配置映射到POJO对象**

src\main\resources\config\myConfig.yml
```yaml
#SMN推送消息类型
smn:
  pulishMsg:
    eventTypes:
    - eventName: "transcodeComplete"
      successMsg: "successMsg"
      errorMsg: "errorMsg"
    - eventName: "thumbnailComplete"
      successMsg: "successMsg"
      errorMsg: "errorMsg"
    - eventName: "reviewComplete"
      successMsg: "successMsg"
      errorMsg: "errorMsg"
    eventNames:
      transcodeEventName: "transcodeComplete"
      thumbnailEventName: "thumbnailComplete"
      reviewEventName: "reviewComplete"

#其他业务配置（映射到其他pojo对象）
···:
  ···:
    ···:
```

#### 实现yaml配置文件加载工厂
>实现**yaml**配置文件加载工厂，以使用@PropertySource注解加载指定yaml文件的配置

YamlPropertyLoaderFactory.java
```java
import org.springframework.boot.env.YamlPropertySourceLoader;
import org.springframework.core.env.PropertySource;
import org.springframework.core.io.support.DefaultPropertySourceFactory;
import org.springframework.core.io.support.EncodedResource;
import java.io.IOException;

/**
 * 实现yaml配置文件加载工厂，以使用@PropertySource注解加载指定yaml文件的配置
 */
public class YamlPropertyLoaderFactory extends DefaultPropertySourceFactory {
    @Override
    public PropertySource<?> createPropertySource(String name, EncodedResource resource) throws IOException {

        if (null == resource) {
            super.createPropertySource(name, resource);
        }
        return new YamlPropertySourceLoader().load(resource.getResource().getFilename(), resource.getResource()).get(0);
    }
}
```

#### YAML配置文件配置项映射到的POJO类

SmnConfigProperties.java
```java
import YamlPropertyLoaderFactory;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.PropertySource;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Map;

@Component
//通过@PropertySource注解指定要读取的yaml配置文件，默认读取src\main\resources\application.yml配置
@PropertySource(value = "classpath:config/myConfig.yml", factory = YamlPropertyLoaderFactory.class)
@ConfigurationProperties(prefix = "smn.pulish-msg")
public class SmnConfigProperties {
    private List<Map<String, String>> eventTypes;
    private Map<String, String> eventNames;

    /**
     * @param setter eventTypes
     */
    public void setEventTypes(List<Map<String, String>> eventTypes) {
        this.eventTypes = eventTypes;
    }

    /**
     * @return getter eventTypes
     */
    public List<Map<String, String>> getEventTypes() {
        return eventTypes;
    }

    public Map<String, String> getEventNames() {
        return eventNames;
    }

    public void setEventNames(Map<String, String> eventNames) {
        this.eventNames = eventNames;
    }
```

#### 注入yaml配置项映射到的POJO对象
```java
import SmnConfigProperties;

public class TimeTest {
	@Autowired
	private SmnConfigProperties smnEvents;

    @Test
    public void test() {

    }
}
```