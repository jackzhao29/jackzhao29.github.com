---
layout: post
title:  "java出现内存溢出解决方案"
keywords: "java,outofmemoryerror"
description: "启动tomcat时有时候会出现java.lang.OutOfMemoryError:Java heap space异常"
category: Java
tags: [Java]
---
#### 如果启动tomcat时出现java.lang.OutOfMemoryError:Java heap space异常时尝试使用下列办法解决<br>
* 修改Tomcat/bin/catalina.bat,添加如下内容set JAVA_OPTS=-Xms256m -Xmx512m -Djava.awt.headless=true [-XX:MaxPermSize=128M]
* eclipse->windows->preferences..->tomcat->jvm..->jvm文本框里，添加-Xms256m -Xmx512m
* eclipse->preference->java->instal jres->edit,增加参数：-Xms256m -Xmx512m<br>
参考原因：JVM中如果98％的时间是用于GC且可用的, Heap size不足2％的时候将抛出此异常信息。
JVM堆的设置是指java程序运行过程中JVM可以调配使用的内存空间的设置.JVM在启动的时候会自动设置Heap size的值，其初始空间(即-Xms)是物理内存的1/64，最大空间(-Xmx)是物理内存的1/4。
可以利用JVM提供的-Xmn -Xms -Xmx等选项可进行设置。Heap Size 最大不要超过可用物理内存的80％，一般的要将-Xms和-Xmx选项设置为相同，而-Xmn为1/4的-Xmx值。
