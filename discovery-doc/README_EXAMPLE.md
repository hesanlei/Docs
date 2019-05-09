# Nepxion Discovery
[![Total lines](https://tokei.rs/b1/github/Nepxion/Discovery?category=lines)](https://github.com/Nepxion/Discovery)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg?label=license)](https://github.com/Nepxion/Discovery/blob/master/LICENSE)
[![Maven Central](https://img.shields.io/maven-central/v/com.nepxion/discovery.svg?label=maven%20central)](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22com.nepxion%22%20AND%20discovery)
[![Javadocs](http://www.javadoc.io/badge/com.nepxion/discovery-plugin-framework.svg)](http://www.javadoc.io/doc/com.nepxion/discovery-plugin-framework)
[![Build Status](https://travis-ci.org/Nepxion/Discovery.svg?branch=master)](https://travis-ci.org/Nepxion/Discovery)
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/8e39a24e1be740c58b83fb81763ba317)](https://www.codacy.com/project/HaojunRen/Discovery/dashboard?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=Nepxion/Discovery&amp;utm_campaign=Badge_Grade_Dashboard)

## 示例演示
本例将模拟一个较为复杂的场景，如下图

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Version.jpg)

## 目录
- [场景描述](#场景描述)
- [引入依赖](#引入依赖)
- [服务注册过滤的操作演示](#服务注册过滤的操作演示)
  - [黑/白名单的IP地址注册的过滤](#黑/白名单的IP地址注册的过滤)
  - [最大注册数的限制的过滤](#最大注册数的限制的过滤)
  - [黑/白名单的IP地址发现的过滤](#黑/白名单的IP地址发现的过滤)
- [服务发现和负载均衡控制的操作演示](#服务发现和负载均衡控制的操作演示)
  - [基于图形化桌面程序的灰度发布](#基于图形化桌面程序的灰度发布)
  - [基于图形化Web程序的灰度发布](#基于图形化Web程序的灰度发布)  
  - [基于Apollo界面的灰度发布](#基于Apollo界面的灰度发布)
  - [基于Nacos界面的灰度发布](#基于Nacos界面的灰度发布)
  - [基于Rest方式的灰度发布](#基于Rest方式的灰度发布)
    - [基于服务的灰度发布](#基于服务的灰度发布)
    - [基于网关的灰度发布](#基于网关的灰度发布)
    - [基于数据库的灰度发布](#基于数据库的灰度发布)
- [用户自定义和编程灰度路由的操作演示](#用户自定义和编程灰度路由的操作演示)
  - [基于服务的用户自定义和编程灰度路由](#基于服务的用户自定义和编程灰度路由)
  - [基于网关的用户自定义和编程灰度路由](#基于网关的用户自定义和编程灰度路由)

## 场景描述
- 系统部署情况：
  - 网关Zuul集群部署了1个
  - 微服务集群部署了3个，分别是A服务集群、B服务集群、C服务集群，分别对应的实例数为2、2、3
- 微服务集群的调用关系为网关Zuul->服务A->服务B->服务C
- 系统调用关系
  - 网关Zuul的1.0版本只能调用服务A的1.0版本，网关Zuul的1.1版本只能调用服务A的1.1版本
  - 服务A的1.0版本只能调用服务B的1.0版本，服务A的1.1版本只能调用服务B的1.1版本
  - 服务B的1.0版本只能调用服务C的1.0和1.1版本，服务B的1.1版本只能调用服务C的1.2版本

用规则来表述上述关系
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rule>
    <discovery>
        <version>
            <!-- 表示网关z的1.0，允许访问提供端服务a的1.0版本 -->
            <service consumer-service-name="discovery-springcloud-example-zuul" provider-service-name="discovery-springcloud-example-a" consumer-version-value="1.0" provider-version-value="1.0"/>
            <!-- 表示网关z的1.1，允许访问提供端服务a的1.1版本 -->
            <service consumer-service-name="discovery-springcloud-example-zuul" provider-service-name="discovery-springcloud-example-a" consumer-version-value="1.1" provider-version-value="1.1"/>
            <!-- 表示消费端服务a的1.0，允许访问提供端服务b的1.0版本 -->
            <service consumer-service-name="discovery-springcloud-example-a" provider-service-name="discovery-springcloud-example-b" consumer-version-value="1.0" provider-version-value="1.0"/>
            <!-- 表示消费端服务a的1.1，允许访问提供端服务b的1.1版本 -->
            <service consumer-service-name="discovery-springcloud-example-a" provider-service-name="discovery-springcloud-example-b" consumer-version-value="1.1" provider-version-value="1.1"/>
            <!-- 表示消费端服务b的1.0，允许访问提供端服务c的1.0和1.1版本 -->
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="1.0" provider-version-value="1.0;1.1"/>
            <!-- 表示消费端服务b的1.1，允许访问提供端服务c的1.2版本 -->
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="1.1" provider-version-value="1.2"/>
        </version>
    </discovery>
</rule>
```

上述微服务分别见discovery-springcloud-example-service、discovery-springcloud-example-zuul和discovery-springcloud-example-gateway三个工程。相应的服务名、端口和版本见下表

| 微服务 | 服务端口 | 管理端口 | 版本 | 区域 |
| --- | --- | --- | --- | --- |
| A1 | 1100 | 5100 | 1.0 | dev |
| A2 | 1101 | 5101 | 1.1 | qa |
| B1 | 1200 | 5200 | 1.0 | dev |
| B2 | 1201 | 5201 | 1.1 | qa |
| C1 | 1300 | 5300 | 1.0 | dev |
| C2 | 1301 | 5301 | 1.1 | qa |
| C3 | 1302 | 5302 | 1.2 | qa |
| Zuul | 1400 | 5400 | 1.0 | 无 |
| Gateway | 1500 | 5500 | 1.0 | 无 |

控制平台见discovery-springcloud-example-console，对应的版本和端口号如下表

| 服务端口 | 管理端口 |
| --- | --- |
| 2222 | 3333 |

Admin见discovery-springcloud-example-admin，对应的版本和端口号如下表

| 服务端口 | 
| --- |
| 5555 |

## 引入依赖
- 启动服务注册发现中心，默认是Eureka。可供选择的有Eureka，Zuul，Zookeeper。Eureka，请启动discovery-springcloud-example-eureka下的应用，后两者自行安装服务器
- 根据上面选择的服务注册发现中心，对示例下的discovery-springcloud-example-service/pom.xml进行组件切换
```xml
<dependency>
    <groupId>com.nepxion</groupId>
    <artifactId>discovery-plugin-starter-eureka</artifactId>
    <!-- <artifactId>discovery-plugin-starter-consul</artifactId> -->
    <!-- <artifactId>discovery-plugin-starter-zookeeper</artifactId> -->
    <!-- <artifactId>discovery-plugin-starter-nacos</artifactId> -->
    <version>${discovery.plugin.version}</version>
</dependency>
```
- 根据上面选择的服务注册发现中心，对控制台下的discovery-springcloud-example-console/pom.xml进行组件切换
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <!-- <artifactId>spring-cloud-starter-consul-discovery</artifactId> -->
    <!-- <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId> -->
    <!-- <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId> -->
</dependency>
```
- :exclamation:如果需要，引入用户自定义和编程灰度路由扩展依赖（三个依赖分别是服务端，网关Zuul端，网关Spring Cloud Gateway端，对应引入）
```xml
<dependency>
    <groupId>com.nepxion</groupId>
    <artifactId>discovery-plugin-strategy-starter-service</artifactId>
    <artifactId>discovery-plugin-strategy-starter-zuul</artifactId>
    <artifactId>discovery-plugin-strategy-starter-gatewway</artifactId>
</dependency>
```

## 服务注册过滤的操作演示
### 黑/白名单的IP地址注册的过滤
- 在rule.xml把本地IP地址写入到相应地方
- 启动DiscoveryApplicationA1.java
- 抛出禁止注册的异常，即本地服务受限于黑名单的IP地址列表，不会注册到服务注册发现中心；白名单操作也是如此，不过逻辑刚好相反

### 最大注册数的限制的过滤
- 在rule.xml修改最大注册数为0
- 启动DiscoveryApplicationA1.java
- 抛出禁止注册的异常，即本地服务受限于最大注册数，不会注册到服务注册发现中心

### 黑/白名单的IP地址发现的过滤
- 在rule.xml把本地IP地址写入到相应地方
- 启动DiscoveryApplicationA1.java和DiscoveryApplicationB1.java、DiscoveryApplicationB2.java
- 你会发现A服务无法获取B服务的任何实例，即B服务受限于黑名单的IP地址列表，不会被A服务的发现；白名单操作也是如此，不过逻辑刚好相反

## 服务发现和负载均衡控制的操作演示
### 基于图形化桌面程序的灰度发布
- 见[入门教程](https://github.com/Nepxion/Docs/blob/master/discovery-doc/README_QUICK_START.md)的“运行图形化灰度发布桌面程序”

### 基于图形化Web程序的灰度发布
- 见[入门教程](https://github.com/Nepxion/Docs/blob/master/discovery-doc/README_QUICK_START.md)的“运行图形化灰度发布Web程序”

### 基于Apollo界面的灰度发布
- 见[入门教程](https://github.com/Nepxion/Docs/blob/master/discovery-doc/README_QUICK_START.md)的“运行Apollo配置界面”

### 基于Nacos界面的灰度发布
- 见[入门教程](https://github.com/Nepxion/Docs/blob/master/discovery-doc/README_QUICK_START.md)的“运行Nacos配置界面”

### 基于Rest方式的灰度发布
#### 基于服务的灰度发布
- 启动discovery-springcloud-example-service下7个DiscoveryApplication，无先后顺序，等待全部启动完毕
- 下面URL的端口号，可以是服务端口号，也可以是管理端口号
- 通过版本改变，达到灰度访问控制，针对A服务
  - 1.1 通过Postman，执行POST [http://localhost:1100/router/routes/](http://localhost:1100/router/routes/)，填入discovery-springcloud-example-b;discovery-springcloud-example-c，查看路由路径，如图1，可以看到符合预期的调用路径
  - 1.2 通过Postman，执行POST [http://localhost:1100/version/update-sync](http://localhost:1100/version/update-sync)，填入1.1，动态把服务A的版本从1.0切换到1.1
  - 1.3 通过Postman，再执行第一步操作，如图2，可以看到符合预期的调用路径，通过版本改变，灰度访问控制成功
- 通过版本访问规则改变，达到灰度访问控制，针对B服务
  - 2.1 通过Postman，执行POST [http://localhost:1200/config/update-sync](http://localhost:1200/config/update-sync)，发送新的版本访问规则（内容见下面）
  - 2.2 通过Postman，执行POST [http://localhost:1201/config/update-sync](http://localhost:1201/config/update-sync)，发送新的版本访问规则（内容见下面）
  - 2.3 上述操作也可以通过控制平台，进行批量更新，见图5。操作的逻辑：B服务的所有版本都只能访问C服务3.0版本，而本例中C服务3.0版本是不存在的，意味着这么做B服务不能访问C服务
  - 2.4 重复1.1步骤，发现调用路径只有A服务->B服务，如图3，通过规则改变，灰度访问控制成功
- 通过版本权重规则改变，达到灰度访问控制，针对B服务
  - 2.1 通过Postman，执行POST [http://localhost:1200/config/update-sync](http://localhost:1200/config/update-sync)，发送新的版本权重规则（内容见下面）
  - 2.2 通过Postman，执行POST [http://localhost:1201/config/update-sync](http://localhost:1201/config/update-sync)，发送新的版本权重规则（内容见下面）
  - 2.3 上述操作也可以通过控制平台，进行批量更新，见图5。操作的逻辑：B服务1.0的版本向A服务提供10%流量，B服务1.1的版本向A服务提供90%流量
  - 2.4 不断重复执行4.1步骤，观察Ribbo负载均衡的时候，在调用B1.0和B1.1命中的概率，灰度权重控制成功
- 通过区域权重规则改变，达到灰度访问控制，针对B服务
  - 3.1 通过Postman，执行POST [http://localhost:1200/config/update-sync](http://localhost:1200/config/update-sync)，发送新的区域权重规则（内容见下面）
  - 3.2 通过Postman，执行POST [http://localhost:1201/config/update-sync](http://localhost:1201/config/update-sync)，发送新的区域权重规则（内容见下面）
  - 3.3 上述操作也可以通过控制平台，进行批量更新（跟版本权重规则类似的操作）。操作的逻辑：B服务1.0的版本位于区域dev，所以它向A服务提供85%流量，B服务1.1的版本位于区域qa，所以它向A服务提供15%流量
  - 3.4 不断重复执行4.1步骤，观察Ribbo负载均衡的时候，在调用B1.0和B1.1命中的概率，灰度权重控制成功  
- 负载均衡的灰度测试
  - 4.1 通过Postman，执行POST [http://localhost:1100/invoke](http://localhost:1100/invoke)，这是example内置的访问路径示例（通过Feign实现）
  - 4.2 重复“通过版本改变，达到灰度访问控制”或者“通过规则改变，达到灰度访问控制”操作，查看Ribbon负载均衡的灰度结果，如图4
- 上述操作，都是单次操作，如需要批量操作，可通过“控制平台”接口，它集成批量操作和推送到远程配置中心的功能，可以取代上面的某些调用方式
- 其它更多操作，请参考“配置中心”、“管理中心”和“控制平台”
- 附录[Postman操作集合](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Nepxion.postman_collection.json)，请下载到本地，导入到Postman中执行相关操作

新的版本访问规则
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rule>
    <discovery>
        <version>
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" consumer-version-value="" provider-version-value="3.0"/>
        </version>
    </discovery>
</rule>
```

新的版本权重规则
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rule>
    <discovery>
        <weight>
            <service consumer-service-name="discovery-springcloud-example-b" provider-service-name="discovery-springcloud-example-c" provider-weight-value="1.0=10;1.1=90"/>
        </weight>
    </discovery>
</rule>
```

新的区域权重规则
```xml
<?xml version="1.0" encoding="UTF-8"?>
<rule>
    <discovery>
        <weight>
            <region provider-weight-value="dev=85;qa=15"/>
        </weight>
    </discovery>
</rule>
```

图1

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result1.jpg)

图2

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result2.jpg)

图3

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result3.jpg)

图4

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result4.jpg)

图5

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result5.jpg)

#### 基于网关的灰度发布
- 在上面基础上，启动discovery-springcloud-example-zuul下DiscoveryApplicationZuul或者启动discovery-springcloud-example-gateway下DiscoveryApplicationGateway
- 因为Zuul和Spring Cloud Gateway是一种特殊的微服务，也遵循Spring Cloud体系的服务注册发现和负载均衡机制，所以所有操作过程跟上面完全一致

图6

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result6.jpg)

图7

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result7.jpg)

#### 基于数据库的灰度发布
- 监听规则的变化，获取客户化的参数，根据参数的变化动态切换数据源
```java
@EventBus
public class MySubscriber {
    @Autowired
    private PluginAdapter pluginAdapter;

    @Subscribe
    public void onCustomization(CustomizationEvent customizationEvent) {
        CustomizationEntity customizationEntity = customizationEvent.getCustomizationEntity();
        String serviceId = pluginAdapter.getServiceId();
        if (customizationEntity != null) {
            Map<String, Map<String, String>> customizationMap = customizationEntity.getCustomizationMap();
            Map<String, String> customizationParameter = customizationMap.get(serviceId);
            System.out.println("========== 获取客户化对象, serviceId=" + serviceId + ", customizationParameter=" + customizationParameter);
            // 根据customizationParameter的参数动态切换数据源
        } else {
            System.out.println("========== 获取客户化对象, serviceId=" + serviceId + ", customizationEntity=" + customizationEntity);
            // 根据customizationParameter的参数动态切换数据源
        }
    }
}
```

## 用户自定义和编程灰度路由的操作演示
下面版本路由策略+区域路由策略+IP和端口路由策略+自定义策略组合为例，具体请参考，图8、图9、图10、图11、图12

### 基于服务的用户自定义和编程灰度路由
- 在服务层，编程灰度路由策略，如下代码，同时启动两种策略：
  - ServiceStrategyContext策略（获取来自RPC方式的方法参数）：因为示例中只有一个方法 String invoke(String value)，表示当服务名为discovery-springcloud-example-b，同时版本为1.0，同时参数value中包含'abc'，三个条件同时满足的情况下，在负载均衡层面，对应的服务示例不会被负载均衡到
  - RequestContextHolder策略（获取来自网关的Header参数）：表示请求的Header中的token包含'abc'，在负载均衡层面，对应的服务实例不会被负载均衡到
```java
// 实现了组合策略，版本路由策略+区域路由策略+IP和端口路由策略+自定义策略
public class MyDiscoveryEnabledStrategy implements DiscoveryEnabledStrategy {
    private static final Logger LOG = LoggerFactory.getLogger(MyDiscoveryEnabledStrategy.class);

    @Autowired
    private ServiceStrategyContextHolder serviceStrategyContextHolder;

    @Autowired
    private PluginAdapter pluginAdapter;

    @Override
    public boolean apply(Server server, Map<String, String> metadata) {
        // 对Rest调用传来的Header参数（例如Token）做策略
        boolean enabled = applyFromHeader(server, metadata);
        if (!enabled) {
            return false;
        }

        // 对RPC调用传来的方法参数做策略
        return applyFromMethod(server, metadata);
    }

    // 根据Rest调用传来的Header参数（例如Token），选取执行调用请求的服务实例
    private boolean applyFromHeader(Server server, Map<String, String> metadata) {
        String token = serviceStrategyContextHolder.getHeader("token");
        String serviceId = pluginAdapter.getServerServiceId(server);

        LOG.info("Serivice端负载均衡用户定制触发：token={}, serviceId={}, metadata={}", token, serviceId, metadata);

        String filterServiceId = "discovery-springcloud-example-c";
        String filterToken = "123";
        if (StringUtils.equals(serviceId, filterServiceId) && StringUtils.isNotEmpty(token) && token.contains(filterToken)) {
            LOG.info("过滤条件：当serviceId={} && Token含有'{}'的时候，不能被Ribbon负载均衡到", filterServiceId, filterToken);

            return false;
        }

        return true;
    }

    // 根据RPC调用传来的方法参数（例如接口名、方法名、参数名或参数值等），选取执行调用请求的服务实例
    @SuppressWarnings("unchecked")
    private boolean applyFromMethod(Server server, Map<String, String> metadata) {
        Map<String, Object> attributes = serviceStrategyContextHolder.getRpcAttributes();

        String serviceId = pluginAdapter.getServerServiceId(server);
        String version = metadata.get(DiscoveryConstant.VERSION);

        LOG.info("Serivice端负载均衡用户定制触发：attributes={}, serviceId={}, metadata={}", attributes, serviceId, metadata);

        String filterServiceId = "discovery-springcloud-example-b";
        String filterVersion = "1.0";
        String filterBusinessValue = "abc";
        if (StringUtils.equals(serviceId, filterServiceId) && StringUtils.equals(version, filterVersion)) {
            if (attributes.containsKey(ServiceStrategyConstant.PARAMETER_MAP)) {
                Map<String, Object> parameterMap = (Map<String, Object>) attributes.get(ServiceStrategyConstant.PARAMETER_MAP);
                String value = parameterMap.get("value").toString();
                if (StringUtils.isNotEmpty(value) && value.contains(filterBusinessValue)) {
                    LOG.info("过滤条件：当serviceId={} && version={} && 业务参数含有'{}'的时候，不能被Ribbon负载均衡到", filterServiceId, filterVersion, filterBusinessValue);

                    return false;
                }
            }
        }

        return true;
    }
}
```

### 基于网关的用户自定义和编程灰度路由
- 在网关层（以Zuul为例），编程灰度路由策略，如下代码，策略：
  - RequestContext策略（获取来自网关的Header参数）：表示请求的Header中的token包含'abc'，在负载均衡层面，对应的服务实例不会被负载均衡到
```java
// 实现了组合策略，版本路由策略+区域路由策略+IP和端口路由策略+自定义策略
public class MyDiscoveryEnabledStrategy implements DiscoveryEnabledStrategy {
    private static final Logger LOG = LoggerFactory.getLogger(MyDiscoveryEnabledStrategy.class);

    @Autowired
    private ZuulStrategyContextHolder zuulStrategyContextHolder;

    @Autowired
    private PluginAdapter pluginAdapter;

    @Override
    public boolean apply(Server server, Map<String, String> metadata) {
        // 对Rest调用传来的Header参数（例如Token）做策略
        return applyFromHeader(server, metadata);
    }

    // 根据Rest调用传来的Header参数（例如Token），选取执行调用请求的服务实例
    private boolean applyFromHeader(Server server, Map<String, String> metadata) {
        String token = zuulStrategyContextHolder.getHeader("token");
        String serviceId = pluginAdapter.getServerServiceId(server);

        LOG.info("Zuul端负载均衡用户定制触发：token={}, serviceId={}, metadata={}", token, serviceId, metadata);

        String filterToken = "abc";
        if (StringUtils.isNotEmpty(token) && token.contains(filterToken)) {
            LOG.info("过滤条件：当Token含有'{}'的时候，不能被Ribbon负载均衡到", filterToken);

            return false;
        }

        return true;
    }
}
```

- 在网关层（以Spring Cloud Gateway为例），编程灰度路由策略，如下代码，策略：
  - GatewayStrategyContext策略（获取来自网关的Header参数）：表示请求的Header中的token包含'abc'，在负载均衡层面，对应的服务实例不会被负载均衡到
```java
// 实现了组合策略，版本路由策略+区域路由策略+IP和端口路由策略+自定义策略
public class MyDiscoveryEnabledStrategy implements DiscoveryEnabledStrategy {
    private static final Logger LOG = LoggerFactory.getLogger(MyDiscoveryEnabledStrategy.class);

    @Autowired
    private GatewayStrategyContextHolder gatewayStrategyContextHolder;

    @Autowired
    private PluginAdapter pluginAdapter;

    @Override
    public boolean apply(Server server, Map<String, String> metadata) {
        // 对Rest调用传来的Header参数（例如Token）做策略
        return applyFromHeader(server, metadata);
    }

    // 根据Rest调用传来的Header参数（例如Token），选取执行调用请求的服务实例
    private boolean applyFromHeader(Server server, Map<String, String> metadata) {
        String token = gatewayStrategyContextHolder.getHeader("token");
        String serviceId = pluginAdapter.getServerServiceId(server);

        LOG.info("Gateway端负载均衡用户定制触发：token={}, serviceId={}, metadata={}", token, serviceId, metadata);

        String filterToken = "abc";
        if (StringUtils.isNotEmpty(token) && token.contains(filterToken)) {
            LOG.info("过滤条件：当Token含有'{}'的时候，不能被Ribbon负载均衡到", filterToken);

            return false;
        }

        return true;
    }
}
```

图8

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result8.jpg)

图9

![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result9.jpg)

图10
只要填入版本的值，版本路由策略将自动开启
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result10.jpg)

图11
只要填入区域的值，区域路由策略将自动开启
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result11.jpg)

图12
只要填入IP和端口的值，IP和端口路由策略将自动开启
![Alt text](https://github.com/Nepxion/Docs/blob/master/discovery-doc/Result12.jpg)