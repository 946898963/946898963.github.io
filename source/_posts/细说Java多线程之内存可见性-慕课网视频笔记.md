---
title: 细说Java多线程之内存可见性----慕课网视频笔记
date: 2016-06-20 23:07:58
categories:
- 技术
- 多线程
tags:
- 多线程
---

## 可见性

可见性：一个线程对共享变量值的修改能够及时的被其他的线程看到，这个时候就说这个变量在线程之间是可见的。

共享变量：一个变量在多个线程的工作内存中都存在副本，那么这个变量就是这几个线程的共享变量。

## Java内存模型（JMM）

java内存模型描述了java程序中各种变量（线程共享变量）的访问规则，以及在JVM中将变量存储到内存和从内存中读取出变量这样的底层细节。

> * 所有的变量都存储在主内存中。
> * 每个线程都有自己独立的工作内存，里面保存了线程使用到的变量的副本（主内存中变量的拷贝）。

JMM
![此处输入图片的描述][1]

两条规定：
> * 线程对共享变量的读写只能在自己的工作内存中进行，不能直接从主内存中读写。
> * 不同线程之间无法直接访问其他线程的工作内存中的变量，线程间变量值得传递需要通过主内存进行。

共享变量可见性实现的原理：

线程１对共享变量的修改，需要被线程２及时的看到，必须要经过以下两个步骤：
> * 把工作内存１中更新过的共享变量，刷新到主内存中。
> * 将主内存中的值更新到线程２的工作内存中。

要实现共享变量的可见性，必须保证两点：
> * 线程修改后的工作变量能够及时的刷新到主内存中。
> * 其他线程能及时的把主内存中共享变量的最新值更新到自己的工作内存中。


## 可见性的实现方式

java语言层面支持的可见性的实现方式：
> * synchronized
> * volatile

### synchronized实现可见性

synchronized能够实现：
> * 原子性（通过同步实现）
> * 可见性

JMM中关于synchronized的两条规定：
> * 线程解锁前必须把共享变量的最新值刷新到主内存中。
> * 线程加锁的时候，将清空工作内存中共享变量的值，从而在使用共享变量的时候，需要从主内存中重新读取最新的共享变量值。（注意：加锁与解锁需要的是同一把锁。）

线程执行互斥代码的过程：
> * 获得互斥锁。
> * 清空工作内存。
> * 从主内存中拷贝变量的最新副本到工作内存中。
> * 执行代码。
> * 把共享变量的最新值刷新到主内存中。
> * 释放互斥锁。

重排序

代码书写的顺序与实际执行顺序不同，指令重排序是编译器或者处理器为了提高程序性能做出的优化。（编译成机器码后，重新调整下顺序，可能更符合CPU的特点，能最大限度的发挥CPU的性能）。

当前重排序的类型：
> * 编译器优化的重排序（编译器优化）
> * 指令集并行的重排序（处理器优化）
> * 内存系统的重排序（处理器优化）

![此处输入图片的描述][2]

as-if-serial语义

无论如何进行重排序，必须保证重排序的执行结果跟顺序执行的结果是一致的。（Java编译器，运行时和处理器都会保证程序在单线程下遵循as-if-serial语义）。

![此处输入图片的描述][3]


synchronized的示例代码：

```
public class SynchronizedDemo {
	//共享变量
    private boolean ready = false;
    private int result = 0;
    private int number = 1;   
    //写操作
    public void write(){
    	ready = true;	      				 //1.1				
    	number = 2;		                    //1.2			    
    }
    //读操作
    public void read(){			   	 
    	if(ready){						     //2.1
    		result = number*3;	 	//2.2
    	}   	
    	System.out.println("result的值为：" + result);
    }

    //内部线程类
    private class ReadWriteThread extends Thread {
    	//根据构造方法中传入的flag参数，确定线程执行读操作还是写操作
    	private boolean flag;
    	public ReadWriteThread(boolean flag){
    		this.flag = flag;
    	}
        @Override                                                                    
        public void run() {
        	if(flag){
        		//构造方法中传入true，执行写操作
        		write();
        	}else{
        		//构造方法中传入false，执行读操作
        		read();
        	}
        }
    }

    public static void main(String[] args)  {
    	SynchronizedDemo synDemo = new SynchronizedDemo();
    	//启动线程执行写操作
    	synDemo .new ReadWriteThread(true).start();
    	try {
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
    	//启动线程执行读操作
    	synDemo.new ReadWriteThread(false).start();
    }
}
```
[源文件地址][4]

