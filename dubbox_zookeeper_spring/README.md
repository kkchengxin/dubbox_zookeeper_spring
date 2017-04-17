# MicroService
dubbo演示项目
用于dubbo的入门演示

Dubbo整合Zookeeper和spring示例程序：
http://blog.csdn.net/u010717403/article/details/51861595
1.Dubbo架构
本篇文章基于dubbox，使用dubbo应该也可以正常运行。
我认为想讲清楚一个任何一个技术框架，首先熟悉架构是非常有必要的。这将对对整个架构的理解有非常大的帮助。
我们首先看看Dubbo的架构,这段摘抄自Dubbo官方文档

节点角色说明：
Provider: 暴露服务的服务提供方。
Consumer: 调用远程服务的服务消费方。
Registry: 服务注册与发现的注册中心。
Monitor: 统计服务的调用次调和调用时间的监控中心。
Container: 服务运行容器。
调用关系说明：
服务容器负责启动，加载，运行服务提供者。
服务提供者在启动时，向注册中心注册自己提供的服务。
服务消费者在启动时，向注册中心订阅自己所需的服务。
注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。 看到这可能会发现，Dubbo其实就是一个分布式RPC的框架。下面我们用一个示范例子来说明该框架的使用方法。
2.运行Dubbo
我这里采用的是当当网修改后的版本DubboX。
0. 安装运行Zookeeper
因为需要用到zookeeper作为注册中心，所以必须先安装运行zookeeper，到apache官网下载zookeeper，解压缩后找到zookeeper根目录下的conf目录下的zoo_sample.cfg文件，复制并重命名一份为zoo.cfg

在这里主要关注第14行zookeeper的开放端口，默认为2181。
修改完毕后在终端中进入zookeeper的根目录，执行bin下面的启动命令bin/zkServer.sh start

先下载到本地，并且解压缩
在Dubbox官网中下载Dubbox，并且解压缩。
编译
在终端下进入Dubbox的根目录，执行编译命令mvn install -Dmaven.test.skip=true。编译过程比较漫长，可以先干其他事情再过来看结果，如果编译过程出错，可以搜索相关错误信息进行解决。
创建相应的工程
编译完成后需要通过maven将项目创建成eclipse项目或者idea项目，由于我使用的是idea，所以这里我直接创建idea项目mvn idea:idea,创建eclipse项目则执行mvn eclipse:eclipse。
完成后将项目导入开发工具中，我在这里导入idea，然后对项目执行maven install操作。

如果报没有配置jdk的错误，则把jdk配置到项目。
启动Dubbo 在项目编译成功后会在dubbo-admin的target目录下生成dubbo-admin-2.8.4文件夹，将此文件夹下的内容拷贝到tomcat/webapps/ROOT目录下,然后在tomcat根目录下执行bin/startup.sh，成功启动tomcat后在浏览器输入http://localhost:8080/，如果提示输入用户名和密码则输入root和root。如果进入如下页面则恭喜你，Dubbo启动成功。

3.开发示例程序
1.开发服务提供者provider
a.首先创建接口类TestRegistryService，创建抽象方法hello。
[java] view plain copy 在CODE上查看代码片派生到我的代码片
package com.stocsis.gavin.registry.service;

/**
 * Created by qhg on 16/6/30.
 */
public interface TestRegistryService {
    public String hello(String name);
}


b.编写接口实现方法TestRegistryServiceImpl,实现具体的方法
[java] view plain copy 在CODE上查看代码片派生到我的代码片
package com.stocsis.gavin.registry.serviceImpl;

import com.stocsis.gavin.registry.service.TestRegistryService;

/**
 * Created by qhg on 16/6/30.
 */
public class TestRegistryServiceImpl implements TestRegistryService {
    public String hello(String name) {
        return "hello "+name;
    }
}


c.编写dubbo-provider.xml配置文件
[html] view plain copy 在CODE上查看代码片派生到我的代码片
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 提供方应用名称信息，这个相当于起一个名字，我们dubbo管理页面比较清晰是哪个应用暴露出来的 -->
    <dubbo:application name="dubbo_provider"></dubbo:application>
    <!-- 使用zookeeper注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <!-- 要暴露的服务接口 -->
    <dubbo:service interface="com.stocsis.gavin.registry.service.TestRegistryService" ref="testRegistryService"/>

    <bean id="testRegistryService" class="com.stocsis.gavin.registry.serviceImpl.TestRegistryServiceImpl"></bean>

</beans>


d.运行provider
[java] view plain copy 在CODE上查看代码片派生到我的代码片
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class DemoProvider {

    public static void main(String[] args)throws Exception {
        ClassPathXmlApplicationContext content = new ClassPathXmlApplicationContext(new String[]{"dubbo-provider.xml"});
        content.start();
        System.in.read();
    }

}


运行成功后查看dubbo的监控页面

2.开发服务消费者consumer
a.编写dubbo-consumer
[html] view plain copy 在CODE上查看代码片派生到我的代码片
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <!-- 提供方应用名称信息，这个相当于起一个名字，我们dubbo管理页面比较清晰是哪个应用暴露出来的 -->
    <dubbo:application name="dubbo_consumer"></dubbo:application>
    <!-- 使用zookeeper注册中心暴露服务地址 -->
    <dubbo:registry address="zookeeper://127.0.0.1:2181" check="true"/>

    <dubbo:reference interface="com.stocsis.gavin.registry.service.TestRegistryService" id="testRegistryService"/>

</beans>


b.编写服务消费类IndexController
[java] view plain copy 在CODE上查看代码片派生到我的代码片
package com.stocsis.gavin.provider.controller;

import com.stocsis.gavin.registry.service.TestRegistryService;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Created by qhg on 16/7/1.
 */
public class IndexController {


    public static void main(String[] args) throws Exception {
        ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[]{"dubbo-consumer.xml"});
        context.start();

        TestRegistryService testRegistryService = (TestRegistryService) context.getBean("testRegistryService"); // 获取远程服务代理
        String hello = testRegistryService.hello("world"); // 执行远程方法

        System.out.println(hello); // 显示调用结果
        System.in.read();
    }
}



在这里不需要自己创建对象，而是直接获取远程bean对象，这就是dubbo提供的主要功能。看看运行结果吧。


dubbo的消费者页面

至此，整个示例结束。
参考：https://github.com/qiuhonggang/MicroService