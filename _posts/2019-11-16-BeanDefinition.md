---
layout:     post
title:      Spring BeanDefinition
subtitle:   BeanDefinition 源码剖析
date:       2019-11-16
author:     Ivan.Ma
header-img: img/post-bg-universe.jpg
catalog: true
tags:
    - Spring
    - BeanDefinition
---


# BeanDefinition

## 概念
他是对bean的定义或者说描述一个bean的数据，也可以称为bean的原数据。

## 类图：
![BeanDefinition](/img/posts/BeanDefinition_class.png "类图")

## Api获取
```java
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(P02Main.class);
        String[] beanDefinitionNames = context.getBeanDefinitionNames();
        Map<String, BeanDefinition> beanDefinitionMap = new HashMap<>();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = context.getBeanDefinition(beanDefinitionName);
            if (beanDefinition.getBeanClassName().startsWith("com.javaleague.examples")) {
                beanDefinitionMap.put(beanDefinition.getClass().getName(), beanDefinition);
            }
        }
        Printer.println(beanDefinitionMap);
```