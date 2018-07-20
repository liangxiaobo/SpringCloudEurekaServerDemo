# SpringCloudEurekaServerDemo
高可用EurekaServer搭建

## 创建Eureka Server 
项目名叫eureka-server，并在下面添加两个profile文件 peer1和peer2
目录结构如下：
 ![a1.png](https://upload-images.jianshu.io/upload_images/2151905-b865236653dd3e56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### peer1内容
``` xml
spring:
  profiles: peer1
server:
  port: 8761
eureka:
  instance:
    hostname: peer1
  client:
    service-url: 
       defaultZone: http://peer2:8762/eureka/
    register-with-eureka: false
    fetch-registry: false
```
### peer2内容
``` xml
spring:
  profiles: peer2
server:
  port: 8762
eureka:
  instance:
    hostname: peer2
  client:
    service-url: 
      defaultZone: http://peer1:8761/eureka/
    register-with-eureka: false
    fetch-registry: false
```
  请注意，每一个EurekaServer同时也都是EurekaClient，所以peer1中的service-url指向的是peer2，向相同实例同步注册表信息,service-url中的peer1和peer2在生产环境应该是真实的IP。
### 配置本机的etc/hosts文件
本地开发测试的话，需要在etc/hosts文件中添加：
``` bash
127.0.0.1 peer1
127.0.0.1 peer2
```
启动在IDEA中启动两的EurekaServer的话，首先看如何一个工程启动多个实例，请看这篇文章:[https://blog.csdn.net/forezp/article/details/76408139](https://blog.csdn.net/forezp/article/details/76408139)
其次可以在```application.properties```中添加```spring.profiles.active=peer1```启动第一个实例，再修改为```spring.profiles.active=peer2```启动第二个实例
第二种启动方式是，用maven打包之后在target目录中打到jar文件用命令启动：
``` bash
java -jar eureka-server.jar -- spring.profiles.active=peer1
java -jar eureka-server.jar -- spring.profiles.active=peer2
```
访问 http://peer1:8761和 http://peer2:8762/

### 创建一个Eureka Client注册服务

配置文件：
``` xml
eureka:
  client:
    service-url:
      defaultZone: http://peer1:8761/eureka/
server:
  port: 8763
spring:
  application:
    name: eureka-client
```
启动eureka-client后，再看
http://peer1:8761
![a5.png](https://upload-images.jianshu.io/upload_images/2151905-5160031a58ceac68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 http://peer2:8762/

![a6.png](https://upload-images.jianshu.io/upload_images/2151905-1ffc2be27a2f6b13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Eureka Client只向peer1提供了注册而peer2也被同步了注册表
