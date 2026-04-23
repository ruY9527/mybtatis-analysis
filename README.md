# MyBatis 源码分析

![MyBatis](https://img.shields.io/badge/MyBatis-3.5.4-orange.svg)
![Java](https://img.shields.io/badge/Java-1.8-blue.svg)
![Maven](https://img.shields.io/badge/Maven-build-red.svg)
![License](https://img.shields.io/badge/License-MIT-green.svg)

> MyBatis 源码深度分析与学习项目，从 HelloWorld 到 SpringBoot 整合，全面解析 MyBatis 核心原理。

---

## 📖 项目简介

本项目是对 MyBatis 框架源码的系统化学习和分析记录，通过循序渐进的方式，从基础使用到高级整合，再到插件扩展，逐步深入理解 MyBatis 的核心设计思想和实现原理。

## 📚 学习路线

建议按以下顺序阅读学习：

```
┌─────────────────────────────────────────────────────────────────────┐
│                        MyBatis 学习路线图                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│   1️⃣  HelloWorld 与执行流程                                          │
│       ↓   [mybatis-work-flow]                                       │
│                                                                     │
│   2️⃣  配置文件解析                                                    │
│       ↓   [mybatis-xml-config]                                      │
│                                                                     │
│   3️⃣  Mapper 文件解析                                                 │
│       ↓   [mybatis-mapper-xml]                                      │
│                                                                     │
│   4️⃣  缓存机制（一级/二级缓存）                                         │
│       ↓   [mybatis-cache-analysis]                                  │
│                                                                     │
│   5️⃣  Spring 整合                                                    │
│       ↓   [mybatis-spring-hello]                                    │
│                                                                     │
│   6️⃣  SpringBoot 整合                                                │
│       ↓   [mybatis-spring-boot-hello]                               │
│                                                                     │
│   7️⃣  分页插件 PageHelper                                             │
│       ↓   [mybatis-pagehelper-analysis]                             │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## 📁 项目结构

```
mybatis-analysis/
│
├── mybatis-work-flow/           # MyBatis HelloWorld 与执行流程分析
│   └── 基础入门，整体执行流程梳理
│
├── mybatis-xml-config/          # mybatis-config.xml 配置文件解析
│   └── 各标签解析过程分析（properties/settings/plugins等）
│
├── mybatis-mapper-xml/          # Mapper XML 文件解析分析
│   └── resultMap/sql/动态SQL解析
│
├── mybatis-cache-analysis/      # 缓存机制深度分析
│   ├── 一级缓存（SqlSession级别）
│   └── 二级缓存（namespace级别）
│
├── mybatis-spring-hello/        # MyBatis + Spring 整合分析
│   └── SqlSessionFactoryBean、MapperScannerConfigurer 源码分析
│
├── mybatis-spring-boot-hello/   # MyBatis + SpringBoot 整合分析
│   └── 自动配置原理、@MapperScan 注解分析
│
└── mybatis-pagehelper-analysis/ # PageHelper 分页插件分析
    └── 插件机制、Interceptor 接口实现
```

## 🔍 模块详解

### 1. [mybatis-work-flow](./mybatis-work-flow) - HelloWorld 与执行流程

**核心内容：**
- MyBatis 基础 HelloWorld 示例
- SqlSessionFactory 构建过程
- SqlSession 获取与 Mapper 代理生成
- SQL 执行流程完整分析

**关键源码路径：**
- `Resources.getResourceAsStream()` - 配置文件加载
- `SqlSessionFactoryBuilder.build()` - 工厂构建
- `DefaultSqlSession` - 会话实现
- `MapperProxy` - 动态代理

---

### 2. [mybatis-xml-config](./mybatis-xml-config) - 配置文件解析

**核心内容：**
- `<properties>` 标签解析
- `<settings>` 配置项解析
- `<typeAliases>` 别名注册
- `<plugins>` 插件加载
- `<environments>` 环境配置
- `<mappers>` Mapper 扫描

**关键类：**
- `XMLConfigBuilder` - 配置解析器
- `Configuration` - 全局配置对象
- `TypeAliasRegistry` - 别名注册中心

---

### 3. [mybatis-mapper-xml](./mybatis-mapper-xml) - Mapper 文件解析

**核心内容：**
- `<resultMap>` 结果映射解析
- `<sql>` SQL 片段解析
- `select|insert|update|delete` 语句解析
- 动态 SQL 解析（DynamicSqlSource vs RawSqlSource）
- `#{}` 参数占位符处理

**关键类：**
- `XMLMapperBuilder` - Mapper 解析器
- `MappedStatement` - SQL 语句封装
- `SqlSource` - SQL 来源接口

---

### 4. [mybatis-cache-analysis](./mybatis-cache-analysis) - 缓存机制

**核心内容：**
- **一级缓存**：SqlSession 级别，基于 HashMap 实现
- **二级缓存**：namespace 级别，基于装饰器模式
- 缓存失效条件分析
- 多 SqlSession 脏数据问题
- 缓存执行流程（先二级后一级再数据库）

**关键类：**
- `BaseExecutor.localCache` - 一级缓存
- `CachingExecutor` - 二级缓存执行器
- `TransactionalCache` - 事务缓存装饰器
- `PerpetualCache` - 永久缓存实现

---

### 5. [mybatis-spring-hello](./mybatis-spring-hello) - Spring 整合

**核心内容：**
- `SqlSessionFactoryBean` 源码分析
- `MapperScannerConfigurer` 扫描注册
- Spring 扩展点运用（FactoryBean、InitializingBean等）
- BeanDefinition 注册与修改
- Mapper 接口代理注入 Spring 容器

**关键类：**
- `SqlSessionFactoryBean` - 工厂 Bean
- `MapperScannerConfigurer` - 扫描配置器
- `MapperFactoryBean` - Mapper 工厂 Bean

---

### 6. [mybatis-spring-boot-hello](./mybatis-spring-boot-hello) - SpringBoot 整合

**核心内容：**
- SpringBoot 自动配置原理
- `@MapperScan` 注解解析
- `MybatisAutoConfiguration` 自动配置类
- `spring.factories` SPI 机制
- SqlSessionTemplate 实现

**关键类：**
- `MapperScannerRegistrar` - 注册器
- `MybatisAutoConfiguration` - 自动配置
- `MybatisProperties` - 配置属性绑定

---

### 7. [mybatis-pagehelper-analysis](./mybatis-pagehelper-analysis) - PageHelper 插件

**核心内容：**
- MyBatis 插件机制（Interceptor 接口）
- `@Intercepts` 注解签名
- `PageInterceptor` 拦截实现
- 分页 SQL 自动改写
- count 查询优化

**关键类：**
- `PageInterceptor` - 拦截器实现
- `PageHelper` - 分页助手
- `Dialect` - 数据库方言抽象

---

## 🛠️ 技术栈

| 技术 | 版本 | 说明 |
|-----|------|-----|
| MyBatis | 3.5.4 | 核心框架 |
| MyBatis-Spring | 2.0.3+ | Spring 整合包 |
| Spring | 5.x | Spring Framework |
| SpringBoot | 2.x | SpringBoot |
| PageHelper | 5.x | 分页插件 |
| MySQL | 5.7+ | 数据库 |
| Java | 1.8 | JDK 版本 |

---

## 💡 核心设计模式

项目中涉及的设计模式总结：

| 设计模式 | 应用场景 | 相关类 |
|---------|---------|--------|
| **工厂模式** | SqlSessionFactory 创建 | `SqlSessionFactoryBuilder` |
| **建造者模式** | Configuration 构建 | `XMLConfigBuilder`、`BaseBuilder` |
| **代理模式** | Mapper 接口实现 | `MapperProxy`、`MapperProxyFactory` |
| **装饰器模式** | 二级缓存增强 | `CachingExecutor`、`TransactionalCache` |
| **责任链模式** | 插件拦截链 | `InterceptorChain`、`Plugin` |
| **模板方法模式** | Executor 执行流程 | `BaseExecutor` |

---

## 🚀 快速开始

```bash
# 克隆项目
git clone https://github.com/baoyang23/mybatis-analysis.git

# 进入项目目录
cd mybatis-analysis

# 选择感兴趣的模块，导入 IDE 进行学习
```

---

## 📝 学习建议

1. **循序渐进**：按照学习路线图顺序阅读，从基础到高级
2. **Debug 跟踪**：在每个模块中设置断点，跟踪源码执行流程
3. **动手实践**：建议自己创建测试项目，验证分析结论
4. **对比思考**：对比不同整合方式的实现差异，理解设计思想
5. **扩展学习**：尝试自定义插件，深入理解插件机制

---

## 📮 联系方式

欢迎技术交流与问题讨论：

- **微信公众号**：深文笔记
- **微信**：l18776416225（请备注：技术交流）

---

## 📜 题记

> 这里记录的是我阅读 MyBatis 源码以及分析代码的心得体会。
> 
> 愿我们学的每一样技术以及框架，都能够有钻研的精神，能够阅读下去。
> 
> 最后，坚持学习下去，励志成为最好的自我。

---

## ⭐ Star History

如果这个项目对你有帮助，欢迎 Star ⭐ 支持！

---

## License

[MIT License](LICENSE)