---
title: 'SpringBoot 2 中使用 Quartz'
date: 2018-07-26 11:30:23
tags:
    - Java
    - SpringBoot 2.0
    - Quartz
categories:
    - 技术笔记
---

这几天在写SpringBoot项目，突然把SpringBoot从1.x更新到了2.x出了一堆的问题，本文讲一下Quartz升级版本后的配置

<!-- more -->

## 引入依赖

在 SpringBoot 1.x 的时候引入依赖是这样的： 

```xml
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>${quartz.version}</version>
</dependency>
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>${quartz.version}</version>
</dependency>
```

SpringBoot 2.x 改成了这样

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-quartz</artifactId>
</dependency>
```
改成这样的后果就是，不用自己用代码配置了。省去了配置各种东西的时间（妈耶，前两天白弄了）

## 配置文件变化

在 SpringBoot 1.x 的时候使用的是  `org.quartz.xx` , 还要自己放到文件`quartz.properties`里面去，再引用

```ini
org.quartz.scheduler.instanceId = AUTO
org.quartz.scheduler.instanceName = DefaultQuartzScheduler
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount = 50
org.quartz.threadPool.threadPriority = 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread = true
org.quartz.jobStore.isClustered = true
org.quartz.jobStore.misfireThreshold = 60000
org.quartz.jobStore.clusterCheckinInterval = 20000
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.StdJDBCDelegate
org.quartz.jobStore.selectWithLockSQL = SELECT * FROM {0}LOCKS WHERE LOCK_NAME = ? FOR UPDATE
org.quartz.jobStore.tablePrefix = QRTZ_
```

吐血三升。在SpringBoot2.x中配置文件和配置项都变了，改到 `application.properties` 里面，配置项的前缀和属性也变了很多：

```ini
spring.quartz.properties.org.quartz.scheduler.instanceName=clusteredScheduler
spring.quartz.properties.org.quartz.scheduler.instanceId=AUTO
spring.quartz.properties.org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
spring.quartz.properties.org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
spring.quartz.properties.org.quartz.jobStore.tablePrefix=QRTZ_
spring.quartz.properties.org.quartz.jobStore.isClustered=true
spring.quartz.properties.org.quartz.jobStore.clusterCheckinInterval=10000
spring.quartz.properties.org.quartz.jobStore.useProperties=false
spring.quartz.properties.org.quartz.threadPool.class=org.quartz.simpl.SimpleThreadPool
spring.quartz.properties.org.quartz.threadPool.threadCount=10
spring.quartz.properties.org.quartz.threadPool.threadPriority=5
spring.quartz.properties.org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread=true
spring.quartz.job-store-type=jdbc
```

这样的话就自动集成了 Quartz 功能。

> 讲道理配置用 yaml 感觉更加好？

```yml
spring:
    quartz:
        job-store-type: jdbc
        properties:
            org:
                quartz:
                    jobStore:
                        class: org.quartz.impl.jdbcjobstore.JobStoreTX
                        clusterCheckinInterval: 10000
                        driverDelegateClass: org.quartz.impl.jdbcjobstore.StdJDBCDelegate
                        isClustered: true
                        tablePrefix: QRTZ_
                        useProperties: false
                    scheduler:
                        instanceId: AUTO
                        instanceName: clusteredScheduler
                    threadPool:
                        class: org.quartz.simpl.SimpleThreadPool
                        threadCount: 10
                        threadPriority: 5
                        threadsInheritContextClassLoaderOfInitializingThread: true
```

配置文件装换工具：http://www.toyaml.com/ （提供 properties 与 yaml 的互相装换）