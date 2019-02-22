# 基于阿里Fescar分布式事务解决案例

一、什么是Fescar？

Fescar全称为：Fast Easy Commit And Rollback，是阿里巴巴公司基于TXC和GTS于2019年开源的微服务架构分布式事务解决方案。该方案基于java实现、简单易用、性能强悍、业务侵入低，是一款能够解决大多数分布式事务场景的极佳选择。原理解析请关注 : https://github.com/alibaba/fescar

二、原理浅析和场景展示

![FESCAR solution](https://camo.githubusercontent.com/b3a71332ae0a91db7f8616286a69b879fcbea672/68747470733a2f2f63646e2e6e6c61726b2e636f6d2f6c61726b2f302f323031382f706e672f31383836322f313534353239363739313037342d33626365376263652d303235652d343563332d393338362d3762393531333564616465382e706e67)

​						该案例实现的微服务架构下的场景示例图



![Typical Process](https://camo.githubusercontent.com/0384806afd7c10544c258ae13717e4229942aa13/68747470733a2f2f63646e2e6e6c61726b2e636f6d2f6c61726b2f302f323031382f706e672f31383836322f313534353239363931373838312d32366661626562392d373166612d346633652d386137612d6663333137643333383966342e706e67)

​								     Fescar原理图浅析



三、核心技术栈

* SpringBoot 2.0.8.RELEASE（2.0以后第一个GA版本）
* SpringCloud Finchley.SR2
* SpringCloudOpenFeigin
* Durid 1.1.10（阿里巴巴开源高性能数据源连接池）
* Mybatis 3.4.6（Mybatis-3）
* Dubbo 2.6.2（阿里巴巴开源高性能RPC框架）
* Fescar 0.1.1（阿里巴巴开源分布式事务解决方案，第二个小版本）
* Nacos 0.8.0（阿里巴巴开源注册中心/配置中心）

四、实现以及规划

* 当前版本实现SpringBoot + Dubbo + Mybatis + Nacos +Fescar技术整合，实现如何在微服务架构中使用分布式事务框架Fescar
* 接下来版本等到Fescar完善到0.5.0版本后开始支持SringCloud相关技术栈，将实现在Cloud微服务架构中解决分布式事务
* 计划暂定上述

五、使用该案例说明

1. 前往Fescar Github官方页面下载最新版本的  https://github.com/alibaba/fescar/releases

2. 前往Nacos Github官方页面下载最新版本的   https://github.com/alibaba/nacos/releases

3. clone此项目到本地，使用maven构建导入IDEA编辑器中，配置项目使用的maven仓库和JDK版本（1.8）

4. 将doc目录中的sql脚本导入到mysql数据库中，在此之前先创建数据库 db_gts_fescar，设置用户名密码为root  root

5. 模块说明：

   - account-gts-fescar-example  用户账户微服务模块
   - dubbo-gts-fescar-example  业务发起方模块
   - gts-fescar-example-common  项目公共架构模块
   - order-gts-fescar-example  订单微服务模块
   - storage-gts-fescar-example  库存微服务模块

   其他未说明模块均需后续更新，暂未使用

6. 首先启动Nacos和Fescar，中间件具体使用说明详见上述Github官方页

7. 分别启动account-gts-fescar-example、storage-gts-fescar-example、order-gts-fescar-example、dubbo-gts-fescar-example四个模块，确定微服务都注册到Nacos和Fescar

8. 使用Postman工具请求Post接口地址：http://localhost:8104/business/dubbo/buy   模拟发起下单业务请求，成功后返回200

9. 接下来测试全局回滚功能，打开dubbo-gts-fescar-example模块下的 BusinessServiceImpl类，打开被注释的代码

   ```
   if (!flag) {
      throw new RuntimeException("测试抛异常后，分布式事务回滚！");
   }
   ```

10. 再次请求测试发生异常后全局事务是否被回滚

六、nacos配置中心

1. 将上述案例dubbo调用模块resources目录下的application.properties中的配置复制到nacos配置中心，并在resources目录下新建bootstrap.properties文件，文件中的spring.application.name的值为配置的dataid，具体如图所示：

   ![1550824725416](C:\Users\heshouyou\AppData\Roaming\Typora\typora-user-images\1550824725416.png)

   ![1550824557296](C:\Users\heshouyou\AppData\Roaming\Typora\typora-user-images\1550824557296.png)

   ![1550825358155](C:\Users\heshouyou\AppData\Roaming\Typora\typora-user-images\1550825358155.png)

2. 给具有@Value或者通过${}引用配置文件值得类加上@refreshScope注解，当配置中心配置变更时会实时推送给应用，spring容器会重新加载具有该注解的bean

   说明：使用bootstrap.properties中设置nacos地址，这是nacos适配springcloud的方式。如果没有使用springcloud请查看官网博客学习配置使用  https://nacos.io/en-us/docs/quick-start.html

3. 配置中心具体内容如下：

   * dataid 为： account-fescar-example.properties

   * 配置内容：server.port=8102

     spring.application.name=account-gts-fescar-example

     \#====================================Dubbo config===============================================

     dubbo.application.id= account-gts-fescar-example

     dubbo.application.name= account-gts-fescar-example

     dubbo.protocol.id=dubbo

     dubbo.protocol.name=dubbo

     dubbo.registry.id=dubbo-account-fescar

     dubbo.registry.address=nacos://127.0.0.1:8848

     dubbo.protocol.port=20880

     dubbo.application.qosEnable=false

     \#===================================注册中心配置==========================================

     \#Nacos注册中心

     spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848

     management.endpoints.web.exposure.include=*

     \#====================================mysql datasource================================

     spring.datasource.driver-class-name=com.mysql.jdbc.Driver

     spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

     spring.datasource.url=jdbc:mysql://127.0.0.1:3306/db_gts_fescar?useSSL=false&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true

     spring.datasource.username=root

     spring.datasource.password=root

     \#自动提交

     spring.datasource.default-auto-commit=true

     \#指定updates是否自动提交

     spring.datasource.auto-commit=true

     spring.datasource.maximum-pool-size=100

     spring.datasource.max-idle=10

     spring.datasource.max-wait=10000

     spring.datasource.min-idle=5

     spring.datasource.initial-size=5

     spring.datasource.validation-query=SELECT 1

     spring.datasource.test-on-borrow=false

     spring.datasource.test-while-idle=true

     \# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒

     spring.datasource.time-between-eviction-runs-millis=18800

     \# 配置一个连接在池中最小生存的时间，单位是毫秒

     spring.datasource.minEvictableIdleTimeMillis=300000

     \#=====================================mybatis配置======================================

     mybatis.mapper-locations=classpath*:/mapper/*.xml

     

   * dataid 为： order-fescar-example.properties

   * 配置内容：server.port=8101

     spring.application.name=order-gts-fescar-example

     \#====================================Dubbo config===============================================

     dubbo.application.id= order-gts-fescar-example

     dubbo.application.name= order-gts-fescar-example

     dubbo.protocol.id=dubbo

     dubbo.protocol.name=dubbo

     dubbo.registry.id=dubbo-order-fescar

     dubbo.registry.address=nacos://127.0.0.1:8848

     dubbo.protocol.port=20881

     dubbo.application.qosEnable=false

     \#===================================注册中心配置==========================================

     \#Nacos注册中心

     spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848

     management.endpoints.web.exposure.include=*

     \#====================================mysql datasource================================

     spring.datasource.driver-class-name=com.mysql.jdbc.Driver

     spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

     spring.datasource.url=jdbc:mysql://127.0.0.1:3306/db_gts_fescar?useSSL=false&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true

     spring.datasource.username=root

     spring.datasource.password=root

     \#自动提交

     spring.datasource.default-auto-commit=true

     \#指定updates是否自动提交

     spring.datasource.auto-commit=true

     spring.datasource.maximum-pool-size=100

     spring.datasource.max-idle=10

     spring.datasource.max-wait=10000

     spring.datasource.min-idle=5

     spring.datasource.initial-size=5

     spring.datasource.validation-query=SELECT 1

     spring.datasource.test-on-borrow=false

     spring.datasource.test-while-idle=true

     \# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒

     spring.datasource.time-between-eviction-runs-millis=18800

     \# 配置一个连接在池中最小生存的时间，单位是毫秒

     spring.datasource.minEvictableIdleTimeMillis=300000

     \#=====================================mybatis配置======================================

     mybatis.mapper-locations=classpath*:/mapper/*.xml

     

   * dataid 为： storage-fescar-example.properties

   * 配置内容：server.port=8100

     spring.application.name=storage-gts-fescar-example

     \#====================================Dubbo config===============================================

     dubbo.application.id= storage-gts-fescar-example

     dubbo.application.name= storage-gts-fescar-example

     dubbo.protocol.id=dubbo

     dubbo.protocol.name=dubbo

     dubbo.registry.id=dubbo-storage-fescar

     dubbo.registry.address=nacos://127.0.0.1:8848

     dubbo.protocol.port=20882

     dubbo.application.qosEnable=false

     \#===================================注册中心配置==========================================

     \#Nacos注册中心

     spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848

     management.endpoints.web.exposure.include=*

     \#====================================mysql datasource================================

     spring.datasource.driver-class-name=com.mysql.jdbc.Driver

     spring.datasource.type=com.alibaba.druid.pool.DruidDataSource

     spring.datasource.url=jdbc:mysql://127.0.0.1:3306/db_gts_fescar?useSSL=false&useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true

     spring.datasource.username=root

     spring.datasource.password=root

     \#自动提交

     spring.datasource.default-auto-commit=true

     \#指定updates是否自动提交

     spring.datasource.auto-commit=true

     spring.datasource.maximum-pool-size=100

     spring.datasource.max-idle=10

     spring.datasource.max-wait=10000

     spring.datasource.min-idle=5

     spring.datasource.initial-size=5

     spring.datasource.validation-query=SELECT 1

     spring.datasource.test-on-borrow=false

     spring.datasource.test-while-idle=true

     \# 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒

     spring.datasource.time-between-eviction-runs-millis=18800

     \# 配置一个连接在池中最小生存的时间，单位是毫秒

     spring.datasource.minEvictableIdleTimeMillis=300000

     \#=====================================mybatis配置======================================

     mybatis.mapper-locations=classpath*:/mapper/*.xml

     

   * dataid 为： dubbo-fescar-example.properties

   * 配置内容：server.port=8104

     spring.application.name=dubbo-gts-fescar-example

     \#============================dubbo config==============================================

     dubbo.application.id=dubbo-gts-fescar-example

     dubbo.application.name=dubbo-gts-fescar-example

     dubbo.protocol.id=dubbo

     dubbo.protocol.name=dubbo

     dubbo.protocol.port=10001

     dubbo.registry.id=dubbo-gts-fescar

     dubbo.registry.address=nacos://127.0.0.1:8848

     dubbo.provider.version=1.0.0

     dubbo.application.qosEnable=false