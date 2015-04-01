---
layout: post
title:  "使用python把excel文件转换成XML文件"
keywords: "python,excel,xml"
description: "用python把Excel文件内容转成XML格式的内容"
category: python
tags: [python]
---
#### 实现步骤如下：
* 需要把存储的excel文件转换成CSV文件
* python `code`
* 
```
import csv 
reader = csv.reader(open("test.csv")) 
writer = open("contacts.xml","w") 
for name, phone, adress in reader: 
     writer.write("<item>\n" + 
                 "\t<name>"+name+"</name>\n" + 
                 "\t<phone>"+phone+"</phone>\n" + 
                 "\t<adress>"+adress+"</adress>\n" + 
                 "</item>\n") 
 writer.close() 
```
* 在CSV所在的目录中运行这段代码，就能生成xml文件
* ![图1](/static/images/python01.jpg)
* ![图2](/static/images/python02.jpg)