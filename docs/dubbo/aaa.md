> Dubbo 的官网：[https://dubbo.apache.org/zh/](https://dubbo.apache.org/zh/)

## 项目结构
- dubbo-demo 作为外层父项目。
- dubbo-api：作为公共模块，暴露接口，服务提供者与服务生产者模块依赖于它；
- dubbo-provider：服务提供者，用于实现接口；
- dubbo-consumer：服务消费者，用于调用远程接口；
![在这里插入图片描述](https://img-blog.csdnimg.cn/bf528bf5ea5248b09c74dfc08b2e58a0.png)

需求：dubbo-consumer 想调用 dubbo-provider 实现的接口方法，但它们位于不同的项目中；

对于这种远程接口调用，可借助 Dubbo 这一 RPC 框架来实现，同时采用 zookeeper 作为注册中心来管理服务；
## 环境构建
在 dubbo-api 中，引入 dubbo 整合 SpringBoot 的依赖：
```xml
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>2.7.3</version>
        </dependency>
```
再引入 dubbo 整合 zookeeper 的依赖：
```xml
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-zookeeper</artifactId>
            <version>2.7.3</version>
            <type>pom</type>
            <exclusions>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```
dubbo-provider 与 dubbo-consumer 中分别引入 dubbo-api 的坐标：
```xml
        <dependency>
            <artifactId>dubbo-api</artifactId>
            <groupId>com.study</groupId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```
## 简单 Demo
dubbo-api 中提供 UserService 接口：
```java
package com.study.api;

public interface UserService {
    User queryUserById(Integer id);
}
```
注意，User 要实现序列化接口，因为 RPC 调用中涉及到序列化传输：
```java
package com.study.api;
import java.io.Serializable;

public class User implements Serializable {
    private Integer id;
    private String name;
    // 省略构造，get/set，toString()
```
dubbo-provider 中实现 UserService 接口的方法：
```java
package com.study.provider;
import com.study.api.User;
import com.study.api.UserService;
import org.apache.dubbo.config.annotation.Service;

@Service(version = "1.0")
public class UserServiceImpl implements UserService {
    @Override
    public User queryUserById(Integer id) {
        User user = new User(id, "study");
        return user;
    }
}
```
注意这里采用的是 dubbo 提供的 `@Service` 注解，暴露服务，version 参数指定版本信息；

dubbo-consumer 中调用该接口方法：
```java
package com.study.consumer;
import com.study.api.UserService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.stereotype.Component;
import javax.annotation.PostConstruct;

@Component
public class UserTest {
    @Reference(version = "1.0")
    private UserService userService;
    @PostConstruct
    public void test() {
        System.out.println(userService.queryUserById(666));
    }
}
```
注意这里采用的是 dubbo 提供的 `@Reference` 注解，且 version 值需与前面 Service 中的一致，这样就能自动去注册中心查找所需服务；`@PostConstruct` 注解能在 Spring 程序启动时自动调用其修饰的方法；
## Dubbo 核心配置
dubbo-provider 服务提供者的配置文件：
```yml
server:
  port: 8001                              # 指定启动端口，防止冲突

dubbo:
  application:
    name: provider
  registry:
    address: zookeeper://localhost:2181   # 绑定的注册中心地址，采用 ZK，默认端口为 2181
    timeout: 6000                         # 超时时间，单位 ms
  metadata-report:
    address: zookeeper://localhost:2181   # 元中心地址
  protocol:
    name: dubbo                           # 协议名称为 dubbo
    port: 20880                           # 协议端口为 20880
  scan:
    base-packages: com.study.provider     # 扫描服务包所在的位置
```
dubbo-consumer 中服务消费者的配置文件：
```yml
server:
  port: 8002                              # 指定启动端口，防止冲突

dubbo:
  application:
    name: consumer
  registry:
    address: zookeeper://localhost:2181   # 绑定的注册中心地址，采用 ZK，默认端口为 2181
```
## 安装启动 Zookeeper
- Zookeeper 下载地址：[https://archive.apache.org/dist/zookeeper/](https://archive.apache.org/dist/zookeeper/)；
- 解压后先进入 conf 目录，将 zoo_sample.cfg 复制一份，改名为 `zoo.cfg`；
- 可在 zookeeper 外层目录下新建 data 目录，然后在 zoo.cfg 中，将 dataDir 的属性值修改为 ../data；
- 最后进入到 bin 目录，双击 `zkServer.cmd` 启动 Zookeeper；

## Demo 运行
先启动 dubbo-provider，再启动 dubbo-consumer，运行结果中输出调用接口的返回值。
![在这里插入图片描述](https://img-blog.csdnimg.cn/55bbc666c4594705ab7757d7ead50f12.png)
## 总结
- 在分布式架构中，不同的服务可能位于不同的服务器上，也就不能像传统单体架构那样直接调用。
- 对于这种异地的调用，采用的是 RPC 调用（远程过程调用）。而 Dubbo 是一个高性能的 RPC 框架，能够为我们解决分布式系统中不同服务之间的通信调用问题等。
- 对于众多服务的管理，需要引入注册中心。在 Dubbo 中，一般采用 Zookeeper 作为配置中心。