执行结果：

执行顺序：1.1-2.1-2.2-1.2（写线程执行完1.1后让出CPU资源，读线程开始执行，执行完后，写线程重新开始执行）。

```
1
```
执行顺序：1.2-2.1-2.2-1.1（写线程执行完1.2后让出CPU资源，读线程开始执行，执行完后，写线程重新开始执行）（存在很多情况使值为0）。
```
0
```
执行顺序：1.1-1.2-2.1-1.2 或者：1.1-2.1-1.2-2.2 或者1.2-1.1-2.1-2.2。
```
6
```
说明结果及时的刷新进了主内存，之所以没加synchronized也可以，这是因为，如果加了synchronized通过两条规范肯定会刷新进主内存，但是如果不加也可能刷新进主内存，但不保证一定会刷新进去。






2.1跟2.2可能重排序，禁止重排序的条件是具备数据依赖关系。

导致共享变量在线程间不可见的原因：
> * 线程的交叉执行
> * 重排序结合线程交叉执行
> * 共享变量更新后的值没有在工作内存跟主内存之间及时的更新 

安全代码
```
//写操作
    public synchronized void write(){
    	ready = true;	      				 //1.1				
    	number = 2;		                    //1.2			    
    }
    //读操作
    public synchronized void read(){			   	 
    	if(ready){						     //2.1
    		result = number*3;	 	//2.2
    	}   	
    	System.out.println("result的值为：" + result);
    }
```
![此处输入图片的描述][5]


### volatile实现可见性

volatile关键字：
> * 能保证volatile变量的可见性
> * 不能保证volatile变量复合操作的原子性

volatile关键字如何实现可见性：

深入来说：通过加入内存屏障和禁止重排序优化来实现的。
> * 对volatile变量执行写操作的时候，会在写操作后加入一条store屏障指令
> * 对volatile变量执行读操作时候，会在读操作前加入一条load屏障指令

volatile变量读写过程：

线程写volatile变量的过程：
> * 改变工作内存中volatile变量副本的值
> * 将改变改变后的副本的值从工作内存刷新到主内存

线程读volatile变量的过程：

> * 从主内存中读取volatile变量的最新值加载到工作内存中
> * 从工作内存中读取volatile的最新值

volatile关键字不能保证原子性：

![此处输入图片的描述][6]

分析代码：
```
public class VolatileDemo {


	private int number = 0;
	
	public int getNumber(){
		return this.number;
	}
	
	public void increase(){
	    this.number++;
	}
	
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		final VolatileDemo volDemo = new VolatileDemo();
		for(int i = 0 ; i < 500 ; i++){
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					volDemo.increase();
				}
			}).start();
		}
		
		//如果还有子线程在运行，主线程就让出CPU资源，
		//直到所有的子线程都运行完了，主线程再继续往下执行
		while(Thread.activeCount() > 1){
			Thread.yield();
		}
		
		System.out.println("number : " + volDemo.getNumber());
	}

}

```
打印结果：可能会出现小于500的情况。

原因分析：volatile不能保证原子性。

number=5
线程A读出操作
线程B读取操作
线程B加一操作
线程B写入最新的number值
此时，主内存值为6，线程B的工作内存值为6，线程A的工作内存值为5。
线程A执行加一操作
线程A写入最新的number值
虽然进行了两次加一操作，但第二次的结果，覆盖了第一次的结果，所以小于500情况出现了。


保证number自增的原子性：
> * synchronized
> * ReentrantLock
> * AtomicInterger


```
public class VolatileDemo {

	private Lock lock = new ReentrantLock();
	private int number = 0;
	
	public int getNumber(){
		return this.number;
	}
	
	public void increase(){
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		lock.lock();
		try {
			this.number++;
		} finally {
			lock.unlock();
		}
	}
	
	/**
	 * @param args
	 */
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		final VolatileDemo volDemo = new VolatileDemo();
		for(int i = 0 ; i < 500 ; i++){
			new Thread(new Runnable() {
				
				@Override
				public void run() {
					volDemo.increase();
				}
			}).start();
		}
		
		//如果还有子线程在运行，主线程就让出CPU资源，
		//直到所有的子线程都运行完了，主线程再继续往下执行
		while(Thread.activeCount() > 1){
			Thread.yield();
		}
		
		System.out.println("number : " + volDemo.getNumber());
	}

}

```
[源码地址][7]

