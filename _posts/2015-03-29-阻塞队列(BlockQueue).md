---
layout: post
title:  "阻塞队列的示例"
keywords: "blockQueue,阻塞队列"
description: "阻塞队列"
category: java
tags: [java]
---
代码如下：

```
package com.aliyun.blockqueue;
 
import java.util.LinkedList;
import java.util.List;
 
/**
 * 阻塞队列实现
 * Created with IntelliJ IDEA.
 */
public class SimpleBlockingQueue {
    private List blockQueue=new LinkedList(); //存放实例的队列
    private int limit=10;  //队列的上限
 
    public SimpleBlockingQueue(int limit){
        this.limit=limit;
    }
 
/**
 * 向队列中添加数据
 * @param object
 */
public synchronized void put(Object object){
   //判断队列是否已满
   while(this.limit==blockQueue.size()){
   try {
           wait();
           System.out.println("进入阻塞"+object);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
      }
    //判断队列是否还没有数据,如果没有唤醒所有线程
    if(blockQueue.size()==0){
        notifyAll();
        System.out.println("没有数据，唤醒所有线程");
     }
    //向队列中添加元素
    this.blockQueue.add(object);
    for(Object obj:this.blockQueue){
        System.out.println("元素"+obj+"已添加到队列中");
    }
 
  }
 
    /**
     * 获取队列中的数据
     */
    public synchronized Object take(){
         //判断队列是否有数据
        while(this.blockQueue.size()==0){
            try {
                wait();
                System.out.println("获取数据时进入阻塞状态");
            } catch (InterruptedException e) {
                e.printStackTrace(); 
            }
        }
        //判断队列是否已满
        if(this.limit==this.blockQueue.size()){
                notifyAll();
            System.out.println("获取数据时队列已满");
        }
        System.out.println(this.blockQueue.remove(0));
        return this.blockQueue.remove(0);
 
    }
}
 
package com.aliyun.blockqueue;
 
import org.junit.Test;
 
/**
 * 测试阻塞队列的实现
 * Created with IntelliJ IDEA.
 */
public class TestSimpleBlockingQueue {
    @Test
    public void testPut(){
        SimpleBlockingQueue simpleBlockingQueue=new SimpleBlockingQueue(5);
        for(int i=0;i<10;i++){
            simpleBlockingQueue.put("测试数据");
            if(i==7){
                simpleBlockingQueue.take();
            }
        }
 
    }
}
```