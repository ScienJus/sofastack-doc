## 客户端熔断

SOFA RPC 已集成 Hystrix 提供熔断能力，当前提供第一个预览版。关于 Hystrix 的更多介绍可以参考 [Hystrix 官方文档](https://github.com/Netflix/Hystrix)，Hystrix 集成能力主要由 [ScienJus](https://github.com/ScienJus) 提供，感谢贡献。

接下来介绍一下如何体验 Hystrix 带来的熔断能力，以下示例使用 `SOFA RPC 5.5.0` 版本，更多 `Hystrix` 的配置及 `SOFA Boot` 集成使用方式将在后续版本提供，敬请关注。

### 准备工作

1. Hystrix 模块作为可选模块默认不会直接加载，如需要使用，需要先主动加入 Hystrix maven 依赖：
```xml
<dependency>
        <groupId>com.netflix.hystrix</groupId>
        <artifactId>hystrix-core</artifactId>
        <version>1.5.12</version>
</dependency>
```
2. 显示开启 `Hystrix` 能力，注入 `HystrixFilter`：
```java
ConsumerConfig<HelloService> consumerConfig = new ConsumerConfig<HelloService>()
        .setInterfaceId(HelloService.class.getName())
        .setParameter(HystrixConstants.SOFA_HYSTRIX_ENABLED, String.valueOf(true))
        .setFilterRef(Collections.<Filter> singletonList(new HystrixFilter()));
```

### FallbackFactory

`FallbackFactory` 提供接口主要提供 `Fallback` 实现的注入能力。

1. 定义接口 `Fallback` 实现：
```java
public class HelloServiceFallback implements HelloService {
    @Override
    public String sayHello(String name, int age) {
        return "fallback " + name + " from server! age: " + age;
    }
}
```
2. 注入 `Fallback` 实现：
```java
ConsumerConfig<HelloService> consumerConfig = new ConsumerConfig<HelloService>()
        .setInterfaceId(HelloService.class.getName())
        .setParameter(HystrixConstants.SOFA_HYSTRIX_ENABLED, String.valueOf(true))
        .setFilterRef(Collections.<Filter> singletonList(new HystrixFilter()));
// 可以直接使用默认的 FallbackFactory 直接注入 Fallback 实现
SofaHystrixConfig.registerFallback(consumerConfig, new HelloServiceFallback());
// 也可以自定义 FallbackFactory 直接注入 FallbackFactory
SofaHystrixConfig.registerFallbackFactory(consumerConfig, new HelloServiceFallbackFactory());
```
3. 当服务端响应失败时，客户端可以触发 `Fallback` 逻辑执行。

### SetterFactory

`SetterFactory` 提供 `Hystrix` 细粒度配置能力，SOFA RPC 已提供默认的 `DefaultSetterFactory` 来生成每个调用方对应的 `Setter`，如有更定制化的述求，也可以针对每个 `ConsumerConfig` 提供自定义 `SetterFactory`。
```java
SofaHystrixConfig.registerSetterFactory(consumerConfig, new CustomSetterFactory());
```

### 支持 Hystrix 的版本信息

SOFA RPC: [5.5.0](https://github.com/alipay/sofa-rpc/releases), SOFA Boot: [2.5.3](https://github.com/alipay/sofa-boot/releases/)。

SOAF RPC 集成验证 Hystrix 版本：`1.5.12`。