volatile使用注意事项、

要在多线程的环境中安全的使用volatile变量，必须同时满足：
> * 对变量的写入操作，不依赖于当前的值
不满足：number++,count=count*5;
满足：布尔变量，记录温度变化的变量等；
> * 该变量没有包含在具有其他变量的不变式中
不满足：不变式 low < up

以上两个条件满足的情况不多，所以volatile的使用要明显的比synchronized少的多。

synchronized跟volatile的比较：

> * volatile不需要加锁，比synchronized更加轻量级，不会阻塞线程；
> * 从内存可见性角度，volatile读操作相当于加锁，volatile写操作相当于解锁；
> * synchronized既能保证可见性，又能保证原子性，而volatile只能保证可见性，不能保证原子性。

## 课程总结：

什么是内存可见性
Java内存模型（JMM）
实现可见性的方式：

> * synchronized
> * volatile
> * final也可以

synchronized和volatile实现内存可见性的原理

synchronized实现可见性
> * 指令重排序
> * as-if-serial语义

volatile实现可见性

> * 能保证可见性
> * 不能保证原子性
> * 注意事项


问：即使没有保证可见性的措施，很多时候共享变量依然能够在主内存跟工作内存间得到及时的更新？

>  答：一般只有在短时间内高并发的情况下才会出现变量得不到及时更新的情况，因为CPU在执行时侯会很快的刷新缓存，所以一般情况下很难看到这种问题。

对64位（long,double）变量的读写可能不是原子操作：
>  java内存模型允许JVM将没有被volatile修饰的64为数据类型的读写操作划分为两次32位的读写操作
导致问题：有可能会出现读取到"半个变量"的情况
解决方法：加volatile关键字

synchronized跟volatile比较：

> * volatile比synchronized更轻量级
> * volatile没有synchronized使用广泛



[细说Java多线程之内存可见性视频链接][13]


  [1]: http://946898963.github.io/2016/06/20/%E7%BB%86%E8%AF%B4Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B9%8B%E5%86%85%E5%AD%98%E5%8F%AF%E8%A7%81%E6%80%A7-%E6%85%95%E8%AF%BE%E7%BD%91%E8%A7%86%E9%A2%91%E7%AC%94%E8%AE%B0/neicunmoxing.png
  [2]: http://946898963.github.io/2016/06/20/%E7%BB%86%E8%AF%B4Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B9%8B%E5%86%85%E5%AD%98%E5%8F%AF%E8%A7%81%E6%80%A7-%E6%85%95%E8%AF%BE%E7%BD%91%E8%A7%86%E9%A2%91%E7%AC%94%E8%AE%B0/chongpaixu.png
  [3]: http://946898963.github.io/2016/06/20/%E7%BB%86%E8%AF%B4Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B9%8B%E5%86%85%E5%AD%98%E5%8F%AF%E8%A7%81%E6%80%A7-%E6%85%95%E8%AF%BE%E7%BD%91%E8%A7%86%E9%A2%91%E7%AC%94%E8%AE%B0/as-if-serialt.png
  [4]: http://img.mukewang.com/down/553ee13d0001c04100000000.rar
  [5]: http://946898963.github.io/2016/06/20/%E7%BB%86%E8%AF%B4Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B9%8B%E5%86%85%E5%AD%98%E5%8F%AF%E8%A7%81%E6%80%A7-%E6%85%95%E8%AF%BE%E7%BD%91%E8%A7%86%E9%A2%91%E7%AC%94%E8%AE%B0/yuanyinfenxi.png
  [6]: http://946898963.github.io/2016/06/20/%E7%BB%86%E8%AF%B4Java%E5%A4%9A%E7%BA%BF%E7%A8%8B%E4%B9%8B%E5%86%85%E5%AD%98%E5%8F%AF%E8%A7%81%E6%80%A7-%E6%85%95%E8%AF%BE%E7%BD%91%E8%A7%86%E9%A2%91%E7%AC%94%E8%AE%B0/bunengyuanzixing.png
  [7]: http://img.mukewang.com/down/553ee1620001abf200000000.rar
[13]: http://www.imooc.com/learn/352
