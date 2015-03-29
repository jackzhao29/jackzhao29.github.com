---
layout: post
title:  "使用XFire框架实现WebService接口的实践"
keywords: "XFire,webservice"
description: "使用XFire框架实现WebService接口的实践"
category: java
tags: [java]
---
#### 开发服务端步骤：
* [下载需要的包](http://xfire.codehaus.org/Download)
* 创建项目名称为XfireWebService的Web项目
* 创建IReaderObject接口
* 创建实现接口ReaderObjectImpl类
* 在项目的src目录下创建META-INF文件夹，再在META-INF文件夹下创建xfire文件夹，如图1：
![图1](/static/images//01.jpg)
* 在xfire文件夹下创建service.xml文件，内容如下：

```
<?xml version="1.0" encoding="UTF-8"?> 
 <beans xmlns="http://xfire.codehaus.org/config/1.0"> 
     <service> 
         <!-- webservice名称，调用时需要指定这个 --> 
         <name>ReaderService</name> 
         <!-- 这个一般是自己公司的网址，意义不大 --> 
         <namespace>http://www.douban.com/people/happy829</namespace> 
         <!-- 接口类 --> 
         <serviceClass>com.cn.service.IReaderObject</serviceClass> 
         <!-- 实现类 --> 
         <implementationClass>com.cn.service.impl.ReaderObjectImpl</implementationClass> 
     </service> 
 </beans>
```
* 在web.xml文件里添加如下内容(添加到<web-app>节点中)

```
<servlet> 
         <servlet-name>XFireServlet</servlet-name> 
         <servlet-class>org.codehaus.xfire.transport.http.XFireConfigurableServlet</servlet-class> 
     </servlet> 
     <servlet-mapping> 
         <servlet-name>XFireServlet</servlet-name> 
         <url-pattern>/services/*</url-pattern> 
     </servlet-mapping> 
```
* 发布项目并且部署到tomcat，并且在浏览器输入http://localhost:8080/XfireWebService/services/ReaderService?wsdl，如图2：

![图2](/static/images/02.jpg)
#### 开发客户端
* 第一种实现方式：通过WebService服务端提供的接口来创建客户，源码如下：

```
/*
 * 
 * 第一种方法：通过WebService服务端提供的接口来创建客户端
 * 客户端必须提供一个与服务端完全一致的接口，包明也要一致
 * 本例中需要在客户端提供IReaderObject接口
 */      
 Service srcModel=new ObjectServiceFactory().create(IReaderObject.class);
                XFireProxyFactory factory=new XFireProxyFactory(XFireFactory.newInstance().getXFire());
                String ServiceUrl="http://localhost:8080/XfireWebService/services/ReaderService";
                try
                {
                        IReaderObject readerObject=(IReaderObject)factory.create(srcModel,ServiceUrl);
                        
                        String str=readerObject.GetObject("张三", 18, "北京市");
                        System.out.println(str);
                                
                }
                catch(Exception ex)
                {
                        ex.printStackTrace();
                }
```
如图3

![图3](/static/images/03.jpg)

* 第二种实现方式：通过wsdl地址来创建动态客户端，源码如下：

```
/**
                 * 第二种方式
                 * 通过wsdl地址来创建动态客户端
                 */
                
                try
                {
                        String ServiceUrl="http://localhost:8080/XfireWebService/services/ReaderService?wsdl";
                        Client client=new Client(new URL(ServiceUrl));
                        Object[] result=client.invoke("GetObject", new Object[]{"李斯",12,"上海市"});
                        System.out.println(result[0]);
                }
                catch(Exception ex)
                {
                        ex.printStackTrace();
                }
```
如图4

![图4](/static/images/04.jpg)
