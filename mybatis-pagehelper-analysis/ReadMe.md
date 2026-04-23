# MyBatis PageHelper 分页插件分析

![MyBatis](https://img.shields.io/badge/MyBatis-3.5.4-orange.svg)
![PageHelper](https://img.shields.io/badge/PageHelper-5.1.10-blue.svg)
![Java](https://img.shields.io/badge/Java-1.8-blue.svg)
![Status](https://img.shields.io/badge/Status-Completed-green.svg)

> [返回主目录](../README.md) | [上一模块：SpringBoot整合](../mybatis-spring-boot-hello)

---

## 📖 模块简介

本模块分析 PageHelper 分页插件的实现原理，通过 MyBatis 插件机制（Interceptor接口）拦截查询语句，自动实现分页功能。

## 🔑 核心知识点

| 概念 | 说明 |
|-----|------|
| **插件机制** | MyBatis 通过 Interceptor 接口提供扩展能力 |
| **拦截签名** | `@Intercepts` + `@Signature` 定义拦截目标 |
| **方言抽象** | `Dialect` 接口支持多种数据库 |
| **ThreadLocal** | `PageHelper` 使用 ThreadLocal 存储分页参数 |

## 🔗 核心源码路径

| 类 | 路径 | 说明 |
|----|------|------|
| `PageInterceptor` | `com.github.pagehelper.PageInterceptor` | 拦截器核心实现 |
| `PageHelper` | `com.github.pagehelper.PageHelper` | 分页参数设置入口 |
| `Dialect` | `com.github.pagehelper.Dialect` | 数据库方言抽象 |
| `ExecutorUtil` | `com.github.pagehelper.util.ExecutorUtil` | 分页SQL执行工具 |

## 🎯 插件配置示例

```xml
<plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <property name="offsetAsPageNum" value="true" />
        <property name="rowBoundsWithCount" value="true" />
        <property name="pageSizeZero" value="true" />
        <property name="reasonable" value="false" />
    </plugin>
</plugins>
```

---

##          							MyBatis 分页插件阅读



#### 题记

​     MyBatis 分页插件  pagehelper 研究. 在最初学习 mybatis 的时候，就有对该插件进行研究学习.  这次整理在过一遍 MyBatis 整体的知识，再对整体以及整合一些其他知识进行学习并且研究。  这里是有研究到 分页插件，其实分页插件使用起来和理解起来还是很容易，并不是特别的复杂.



#### 上手使用

​     如果使用过了的朋友，可以跳过这个环节，如果没有使用的话，那就可以来来看看大致的使用方法.  这里是配合 MyBatis 来单独使用的一个简单的 demo .

​     项目地址 :    https://github.com/baoyang23/mybtatis-analysis/tree/master/mybatis-pagehelper-analysis

  可以 clone 或者下载下来看这个项目，这里我就不一一的详细说明了. 

但是还是要说下 怎么讲 分页插件给引入到项目里面来.

我们只需再 plugins 里面添加相应的 plugin 即可.  同时我们添加进来的 PageInterceptor 也是一个全限定类名，所以我们就可以 debug 到这个类里面来看.

```xml
<plugins>
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!--<property name="dialect" value="mysql" />-->
        <!-- 该参数默认为false -->
        <!-- 设置为true时，会将RowBounds第一个参数offset当成pageNum页码使用 -->
        <!-- 和startPage中的pageNum效果一样 -->
        <property name="offsetAsPageNum" value="true" />
        <!-- 该参数默认为false -->
        <!-- 设置为true时，使用RowBounds分页会进行count查询 -->
        <property name="rowBoundsWithCount" value="true" />
        <!-- 设置为true时，如果pageSize=0或者RowBounds.limit = 0就会查询出全部的结果 -->
        <!-- （相当于没有执行分页查询，但是返回结果仍然是Page类型） -->
        <property name="pageSizeZero" value="true" />
        <!-- 3.3.0版本可用 - 分页参数合理化，默认false禁用 -->
        <!-- 启用合理化时，如果pageNum<1会查询第一页，如果pageNum>pages会查询最后一页 -->
        <!-- 禁用合理化时，如果pageNum<1或pageNum>pages会返回空数据 -->
        <property name="reasonable" value="false" />
        <!-- 3.5.0版本可用 - 为了支持startPage(Object params)方法 -->
        <!-- 增加了一个`params`参数来配置参数映射，用于从Map或ServletRequest中取值 -->
        <!-- 可以配置pageNum,pageSize,count,pageSizeZero,reasonable,不配置映射的用默认值 -->
        <!-- 不理解该含义的前提下，不要随便复制该配置 -->
        <property name="params" value="pageNum=start;pageSize=limit;" />
        <!-- always总是返回PageInfo类型,check检查返回类型是否为PageInfo,none返回Page -->
        <property name="returnPageInfo" value="check" />
    </plugin>
</plugins>
```



使用的话就参考 :  com.iyang.mybatis.PageHelperInit  这个类来看下简单的使用.



#### com.github.pagehelper.PageInterceptor  阅读 & 分析

可以看到这个类实现了  MyBatis 的接口，并且看到类上的注解，这里可以参考官网所给出来的例子。

```java
@Intercepts(
        {
                @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
                @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
        }
)
public class PageInterceptor implements Interceptor { 
}
```



这里我们依然采用 debug 的方式来对其方法进行阅读.  



**com.github.pagehelper.PageInterceptor#setProperties**

   可以看到先加载是 setProperties 方法。 从堆栈信息来看, org.apache.ibatis.builder.xml.XMLConfigBuilder#parseConfiguration 走的 this.pluginElement(root.evalNode("plugins")) 方法， 最后从 org.apache.ibatis.builder.xml.XMLConfigBuilder#pluginElement 中的 interceptorInstance.setProperties(properties) 调用的 setProperties 方法，也就是走到我们的 debug 中来.



最后来看，我们的分页插件包里面的类调用的 setProperties 方法. 传入进来的 properties 就是我们在 xml 里面配置的属性.

```java
@Override
public void setProperties(Properties properties) {
    //缓存 count ms
// com.github.pagehelper.cache.SimpleCache 这里创建的是一个简单的缓存 Map.    
    msCountMap = CacheFactory.createCache(properties.getProperty("msCountCache"), "ms", properties);
    
// 如果没有的话，就设置默认的 com.github.pagehelper.PageHelper     
    String dialectClass = properties.getProperty("dialect");
    if (StringUtil.isEmpty(dialectClass)) {
        dialectClass = default_dialect_class;
    }
 
 // 实例化 PageHelper 类并且调用其setProperties 方法,将properties属性给注入进去.   
    try {
        Class<?> aClass = Class.forName(dialectClass);
        dialect = (Dialect) aClass.newInstance();
    } catch (Exception e) {
        throw new PageException(e);
    }
    dialect.setProperties(properties);

// 如果有配置 countSuffix 的话，就会赋值进去.    
    String countSuffix = properties.getProperty("countSuffix");
    if (StringUtil.isNotEmpty(countSuffix)) {
        this.countSuffix = countSuffix;
    }
}
```

可以看到，这里先是对一些属性进行判断，然后如果有配置 dialect 的话，就实例化配置的类，如果没有配置的话，就会实例化默认的类. com.github.pagehelper.PageHelper 这就是默认的.



**com.github.pagehelper.PageInterceptor#plugin 方法**



org.apache.ibatis.plugin.InterceptorChain#pluginAll  可以看到这里会对所有的拦截器以及并实现类，然后调用其 plugin 方法.

```
public Object pluginAll(Object target) {
  for (Interceptor interceptor : interceptors) {
    target = interceptor.plugin(target);
  }
  return target;
}
```





**com.github.pagehelper.PageInterceptor#intercept 方法**

 从堆栈信息来看, 是从我们调用的 mapper 查询方法走到这里来的.


org.apache.ibatis.plugin.Plugin#invoke   --->    com.github.pagehelper.PageInterceptor#intercept 



```java
// 这里传入进来的invocation对象，里面含有  target 是 CachingExecutor,method方法是Executor.query方法.
// args 是传入进来的参数
@Override
public Object intercept(Invocation invocation) throws Throwable {
    try {
 // 获取参数       
        Object[] args = invocation.getArgs();
 // 根据下标分别获取对应的信息来.       
        MappedStatement ms = (MappedStatement) args[0];
        Object parameter = args[1];
        RowBounds rowBounds = (RowBounds) args[2];
        ResultHandler resultHandler = (ResultHandler) args[3];
        Executor executor = (Executor) invocation.getTarget();
        CacheKey cacheKey;
        BoundSql boundSql;
        //由于逻辑关系，只会进入一次
        if (args.length == 4) {
            //4 个参数时
            // 获取 sql 信息
            boundSql = ms.getBoundSql(parameter);
            // 创建缓存cacheKey  
            cacheKey = executor.createCacheKey(ms, parameter, rowBounds, boundSql);
        } else {
            //6 个参数时
            cacheKey = (CacheKey) args[4];
            boundSql = (BoundSql) args[5];
        }
        
// 再次检查 com.github.pagehelper.Dialect 是不是有实例化        
        checkDialectExists();

        List resultList;
        //调用方法判断是否需要进行分页，如果不需要，直接返回结果
// com.github.pagehelper.PageHelper#skip 走这里来, 返回的是false的话，就会进行分页查询.      
        if (!dialect.skip(ms, parameter, rowBounds)) {
            //判断是否需要进行 count 查询
// 这里判断是不是需要 count 查询, count 查询完了后,判断是不是还要分页查询. 
// 最后都会走 afterAll 方法,            
            if (dialect.beforeCount(ms, parameter, rowBounds)) {
                //查询总数
                Long count = count(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                //处理查询总数，返回 true 时继续分页查询，false 时直接返回
                if (!dialect.afterCount(count, parameter, rowBounds)) {
                    //当查询总数为 0 时，直接返回空的结果
                    return dialect.afterPage(new ArrayList(), parameter, rowBounds);
                }
            }
// com.github.pagehelper.util.ExecutorUtil#pageQuery,            
            resultList = ExecutorUtil.pageQuery(dialect, executor,
                    ms, parameter, rowBounds, resultHandler, boundSql, cacheKey);
        } else {
            //rowBounds用参数值，不使用分页插件处理时，仍然支持默认的内存分页
            resultList = executor.query(ms, parameter, rowBounds, resultHandler, cacheKey, boundSql);
        }
// 该方法可以看到,是对 Page 来进行参数设置等操作.        
        return dialect.afterPage(resultList, parameter, rowBounds);
    } finally {
        if(dialect != null){
// 这里对  com.github.pagehelper.page.PageAutoDialect#dialectThreadLocal
//  com.github.pagehelper.page.PageMethod#LOCAL_PAGE 中的 ThreadLocal 进行 remove,避免不清除的话会存在内存泄露的问题,最后导致出内存溢出的问题.            
            dialect.afterAll();
        }
    }
}
```



**com.github.pagehelper.PageHelper#skip 方法** 

这里走到的是我们之前实例化的 PageHelper 类来.

```java
@Override
public boolean skip(MappedStatement ms, Object parameterObject, RowBounds rowBounds) {
    if (ms.getId().endsWith(MSUtils.COUNT)) {
        throw new RuntimeException("在系统中发现了多个分页插件，请检查系统配置!");
    }
    Page page = pageParams.getPage(parameterObject, rowBounds);
    if (page == null) {
        return true;
    } else {
        //设置默认的 count 列
        if (StringUtil.isEmpty(page.getCountColumn())) {
            page.setCountColumn(pageParams.getCountColumn());
        }
        autoDialect.initDelegateDialect(ms);
        return false;
    }
}
```



**com.github.pagehelper.util.ExecutorUtil#pageQuery 方法**

从名字可以了解为 分页查询.

```java
public static  <E> List<E> pageQuery(Dialect dialect, Executor executor, MappedStatement ms, Object parameter,
                             RowBounds rowBounds, ResultHandler resultHandler,
                             BoundSql boundSql, CacheKey cacheKey) throws SQLException {
    //判断是否需要进行分页查询
    if (dialect.beforePage(ms, parameter, rowBounds)) {
        //生成分页的缓存 key
        CacheKey pageKey = cacheKey;
        //处理参数对象
        parameter = dialect.processParameterObject(ms, parameter, boundSql, pageKey);
        //调用方言获取分页 sql
        String pageSql = dialect.getPageSql(ms, boundSql, parameter, rowBounds, pageKey);
        BoundSql pageBoundSql = new BoundSql(ms.getConfiguration(), pageSql, boundSql.getParameterMappings(), parameter);

        Map<String, Object> additionalParameters = getAdditionalParameter(boundSql);
        //设置动态参数
        for (String key : additionalParameters.keySet()) {
            pageBoundSql.setAdditionalParameter(key, additionalParameters.get(key));
        }
        //执行分页查询
        return executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, pageKey, pageBoundSql);
    } else {
        //不执行分页的情况下，也不执行内存分页
        return executor.query(ms, parameter, RowBounds.DEFAULT, resultHandler, cacheKey, boundSql);
    }
}
```

其实从 PageHelper 这个项目来看，代码里面的注解就已经让你一看就很明白了.



#### 理解

​		可以看到  com.github.pagehelper.Dialect 下面是有很多子类，就是为了兼容不同的数据库类型. com.github.pagehelper.dialect.helper 从该包下可以看到不同类型的类.

​        在 com.github.pagehelper.cache 包下，也是有 CacheFactory 类，缓存工厂，然后由工厂来生成缓存. 

 虽然只是一个简单的插件，但是还是可以看到，从作者的角度已经我们阅读里理解，不难可以看到作者是想兼容到许多数据库的.   并且还采用设计模式来便于阅读和理解。



####  总结

​     其实这个插件，我们在最初将 work-flow 的时候，就已经对插件做过类似的扩展说明. 

​     于是我们这里只是在 mybatis 的配置文件中, 配置了 com.github.pagehelper.PageInterceptor 类，然后该类在加载配置的时候或者后续方法中，会有对 com.github.pagehelper.PageHelper 初始化.  最后也是利用 com.github.pagehelper.PageHelper 中的方法来进行判断等操作.

​     并且这个插件也是国内的人开源出来的，可以看到其中文的注释，还是蛮详细的，阅读起来也不是很麻烦，整体感觉还是很好的.





