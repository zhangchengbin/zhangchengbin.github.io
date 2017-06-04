---
layout: post
title:  Spring+Mybatis之多数据源配置
date:   2017-6-4 11:10:28 +0800
categories: spring
tag: 
---

* content
{:toc}


### 1.配置多个数据源(采用连接池c3p0，直接jdbc连也行)
#### 数据源1：dataSource

```xml
<!--1.配置连接池属性-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="#{jdbc['mysql.driver']}"/>
    <property name="jdbcUrl" value="#{jdbc['jdbc.url']}"/>
    <property name="user" value="#{jdbc['jdbc.username']}"/>
    <property name="password" value="#{jdbc['jdbc.password']}"/>
    <property name="maxPoolSize" value="30"/>
    <property name="minPoolSize" value="10"/>
    <!--关闭连接后不自动commit-->
    <property name="autoCommitOnClose" value="false"/>
    <!--获取连接超时时间-->
    <property name="checkoutTimeout" value="10000"/>
    <!--获取连接重试次数-->
    <property name="acquireRetryAttempts" value="3"/>
</bean>
```  

#### 数据源2：dataSource2


```xml
<!--1.材料 连接池属性-->
<bean id="materialDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="#{jdbc['material.mysql.driver']}"/>
    <property name="jdbcUrl" value="#{jdbc['material.jdbc.url']}"/>
    <property name="user" value="#{jdbc['material.jdbc.username']}"/>
    <property name="password" value="#{jdbc['material.jdbc.password']}"/>
    <property name="maxPoolSize" value="30"/>
    <property name="minPoolSize" value="10"/>
    <!--关闭连接后不自动commit-->
    <property name="autoCommitOnClose" value="false"/>
    <!--获取连接超时时间-->
    <property name="checkoutTimeout" value="10000"/>
    <!--获取连接重试次数-->
    <property name="acquireRetryAttempts" value="3"/>
</bean>
```  

### 2, 配置mybatis数据会话管理（sqlSessionFactoryBean）
#### 会话factory1：sqlSessionFactory

```xml
<!--2.配置SqlSessionFactory对象-->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--注入数据库连接池-->
    <property name="dataSource" ref="dataSource"/>
    <!--配置mybatis全局配置文件:mybatis-config.xml-->
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <!--扫描entity包,使用别名,多个用;隔开-->
    <property name="typeAliasesPackage" value="com.huixin.entity;com.huixin.dto"/>
    <!--扫描sql配置文件:mapper需要的xml文件-->
    <property name="mapperLocations" value="classpath:mapper/*.xml"/>
</bean>
```
#### 会话factory2：materialSqlSessionFactory

```xml
<bean id="materialSqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!--注入数据库连接池-->
    <property name="dataSource" ref="materialDataSource"/>
    <!--配置mybatis全局配置文件:mybatis-config.xml-->
    <property name="configLocation" value="classpath:mybatis-config.xml"/>
    <!--扫描entity包,使用别名,多个用;隔开-->
    <property name="typeAliasesPackage" value="com.huixin.material.entity"/>
    <!--扫描sql配置文件:mapper需要的xml文件-->
    <property name="mapperLocations" value="classpath:materialMapper/*.xml"/>
</bean>
```
### 3， 配置Dao接口层（==重点注意==）

> mybatis用mapperScannerConfigurer扫描相关的dao(或mapper)的interface的包，将其注入到spring进行管理；
> 
> dao层需要注入第2步配置的sessionFactory来获得session(session中加入事物等操作)，跟sessionFactory中配置的*Mapper.xml进行一一匹配；

#### mapperScanner1:

```xml
<!--3:配置扫描Dao接口包,动态实现DAO接口,注入到spring容器-->
<bean id="mapperScaner1" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--注入SqlSessionFactory-->
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    <!-- 给出需要扫描的Dao接口-->
    <property name="basePackage" value="com.huixin.dao"/>
</bean>

```
> ==不同数据源的扫描dao层不要放在一起，不然数据源1扫描了数据源2的dao，那 去数据源1的mappe.xml中匹配时，会找不到相应方法，而报错！！！==

#### mapperScanner2:

```xml
<bean id="mapperScaner2" class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <!--注入SqlSessionFactory-->
    <property name="sqlSessionFactoryBeanName" value="materialSqlSessionFactory"/>
    <!-- 给出需要扫描的Dao接口-->
    <property name="basePackage" value="com.huixin.material.dao"/>
</bean>

```
