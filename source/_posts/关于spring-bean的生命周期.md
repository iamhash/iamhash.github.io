---
title: 关于spring bean的生命周期
date: 2025-09-08 19:58:01
tags: [spring,技术碎片]
---
## 背景
spring三大核心思想：

- IOC（控制反转）：在程序中手动创建对象的控制权交给spring框架来管理。实现：容器管理Bean生命周期

  <!-- more -->

- DI（依赖注入）：依赖关系的硬编码问题。实现：Setter/构造器/注解注入

- AOP：将与业务无关却被业务模块所共同调用的逻辑封装起来。实现：动态代理与切面配置

IOC是一种设计思想，并非具体技术实现。原本在程序中手动创建对象的控制权交给spring框架来管理。

控制：指的是对象创建（实例化、管理）的权力
反转：控制权交给外部环境（Spring 框架、IoC 容器）

在 Spring 中， IoC 容器是 Spring 用来实现 IoC 的载体， IoC 容器实际上就是个 Map（key，value），Map 中存放的是各种对象。

**Bean：IoC 容器所管理的对象**

而下面会详细讲解IOC中的Bean是如何从出生到消亡的。

## 核心步骤

### 1.创建 Bean 的实例：

Bean 容器首先会找到**配置（XML、注解、Java Config）中的 Bean** 定义，然后使用 Java 反射 API 来创建 Bean 的实例。

### **2.Bean 属性赋值/填充：**

为 Bean 设置**相关属性和依赖**，例如@Autowired 等注解注入的对象、@Value 注入的值、setter方法或构造函数注入依赖和值、@Resource注入的各种资源。这一步就是把依赖关系补齐，让 Bean 准备好工作。

### **3.Bean 初始化：**

**（1）Aware接口回调：**（Aware 接口能让 Bean 能拿到 Spring 容器资源—比如容器上下文，让bean感知并操作spring容器资源）

如果 Bean 实现了 BeanNameAware 接口，调用 setBeanName()方法，获取当前 Bean 在容器中的名字。
如果 Bean 实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader()方法，传入 ClassLoader对象的实例。
如果 Bean 实现了 BeanFactoryAware 接口，调用 setBeanFactory()方法，传入 BeanFactory对象的实例。
与上面的类似，如果实现了其他 *.Aware接口，就调用相应的方法。

ApplicationContextAware——获取 ApplicationContext，上下文中所有 Bean、环境等资源。

**作用：让 Bean 能感知到 Spring 容器的一些内部信息或资源。**

**（2）BeanPostProcessor 前置处理（初始化前增强）**

如果有和加载这个 Bean 的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessBeforeInitialization() 方法

作用：例如：

对 Bean 的属性进行统一修改（给某个 Bean 的某个字段赋默认值）

检查 Bean 的合法性

提前代理一些特殊 Bean 

**不是 AOP 本身，但 AOP 是基于这个扩展点实现的。**
**Spring AOP（比如基于代理的事务管理）就是在 postProcessAfterInitialization 阶段，把原始 Bean 替换成了代理 Bean。**

Spring 内部就大量用到 BeanPostProcessor，例如：

AutowiredAnnotationBeanPostProcessor → 负责 @Autowired 注入

CommonAnnotationBeanPostProcessor → 负责 @PostConstruct 和 @PreDestroy

ApplicationContextAwareProcessor → 处理 *Aware 接口

AOP 的 AnnotationAwareAspectJAutoProxyCreator → 生成动态代理

**（3）执行初始化方法**

如果 Bean 实现了**InitializingBean接口**，执行afterPropertiesSet()方法。
如果 Bean 在配置文件中的定义包含 **init-method 属性**，执行指定的方法。

【通常用于打开数据库连接、初始化缓存和数据资源、启动线程池等操作】

执行自定义初始化逻辑：@PostConstruct、afterPropertiesSet、init-method，这里三个可能的执行途径：

@PostConstruct 注解的方法
Spring 在初始化前调用这个方法，用于开发者定义初始化逻辑。

实现 InitializingBean 接口的 afterPropertiesSet() 方法
Spring 调用这个接口的方法。

init-method / @Bean(initMethod=...) 指定的方法
在配置文件或注解中明确指定的初始化方法。

**（4）BeanPostProcessor 后置处理（初始化后增强）**

如果有和加载这个 Bean 的 Spring 容器相关的 BeanPostProcessor 对象，执行postProcessAfterInitialization() 方法。

允许开发者或 Spring 内部组件对 Bean 做“初始化后”的增强，比如 AOP 代理替换。

在 Bean 完全初始化好后，Spring 再次对它进行处理（常见于 AOP，动态代理等）

**4.销毁 Bean 阶段（容器关闭或 Bean 移除时）**

销毁 Bean：销毁并不是说要立马把 Bean 给销毁掉，**而是把 Bean 的销毁方法先记录下来，将来需要销毁 Bean 或者销毁容器的时候，就调用这些方法去释放 Bean 所持有的资源。**

如果 Bean 实现了 DisposableBean 接口，执行 destroy() 方法。
如果 Bean 在配置文件中的定义包含 destroy-method 属性，执行指定的 Bean 销毁方法。或者，也可以直接通过@PreDestroy 注解标记 Bean 销毁之前执行的方法。【通常用于释放资源，比如：关闭数据库连接、停止线程池、清理缓存】

**整体流程**：1，实例化（利用反射API创建bean实例）——>2.依赖注入，属性赋值——>3.初始化（资源注入—前置处理—初始化方法—后置处理）——>4.销毁（注册销毁方法，容器关闭时触发）

## 表格整理

| 步骤                           | 发生时机                  | 作用/意义                                                    |
| ------------------------------ | ------------------------- | ------------------------------------------------------------ |
| **实例化**                     | Spring 容器创建 Bean 对象 | new 出一个对象，Bean 还没注入依赖。                          |
| **属性填充**                   | 实例化后                  | 依赖注入，把其他 Bean、配置值等注入进来。                    |
| **Aware 接口回调**             | 依赖注入完成后            | 让 Bean 知道它的名字、容器引用、环境等，方便和容器交互。     |
| **BeanPostProcessor 前置处理** | 初始化方法调用前          | 允许开发者或 Spring 内部组件对 Bean 做“初始化前”的增强。     |
| **初始化回调**                 | 前置处理完成后            | 执行自定义初始化逻辑：`@PostConstruct`、`afterPropertiesSet`、`init-method`。 |
| **BeanPostProcessor 后置处理** | 初始化方法调用后          | 允许开发者或 Spring 内部组件对 Bean 做“初始化后”的增强，比如 AOP 代理替换。 |
| **Bean 可用**                  | 初始化完毕                | Bean 正式交给应用使用。                                      |
| **销毁回调**                   | 容器关闭 / 作用域结束时   | 执行销毁逻辑，释放资源，调用 `@PreDestroy`、`destroy-method`、`DisposableBean.destroy()`。 |