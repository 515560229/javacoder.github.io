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
```json
{
    "org.springframework.context.annotation.ScannedGenericBeanDefinition": {
        "abstract": false,
        "autowireCandidate": true,
        "autowireMode": 0,
        "beanClassName": "com.javaleague.examples.spring4.common.BeanD",
        "constructorArgumentValues": {
            "argumentCount": 0,
            "empty": true,
            "genericArgumentValues": [],
            "indexedArgumentValues": {}
        },
        "dependencyCheck": 0,
        "enforceDestroyMethod": false,
        "enforceInitMethod": false,
        "lazyInit": false,
        "lenientConstructorResolution": true,
        "metadata": {
            "abstract": false,
            "annotation": false,
            "annotationTypes": [
                "org.springframework.stereotype.Component"
            ],
            "className": "com.javaleague.examples.spring4.common.BeanD",
            "concrete": true,
            "final": false,
            "independent": true,
            "interface": false,
            "interfaceNames": [],
            "memberClassNames": [],
            "superClassName": "java.lang.Object"
        },
        "methodOverrides": {
            "empty": true,
            "overrides": []
        },
        "nonPublicAccessAllowed": true,
        "primary": false,
        "propertyValues": {
            "converted": false,
            "empty": true,
            "propertyValueList": [],
            "propertyValues": []
        },
        "prototype": false,
        "qualifiers": [],
        "resolvedAutowireMode": 0,
        "resource": {
            "description": "file [F:\\git\\spring4-exaples\\spring4\\target\\classes\\com\\javaleague\\examples\\spring4\\common\\BeanD.class]",
            "file": "F:\\git\\spring4-exaples\\spring4\\target\\classes\\com\\javaleague\\examples\\spring4\\common\\BeanD.class",
            "filename": "BeanD.class",
            "inputStream": {
                "channel": {
                    "open": true
                },
                "fD": {}
            },
            "open": false,
            "outputStream": {
                "channel": {
                    "open": true
                },
                "fD": {}
            },
            "path": "F:/git/spring4-exaples/spring4/target/classes/com/javaleague/examples/spring4/common/BeanD.class",
            "readable": true,
            "uRI": "file:/F:/git/spring4-exaples/spring4/target/classes/com/javaleague/examples/spring4/common/BeanD.class",
            "uRL": "file:/F:/git/spring4-exaples/spring4/target/classes/com/javaleague/examples/spring4/common/BeanD.class",
            "writable": true
        },
        "resourceDescription": "file [F:\\git\\spring4-exaples\\spring4\\target\\classes\\com\\javaleague\\examples\\spring4\\common\\BeanD.class]",
        "role": 0,
        "scope": "singleton",
        "singleton": true,
        "source": {
            "$ref": "$.org.springframework.context.annotation.ScannedGenericBeanDefinition.resource"
        },
        "synthetic": false
    },
    "org.springframework.beans.factory.support.GenericBeanDefinition": {
        "abstract": false,
        "autowireCandidate": true,
        "autowireMode": 0,
        "beanClassName": "com.javaleague.examples.spring4.common.BeanC",
        "constructorArgumentValues": {
            "argumentCount": 0,
            "empty": true,
            "genericArgumentValues": [],
            "indexedArgumentValues": {}
        },
        "dependencyCheck": 0,
        "enforceDestroyMethod": true,
        "enforceInitMethod": true,
        "lazyInit": false,
        "lenientConstructorResolution": true,
        "methodOverrides": {
            "empty": true,
            "overrides": []
        },
        "nonPublicAccessAllowed": true,
        "primary": false,
        "propertyValues": {
            "converted": false,
            "empty": false,
            "propertyValueList": [
                {
                    "converted": false,
                    "name": "beanA",
                    "optional": false,
                    "originalPropertyValue": {
                        "$ref": "@"
                    },
                    "value": {
                        "beanName": "beanA",
                        "toParent": false
                    }
                }
            ],
            "propertyValues": [
                {
                    "$ref": "$.org.springframework.beans.factory.support.GenericBeanDefinition.propertyValues.propertyValueList[0]"
                }
            ]
        },
        "prototype": false,
        "qualifiers": [],
        "resolvedAutowireMode": 0,
        "resource": {
            "description": "URL [file:/F:/git/spring4-exaples/spring4/target/classes/p02.xml]",
            "file": "F:\\git\\spring4-exaples\\spring4\\target\\classes\\p02.xml",
            "filename": "p02.xml",
            "inputStream": {},
            "open": false,
            "readable": true,
            "uRI": "file:/F:/git/spring4-exaples/spring4/target/classes/p02.xml",
            "uRL": "file:/F:/git/spring4-exaples/spring4/target/classes/p02.xml"
        },
        "resourceDescription": "URL [file:/F:/git/spring4-exaples/spring4/target/classes/p02.xml]",
        "role": 0,
        "scope": "",
        "singleton": true,
        "synthetic": false
    },
    "org.springframework.beans.factory.annotation.AnnotatedGenericBeanDefinition": {
        "abstract": false,
        "autowireCandidate": true,
        "autowireMode": 0,
        "beanClassName": "com.javaleague.examples.spring4.p02.P02Main",
        "constructorArgumentValues": {
            "argumentCount": 0,
            "empty": true,
            "genericArgumentValues": [],
            "indexedArgumentValues": {}
        },
        "dependencyCheck": 0,
        "enforceDestroyMethod": true,
        "enforceInitMethod": true,
        "lazyInit": false,
        "lenientConstructorResolution": true,
        "metadata": {
            "abstract": false,
            "annotation": false,
            "annotationTypes": [
                "org.springframework.context.annotation.ComponentScan",
                "org.springframework.context.annotation.ImportResource"
            ],
            "className": "com.javaleague.examples.spring4.p02.P02Main",
            "concrete": true,
            "final": false,
            "independent": true,
            "interface": false,
            "interfaceNames": [],
            "introspectedClass": "com.javaleague.examples.spring4.p02.P02Main",
            "memberClassNames": [],
            "superClassName": "java.lang.Object"
        },
        "methodOverrides": {
            "empty": true,
            "overrides": []
        },
        "nonPublicAccessAllowed": true,
        "primary": false,
        "propertyValues": {
            "converted": false,
            "empty": true,
            "propertyValueList": [],
            "propertyValues": []
        },
        "prototype": false,
        "qualifiers": [],
        "resolvedAutowireMode": 0,
        "role": 0,
        "scope": "singleton",
        "singleton": true,
        "synthetic": false
    }
}
```