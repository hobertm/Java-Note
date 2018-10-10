
# 并发编程  

- 线程基础  

线程安全概念：当多个线程访问某一个类（对象或方法）时，这个对象始终都能表现出正确的行为，  
那么这个类（对象或方法）就是线程安全的。  
synchronized：可以在任意对象及方法上加锁，而加锁的这段代码称为"互斥区"或"临界区"  

```java
package com.bjsxt.base.sync001;


public class MyThread extends Thread{
	
	private int count = 5 ;
	
	//synchronized加锁
	public void run(){
		count--; 
		System.out.println(this.currentThread().getName() + " count = "+ count);
	}
	
	public static void main(String[] args) {
		/**
		 * 分析：当多个线程访问myThread的run方法时，以排队的方式进行处理（这里排对是按照CPU分配的先后顺序而定的），
		 * 		一个线程想要执行synchronized修饰的方法里的代码：
		 * 		1 尝试获得锁
		 * 		2 如果拿到锁，执行synchronized代码体内容；拿不到锁，这个线程就会不断的尝试获得这把锁，直到拿到为止，
		 * 		   而且是多个线程同时去竞争这把锁。（也就是会有锁竞争的问题）
		 */
		MyThread myThread = new MyThread();
		Thread t1 = new Thread(myThread,"t1");
		Thread t2 = new Thread(myThread,"t2");
		Thread t3 = new Thread(myThread,"t3");
		Thread t4 = new Thread(myThread,"t4");
		Thread t5 = new Thread(myThread,"t5");
		t1.start();
		t2.start();
		t3.start();
		t4.start();
		t5.start();
	}
}

/*运行结果：
t1 count = 2
t5 count = 0
t4 count = 1
t3 count = 2
t2 count = 2

解决方法在void前面加锁  

*/


```

多个线程多把锁  

```java
package com.bjsxt.base.sync002;
/**
 * 关键字synchronized取得的锁都是对象锁，而不是把一段代码（方法）当做锁，
 * 所以代码中哪个线程先执行synchronized关键字的方法，哪个线程就持有该方法所属对象的锁（Lock），
 * 
 * 在静态方法上加synchronized关键字，表示锁定.class类，类一级别的锁（独占.class类）。
 *
 */
public class MultiThread {

	//这个地方一直就应该有static，否则出现不了线程不安全
	private static int num = 0;
	
	/** static */
	public synchronized void printNum(String tag){
		try {
			
			if(tag.equals("a")){
				num = 100;
				System.out.println("tag a, set num over!");
				Thread.sleep(1000);
			} else {
				num = 200;
				System.out.println("tag b, set num over!");
			}
			
			System.out.println("tag " + tag + ", num = " + num);
			
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	//注意观察run方法输出顺序
	public static void main(String[] args) {
		
		//俩个不同的对象
		final MultiThread m1 = new MultiThread();
		final MultiThread m2 = new MultiThread();
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				m1.printNum("a");
			}
		});
		
		Thread t2 = new Thread(new Runnable() {
			@Override 
			public void run() {
				m2.printNum("b");
			}
		});		
		
		t1.start();
		t2.start();
		
	}
	
	
}

/*
tag b, set num over!
tag a, set num over!
tag b, num = 100
tag a, num = 100

原因t1和t2拿到的是不同的锁m1和m2  

解决方法：
加static
在静态方法上加synchronized关键字，表示锁定.class类，类一级别的锁（独占.class类）。

*/

```

对象锁的同步和异步问题  

```java
package com.bjsxt.base.sync003;

/**
 * 对象锁的同步和异步问题
 *
 */
public class MyObject {

	public synchronized void method1(){
		try {
			System.out.println(Thread.currentThread().getName());
			Thread.sleep(4000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	/** synchronized */
	public void method2(){
			System.out.println(Thread.currentThread().getName());
	}
	
	public static void main(String[] args) {
		
		final MyObject mo = new MyObject();
		
		/**
		 * 分析：
		 * t1线程先持有object对象的Lock锁，t2线程可以以异步的方式调用对象中的非synchronized修饰的方法
		 * t1线程先持有object对象的Lock锁，t2线程如果在这个时候调用对象中的同步（synchronized）方法则需等待，也就是同步
		 */
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				mo.method1();
			}
		},"t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				mo.method2();
			}
		},"t2");
		
		t1.start();
		t2.start();
		
	}
	
}
/*
结果：
t1
t2

如果synchronized修饰method2，就会导致t2在4秒后打印  
因为获得的都是mo对象的锁

*/
```

- 脏读  

```java
package com.bjsxt.base.sync004;
/**
 * 业务整体需要使用完整的synchronized，保持业务的原子性。
 *
 */
public class DirtyRead {

	private String username = "bjsxt";
	private String password = "123";
	
	public synchronized void setValue(String username, String password){
		this.username = username;
		
		try {
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		this.password = password;
		
		System.out.println("setValue最终结果：username = " + username + " , password = " + password);
	}
	
	public void getValue(){
		System.out.println("getValue方法得到：username = " + this.username + " , password = " + this.password);
	}
	
	
	public static void main(String[] args) throws Exception{
		
		final DirtyRead dr = new DirtyRead();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				dr.setValue("z3", "456");		
			}
		});
		t1.start();
		Thread.sleep(1000);
		
		dr.getValue();
	}
}

/*结果：
getValue方法得到：username = z3 , password = 123
setValue最终结果：username = z3 , password = 456

在get和set方法都应该加上同步  

*/
```

- 同步的细节  
锁重入  

```java
package com.bjsxt.base.sync005;
/**
 * synchronized的重入
 *
 */
public class SyncDubbo1 {

	public synchronized void method1(){
		System.out.println("method1..");
		method2();
	}
	public synchronized void method2(){
		System.out.println("method2..");
		method3();
	}
	public synchronized void method3(){
		System.out.println("method3..");
	}
	
	public static void main(String[] args) {
		final SyncDubbo1 sd = new SyncDubbo1();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				sd.method1();
			}
		});
		t1.start();
	}
}

/*结果：
method1..
method2..
method3..

这样是安全的

*/

```

子父类之间的同步  

```java
package com.bjsxt.base.sync005;
/**
 * synchronized的重入
 *
 */
public class SyncDubbo2 {

	static class Main {
		public int i = 10;
		public synchronized void operationSup(){
			try {
				i--;
				System.out.println("Main print i = " + i);
				Thread.sleep(100);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	static class Sub extends Main {
		public synchronized void operationSub(){
			try {
				while(i > 0) {
					i--;
					System.out.println("Sub print i = " + i);
					Thread.sleep(100);		
					this.operationSup();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				Sub sub = new Sub();
				sub.operationSub();
			}
		});
		
		t1.start();
	}

/*结果：
Sub print i = 9
Main print i = 8
Sub print i = 7
Main print i = 6
Sub print i = 5
Main print i = 4
Sub print i = 3
Main print i = 2
Sub print i = 1
Main print i = 0

都加了同步是安全的

*/

}
```

synchronized异常  

```java
package com.bjsxt.base.sync005;
/**
 * synchronized异常
 *
 */
public class SyncException {

	private int i = 0;
	public synchronized void operation(){
		while(true){
			try {
				i++;
				Thread.sleep(100);
				System.out.println(Thread.currentThread().getName() + " , i = " + i);
				if(i == 20){
					//Integer.parseInt("a");
					throw new RuntimeException();
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		
		final SyncException se = new SyncException();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				se.operation();
			}
		},"t1");
		t1.start();
	}
	
	
}

```

不同类型的锁  

```java
package com.bjsxt.base.sync006;

/**
 * 使用synchronized代码块加锁,比较灵活
 *
 */
public class ObjectLock {

	public void method1(){
		synchronized (this) {	//对象锁
			try {
				System.out.println("do method1..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public void method2(){		//类锁
		synchronized (ObjectLock.class) {
			try {
				System.out.println("do method2..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	private Object lock = new Object();
	public void method3(){		//任何对象锁
		synchronized (lock) {
			try {
				System.out.println("do method3..");
				Thread.sleep(2000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	
	public static void main(String[] args) {
		
		final ObjectLock objLock = new ObjectLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method1();
			}
		});
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method2();
			}
		});
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				objLock.method3();
			}
		});
		
		t1.start();
		t2.start();
		t3.start();
		
		
	}
	
}

```

不要用字符串常量加锁  
可以使用new String的方式  

```java
package com.bjsxt.base.sync006;
/**
 * synchronized代码块对字符串的锁，注意String常量池的缓存功能
 *
 */
public class StringLock {

	public void method() {
		//new String("字符串常量")
		synchronized ("字符串常量") {
			try {
				while(true){
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + "开始");
					Thread.sleep(1000);		
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + "结束");
				}
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
		final StringLock stringLock = new StringLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				stringLock.method();
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				stringLock.method();
			}
		},"t2");
		
		t1.start();
		t2.start();
	}
}

```

换了锁，就不能同步了  

```java
package com.bjsxt.base.sync006;
/**
 * 锁对象的改变问题
 *
 */
public class ChangeLock {

	private String lock = "lock";
	
	private void method(){
		synchronized (lock) {
			try {
				System.out.println("当前线程 : "  + Thread.currentThread().getName() + "开始");
				lock = "change lock";
				Thread.sleep(2000);
				System.out.println("当前线程 : "  + Thread.currentThread().getName() + "结束");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	}
	
	public static void main(String[] args) {
	
		final ChangeLock changeLock = new ChangeLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				changeLock.method();
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				changeLock.method();
			}
		},"t2");
		t1.start();
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}
	
}

```

changeAttributte同一对象属性的修改不会影响锁的情况  

```java
package com.bjsxt.base.sync006;
/**
 * 同一对象属性的修改不会影响锁的情况
 *
 */
public class ModifyLock {
	
	private String name ;
	private int age ;
	
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getAge() {
		return age;
	}
	public void setAge(int age) {
		this.age = age;
	}
	
	public synchronized void changeAttributte(String name, int age) {
		try {
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 开始");
			this.setName(name);
			this.setAge(age);
			
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 修改对象内容为： " 
					+ this.getName() + ", " + this.getAge());
			
			Thread.sleep(2000);
			System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 结束");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		final ModifyLock modifyLock = new ModifyLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				modifyLock.changeAttributte("张三", 20);
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				modifyLock.changeAttributte("李四", 21);
			}
		},"t2");
		
		t1.start();
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}
	
}

```

死锁问题  

```java
package com.bjsxt.base.sync006;

/**
 * 死锁问题，在设计程序时就应该避免双方相互持有对方的锁的情况
 *
 */
public class DeadLock implements Runnable{

	private String tag;
	private static Object lock1 = new Object();
	private static Object lock2 = new Object();
	
	public void setTag(String tag){
		this.tag = tag;
	}
	
	@Override
	public void run() {
		if(tag.equals("a")){
			synchronized (lock1) {
				try {
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock1执行");
					Thread.sleep(2000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				synchronized (lock2) {
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock2执行");
				}
			}
		}
		if(tag.equals("b")){
			synchronized (lock2) {
				try {
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock2执行");
					Thread.sleep(2000);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				synchronized (lock1) {
					System.out.println("当前线程 : "  + Thread.currentThread().getName() + " 进入lock1执行");
				}
			}
		}
	}
	
	public static void main(String[] args) {
		
		DeadLock d1 = new DeadLock();
		d1.setTag("a");
		DeadLock d2 = new DeadLock();
		d2.setTag("b");
		 
		Thread t1 = new Thread(d1, "t1");
		Thread t2 = new Thread(d2, "t2");
		 
		t1.start();
		try {
			Thread.sleep(500);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		t2.start();
	}
	

	
}

```

- volatile  
多线程有自己的内存存储成员变量，volatile可以强制读取主存的变量  
拥有多线程间可见性，没有原子性  
性能比同步高很多  

```java
package com.bjsxt.base.sync007;

public class RunThread extends Thread{

	private volatile boolean isRunning = true;
	private void setRunning(boolean isRunning){
		this.isRunning = isRunning;
	}
	
	public void run(){
		System.out.println("进入run方法..");
		int i = 0;
		while(isRunning == true){
			//..
		}
		System.out.println("线程停止");
	}
	
	public static void main(String[] args) throws InterruptedException {
		RunThread rt = new RunThread();
		rt.start();
		Thread.sleep(1000);
		rt.setRunning(false);
		System.out.println("isRunning的值已经被设置了false");
	}
	
	
}

/*
如果不加volatile，发现线程不会停止，因为改的只是主内存的false，线程自己空间里的没改  
加了volatile，就会强制读取主内存中的变量  
*/

```

说明volatile没有原子性  
t1与t2两个线程准备操作它，当t1在自己存储空间内修改完count值之后，并没有及时将count修改回去，  
而是执行了count其它的操作——这时候，t2开始执行该操作，但是它并没有发现count值进行了改变，  
这样就造成了count值没有被及时更新而产生的相关错误。  

```java
package com.bjsxt.base.sync007;

import java.util.concurrent.atomic.AtomicInteger;

/**
 * volatile关键字不具备synchronized关键字的原子性（同步）
 *
 */
public class VolatileNoAtomic extends Thread{
	private static volatile int count;
	//private static AtomicInteger count = new AtomicInteger(0);
	private static void addCount(){
		for (int i = 0; i < 1000; i++) {
			count++ ;

			//count.incrementAndGet();
		}
		System.out.println(count);
	}
	
	public void run(){
		addCount();
	}
	
	public static void main(String[] args) {
		
		VolatileNoAtomic[] arr = new VolatileNoAtomic[100];
		for (int i = 0; i < 10; i++) {
			arr[i] = new VolatileNoAtomic();
		}
		
		for (int i = 0; i < 10; i++) {
			arr[i].start();
		}
	}
	
	
	
	
}


/*结果：
1775
4380
4048
2840
1919
5455
6311
6166
7219
7095

最终结果不为一万，说明volatile没有原子性  
直接把变量定义为原子类型，注释中的类型  
*/
```


- 多线程通信  
wait，notify必须配合synchronized  
wait释放锁  
nofity不释放  


引入例子  
不好在线程二一直在轮训判断  

```java
package com.bjsxt.base.conn008;

import java.util.ArrayList;
import java.util.List;

public class ListAdd1 {

	
	private volatile static List list = new ArrayList();	
	
	public void add(){
		list.add("bjsxt");
	}
	public int size(){
		return list.size();
	}
	
	public static void main(String[] args) {
		
		final ListAdd1 list1 = new ListAdd1();
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					for(int i = 0; i <10; i++){
						list1.add();
						System.out.println("当前线程：" + Thread.currentThread().getName() + "添加了一个元素..");
						Thread.sleep(500);
					}	
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				while(true){
					if(list1.size() == 5){
						System.out.println("当前线程收到通知：" + Thread.currentThread().getName() + " list size = 5 线程停止..");
						throw new RuntimeException();
					}
				}
			}
		}, "t2");		
		
		t1.start();
		t2.start();
	}
	
	
}
/*
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
Exception in thread "t2" 当前线程收到通知：t2 list size = 5 线程停止..
java.lang.RuntimeException
	at com.bjsxt.base.conn008.ListAdd1$2.run(ListAdd1.java:43)
	at java.lang.Thread.run(Thread.java:748)
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
*/

```

使用wait，notify的改进  
但是要线程一执行完，不是实时的  
因为notfiy不释放锁，所以要线程一执行完  

```java
package com.bjsxt.base.conn008;

import java.util.ArrayList;
import java.util.List;
import java.util.Queue;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.LinkedBlockingQueue;
/**
 * wait notfiy 方法，wait释放锁，notfiy不释放锁
 *
 */
public class ListAdd2 {
	private volatile static List list = new ArrayList();	
	
	public void add(){
		list.add("bjsxt");
	}
	public int size(){
		return list.size();
	}
	
	public static void main(String[] args) {
		
		final ListAdd2 list2 = new ListAdd2();
		
		// 1 实例化出来一个 lock
		// 当使用wait 和 notify 的时候 ， 一定要配合着synchronized关键字去使用
		final Object lock = new Object();
		
		//final CountDownLatch countDownLatch = new CountDownLatch(1);
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					synchronized (lock) {
						for(int i = 0; i <10; i++){
							list2.add();
							System.out.println("当前线程：" + Thread.currentThread().getName() + "添加了一个元素..");
							Thread.sleep(500);
							if(list2.size() == 5){
								System.out.println("已经发出通知..");
								//countDownLatch.countDown();
								lock.notify();
							}
						}						
					}
				} catch (InterruptedException e) {
					e.printStackTrace();
				}

			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				synchronized (lock) {
					if(list2.size() != 5){
						try {
							System.out.println("t2进入...");
							lock.wait();
							//countDownLatch.await();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
					System.out.println("当前线程：" + Thread.currentThread().getName() + "收到通知线程停止..");
					throw new RuntimeException();
				}
			}
		}, "t2");	
		
		t2.start();
		t1.start();
		
	}
	
}


/*
t2进入...
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
已经发出通知..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t2收到通知线程停止..
Exception in thread "t2" java.lang.RuntimeException
	at com.bjsxt.base.conn008.ListAdd2$2.run(ListAdd2.java:71)
	at java.lang.Thread.run(Thread.java:748)
	*/
```

使用countDownLatch的countDown和await  
不需要配合synchronized使用，解决notify不释放锁的问题   

```java
package com.bjsxt.base.conn008;

import java.util.ArrayList;
import java.util.List;
import java.util.Queue;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.LinkedBlockingDeque;
import java.util.concurrent.LinkedBlockingQueue;
/**
 * wait notfiy 方法，wait释放锁，notfiy不释放锁
 *
 */
public class ListAdd2 {
	private volatile static List list = new ArrayList();	
	
	public void add(){
		list.add("bjsxt");
	}
	public int size(){
		return list.size();
	}
	
	public static void main(String[] args) {
		
		final ListAdd2 list2 = new ListAdd2();
		
		// 1 实例化出来一个 lock
		// 当使用wait 和 notify 的时候 ， 一定要配合着synchronized关键字去使用
		//final Object lock = new Object();
		
		final CountDownLatch countDownLatch = new CountDownLatch(1);
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					//synchronized (lock) {
						for(int i = 0; i <10; i++){
							list2.add();
							System.out.println("当前线程：" + Thread.currentThread().getName() + "添加了一个元素..");
							Thread.sleep(500);
							if(list2.size() == 5){
								System.out.println("已经发出通知..");
								countDownLatch.countDown();
								//lock.notify();
							}
						}						
					//}
				} catch (InterruptedException e) {
					e.printStackTrace();
				}

			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				//synchronized (lock) {
					if(list2.size() != 5){
						try {
							//System.out.println("t2进入...");
							//lock.wait();
							countDownLatch.await();
						} catch (InterruptedException e) {
							e.printStackTrace();
						}
					}
					System.out.println("当前线程：" + Thread.currentThread().getName() + "收到通知线程停止..");
					throw new RuntimeException();
				//}
			}
		}, "t2");	
		
		t2.start();
		t1.start();
		
	}
	
}

/*
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
已经发出通知..
Exception in thread "t2" 当前线程：t1添加了一个元素..
当前线程：t2收到通知线程停止..
java.lang.RuntimeException
	at com.bjsxt.base.conn008.ListAdd2$2.run(ListAdd2.java:71)
	at java.lang.Thread.run(Thread.java:748)
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..
当前线程：t1添加了一个元素..

*/

```

- queue模拟，使用wait和notify  

```java
package com.bjsxt.base.conn009;

import java.util.LinkedList;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class MyQueue {
	
	//1 需要一个承装元素的集合 
	private LinkedList<Object> list = new LinkedList<Object>();
	
	//2 需要一个计数器
	private AtomicInteger count = new AtomicInteger(0);
	
	//3 需要制定上限和下限
	private final int minSize = 0;
	
	private final int maxSize ;
	
	//4 构造方法
	public MyQueue(int size){
		this.maxSize = size;
	}
	
	//5 初始化一个对象 用于加锁
	private final Object lock = new Object();
	
	
	//put(anObject): 把anObject加到BlockingQueue里,如果BlockQueue没有空间,则调用此方法的线程被阻断，直到BlockingQueue里面有空间再继续.
	public void put(Object obj){
		synchronized (lock) {
			while(count.get() == this.maxSize){
				try {
					lock.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			//1 加入元素
			list.add(obj);
			//2 计数器累加
			count.incrementAndGet();
			//3 通知另外一个线程（唤醒）
			lock.notify();
			System.out.println("新加入的元素为:" + obj);
		}
	}
	
	
	//take: 取走BlockingQueue里排在首位的对象,若BlockingQueue为空,阻断进入等待状态直到BlockingQueue有新的数据被加入.
	public Object take(){
		Object ret = null;
		synchronized (lock) {
			while(count.get() == this.minSize){
				try {
					lock.wait();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
			//1 做移除元素操作
			ret = list.removeFirst();
			//2 计数器递减
			count.decrementAndGet();
			//3 唤醒另外一个线程
			lock.notify();
		}
		return ret;
	}
	
	public int getSize(){
		return this.count.get();
	}
	
	
	public static void main(String[] args) {
		
		final MyQueue mq = new MyQueue(5);
		mq.put("a");
		mq.put("b");
		mq.put("c");
		mq.put("d");
		mq.put("e");
		
		System.out.println("当前容器的长度:" + mq.getSize());
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				mq.put("f");
				mq.put("g");
			}
		},"t1");
		
		t1.start();
		
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				Object o1 = mq.take();
				System.out.println("移除的元素为:" + o1);
				Object o2 = mq.take();
				System.out.println("移除的元素为:" + o2);
			}
		},"t2");
		
		
		try {
			TimeUnit.SECONDS.sleep(2);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		
		t2.start();
		
		
	}
}

```

ThreadLocal的使用  
th与当前线程绑定  
t1:张三  
t2:null  

```java
package com.bjsxt.base.conn010;

public class ConnThreadLocal {

	public static ThreadLocal<String> th = new ThreadLocal<String>();
	
	public void setTh(String value){
		th.set(value);
	}
	public void getTh(){
		System.out.println(Thread.currentThread().getName() + ":" + this.th.get());
	}
	
	public static void main(String[] args) throws InterruptedException {
		
		final ConnThreadLocal ct = new ConnThreadLocal();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				ct.setTh("张三");
				ct.getTh();
			}
		}, "t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					Thread.sleep(1000);
					ct.getTh();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}, "t2");
		
		t1.start();
		t2.start();
	}
	
}

```

单例恶汉式  

```java
package com.bjsxt.base.conn011;

public class Singleton {
	
	private Singleton(){
	}
	private static Singleton single = new Singleton();
	
	public static Singleton getInstance(){
		return single;
	}
	
}


```

单例懒汉式  
两次判断  

```java
package com.bjsxt.base.conn011;

public class DubbleSingleton {

	private static DubbleSingleton ds;
	
	public  static DubbleSingleton getDs(){
		if(ds == null){
			try {
				//模拟初始化对象的准备时间...
				Thread.sleep(3000);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
			synchronized (DubbleSingleton.class) {
				if(ds == null){
					ds = new DubbleSingleton();
				}
			}
		}
		return ds;
	}
	
	public static void main(String[] args) {
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t2");
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				System.out.println(DubbleSingleton.getDs().hashCode());
			}
		},"t3");
		
		t1.start();
		t2.start();
		t3.start();
	}
	
}

```

- 同步类容器和并发容器  

同步类容器如vector和hashtable，在迭代时，另一个线程去修改会抛异常  
同步类容器降低了并发性和吞吐量  
下面的用法会返回线程安全的map  
Map<String, String> map = Collections.synchronizedMap(new HashMap<String, String>());  




- 并发容器介绍  

ConcurrentHashMap的锁分段技术  

HashTable容器在竞争激烈的并发环境下表现出效率低下的原因，是因为所有访问  
HashTable的线程都必须竞争同一把锁，那假如容器里有多把锁，每一把锁用于锁  
容器其中一部分数据，那么当多线程访问容器里不同数据段的数据时，线程间就不  
会存在锁竞争，从而可以有效的提高并发访问效率，这就是ConcurrentHashMap所  
使用的锁分段技术，首先将数据分成一段一段的存储，然后给每一段数据配一把锁，  
当一个线程占用锁访问其中一个段数据的时候，其他段的数据也能被其他线程访问。  

什么是CopyOnWrite容器  

CopyOnWrite容器即写时复制的容器。通俗的理解是当我们往一个容器添加元素的时候，  
不直接往当前容器添加，而是先将当前容器进行Copy，复制出一个新的容器，然后新的  
容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。这样做的好处是  
我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任  
何元素。所以CopyOnWrite容器也是一种读写分离的思想，读和写不同的容器。  





- 并发Queue  
高性能队列ConcurrentLinkedQueue  
阻塞队列BlockingQueue  


ConcurrentLinkedQueue
非阻塞并发效率高  
offer和add没有区别  
poll和peek都是取元素，poll会删除取出元素  

```java
		//高性能无阻塞无界队列：ConcurrentLinkedQueue
		/*
		ConcurrentLinkedQueue<String> q = new ConcurrentLinkedQueue<String>();
		q.offer("a");
		q.offer("b");
		q.offer("c");
		q.offer("d");
		q.add("e");
		
		System.out.println(q.poll());	//a 从头部取出元素，并从队列里删除
		System.out.println(q.size());	//4
		System.out.println(q.peek());	//b
		System.out.println(q.size());	//4
		*/
```

BlockingQueue  介绍五种  
1、ArrayBlockingQueue是一个有边界的阻塞队列，它的内部实现是一个数组。  
有边界的意思是它的容量是有限的，我们必须在其初始化的时候指定它的容  
量大小，容量大小一旦指定就不可改变。   

2、LinkedBlockingQueue阻塞队列大小的配置是可选的，如果我们初始化时指  
定一个大小，它就是有边界的，如果不指定，它就是无边界的。说是无边界，其  
实是采用了默认大小为Integer.MAX_VALUE的容量 。它的内部实现是一个链表。 

3、SynchronousQueue队列内部仅允许容纳一个元素。当一个线程插入一个元素后  
会被阻塞，除非这个元素被另一个线程消费。  



```java

		/*
		ArrayBlockingQueue
		add将指定的元素插入到此队列中（如果立即可行且不会违反容量限制），
		在成功时返回 true，如果当前没有可用空间，则抛出 IllegalStateException。
		offer如果此队列已满，则返回 false。
		put将指定元素插入到此队列的尾部，如有必要，则等待空间变得可用。
		*/

		ArrayBlockingQueue<String> array = new ArrayBlockingQueue<String>(5);
		array.put("a");
		array.put("b");
		array.add("c");
		array.add("d");
		array.add("e");
		//array.add("f");
		//System.out.println(array.offer("a", 3, TimeUnit.SECONDS));


		//LinkedBlockingQueue
		//drainTo可以转成collection
		LinkedBlockingQueue<String> q = new LinkedBlockingQueue<String>();
		q.offer("a");
		q.offer("b");
		q.offer("c");
		q.offer("d");
		q.offer("e");
		q.add("f");
		//System.out.println(q.size());
		
/*		for (Iterator iterator = q.iterator(); iterator.hasNext();) {
			String string = (String) iterator.next();
			System.out.println(string);
		}
*/
		
/*		List<String> list = new ArrayList<String>();
		System.out.println(q.drainTo(list, 3));
		System.out.println(list.size());
		for (String string : list) {
			System.out.println(string);
		}
*/


		//SynchronousQueue  
		//要调用add方法，必须有一个线程在take 
		final SynchronousQueue<String> q = new SynchronousQueue<String>();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println(q.take());
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
		t1.start();
		Thread t2 = new Thread(new Runnable() {
			
			@Override
			public void run() {
				q.add("asdasd");
			}
		});
		t2.start();		
	}
```

4、PriorityBlockingQueue  
PriorityBlockingQueue是一个支持优先级的无界队列。默认情况下元素采取自然顺序排列，  
也可以通过比较器comparator来指定元素的排序规则。元素按照升序排列。  

依赖的task类  

```java
package com.bjsxt.base.coll013;

public class Task implements Comparable<Task>{
	
	private int id ;
	private String name;
	public int getId() {
		return id;
	}
	public void setId(int id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	
	@Override
	public int compareTo(Task task) {
		return this.id > task.id ? 1 : (this.id < task.id ? -1 : 0);  
	}
	
	public String toString(){
		return this.id + "," + this.name;
	}
	
}

```

第一次使用take取值后才会排序  

```java
package com.bjsxt.base.coll013;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Iterator;
import java.util.List;
import java.util.concurrent.PriorityBlockingQueue;

public class UsePriorityBlockingQueue {

	
	public static void main(String[] args) throws Exception{
		
		
		PriorityBlockingQueue<Task> q = new PriorityBlockingQueue<Task>();
		
		Task t1 = new Task();
		t1.setId(3);
		t1.setName("id为3");
		Task t2 = new Task();
		t2.setId(4);
		t2.setName("id为4");
		Task t3 = new Task();
		t3.setId(1);
		t3.setName("id为1");
		Task t4 = new Task();
		t4.setId(7);
		t4.setName("id为7");
		Task t5 = new Task();
		t5.setId(6);
		t5.setName("id为6");
		
		//return this.id > task.id ? 1 : 0;
		q.add(t1);	//3
		q.add(t2);	//4
		q.add(t3);  //1
		q.add(t4);  //7
		q.add(t5);  //6
		
		System.out.println("容器：" + q);
		System.out.println(q.take().getId());
		System.out.println("容器：" + q);
//		System.out.println(q.take().getId());
//		System.out.println(q.take().getId());
		
	}
}


```

5、DelayQueue  
DelayQueue是一个支持延时获取元素的无界阻塞队列。队列使用PriorityQueue来实现。  
队列中的元素必须实现Delayed接口，在创建元素时可以指定多久才能从队列中获取当前元素。  
只有在延迟期满时才能从队列中提取元素。  

我们可以将DelayQueue运用在以下应用场景：  
缓存系统的设计：可以用DelayQueue保存缓存元素的有效期，使用一个线程循环查询DelayQueue，  
一旦能从DelayQueue中获取元素时，表示缓存有效期到了。  
定时任务调度。使用DelayQueue保存当天将会执行的任务和执行时间，一旦从DelayQueue中获取到  
任务就开始执行，从比如TimerQueue就是使用DelayQueue实现的。  
队列中的Delayed必须实现compareTo来指定元素的顺序。比如让延时时间最长的放在队列的末尾。  


延迟队列的例子  
网民类  
实现Delayed  
重写getDelay和compareTo  

```java
package com.bjsxt.base.coll013;

import java.util.concurrent.Delayed;
import java.util.concurrent.TimeUnit;

public class Wangmin implements Delayed {  
    
    private String name;  
    //身份证  
    private String id;  
    //截止时间  
    private long endTime;  
    //定义时间工具类
    private TimeUnit timeUnit = TimeUnit.SECONDS;
      
    public Wangmin(String name,String id,long endTime){  
        this.name=name;  
        this.id=id;  
        this.endTime = endTime;  
    }  
      
    public String getName(){  
        return this.name;  
    }  
      
    public String getId(){  
        return this.id;  
    }  
      
    /** 
     * 用来判断是否到了截止时间 
     */  
    @Override  
    public long getDelay(TimeUnit unit) { 
        //return unit.convert(endTime, TimeUnit.MILLISECONDS) - unit.convert(System.currentTimeMillis(), TimeUnit.MILLISECONDS);
    	return endTime - System.currentTimeMillis();
    }  
  
    /** 
     * 相互批较排序用 
     */  
    @Override  
    public int compareTo(Delayed delayed) {  
    	Wangmin w = (Wangmin)delayed;  
        return this.getDelay(this.timeUnit) - w.getDelay(this.timeUnit) > 0 ? 1:0;  
    }  
  
}  
```

网吧类  

```java
package com.bjsxt.base.coll013;

import java.util.concurrent.DelayQueue;

public class WangBa implements Runnable {  
    
    private DelayQueue<Wangmin> queue = new DelayQueue<Wangmin>();  
    
    public boolean yinye =true;  
      
    public void shangji(String name,String id,int money){  
        Wangmin man = new Wangmin(name, id, 1000 * money + System.currentTimeMillis());  
        System.out.println("网名"+man.getName()+" 身份证"+man.getId()+"交钱"+money+"块,开始上机...");  
        this.queue.add(man);  
    }  
      
    public void xiaji(Wangmin man){  
        System.out.println("网名"+man.getName()+" 身份证"+man.getId()+"时间到下机...");  
    }  
  
    @Override  
    public void run() {  
        while(yinye){  
            try {  
                Wangmin man = queue.take();  
                xiaji(man);  
            } catch (InterruptedException e) {  
                e.printStackTrace();  
            }  
        }  
    }  
      
    public static void main(String args[]){  
        try{  
            System.out.println("网吧开始营业");  
            WangBa siyu = new WangBa();  
            Thread shangwang = new Thread(siyu);  
            shangwang.start();  
              
            siyu.shangji("路人甲", "123", 1);  
            siyu.shangji("路人乙", "234", 10);  
            siyu.shangji("路人丙", "345", 5);  
        }  
        catch(Exception e){  
            e.printStackTrace();
        }  
  
    }  
}  
```


- future模式  

什么是Future模型：
该模型是将异步请求和代理模式联合的模型产物。类似商品订单模型。  
客户端发送一个长时间的请求，服务端不需等待该数据处理完成便立  
即返回一个伪造的代理数据（相当于商品订单，不是商品本身），用  
户也无需等待，先去执行其他的若干操作后，再去调用服务器已经完  
成组装的真实数据。该模型充分利用了等待的时间片段。  

- master worker模式  

Master-Worker模式是常用的并行设计模式。它的核心思想是，系统  
有两个进程协议工作：Master进程和Worker进程。Master进程负责接  
收和分配任务，Worker进程负责处理子任务。当各个Worker进程将子任  
务处理完后，将结果返回给Master进程，由Master进行归纳和汇总，  
从而得到系统结果  

- 生产者消费者模式  

生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。  
生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所  
以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消  
费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于  
一个缓冲区，平衡了生产者和消费者的处理能力。
这个阻塞队列就是用来给生产者和消费者解耦的。纵观大多数设计模式，  
都会找一个第三者出来进行解耦，如工厂模式的第三者是工厂类，模板  
模式的第三者是模板类。在学习一些设计模式的过程中，如果先找到这  
个模式的第三者，能帮助我们快速熟悉一个设计模式。


- Java 线程池

Java通过Executors提供四种线程池，分别为：  

newSingleThreadExecutor   
创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。  
newCachedThreadPool  
创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。  
newFixedThreadPool   
创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。  
newScheduledThreadPool   
创建一个定长线程池，支持定时及周期性任务执行。  

```java
package com.bjsxt.height.concurrent017;

class Temp extends Thread {
    public void run() {
        System.out.println("run");
    }
}

public class ScheduledJob {
	
    public static void main(String args[]) throws Exception {
    
    	Temp command = new Temp();
    	//初始化容量
        ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
        //5是初始化延迟，1是下次执行的延迟
        ScheduledFuture<?> scheduleTask = scheduler.scheduleWithFixedDelay(command, 5, 1, TimeUnit.SECONDS);
    
    }
}
```

自定义线程池  

使用有界队列  
二三四会进队列  
第五个会创建新线程  
若队列已满，则在总线程数不大于maximumPoolSize的前提下，创建新的线程  
没有拒绝策略时，第六个会拒绝，并抛异常  

```java
package com.bjsxt.height.concurrent018;

public class UseThreadPoolExecutor1 {


	public static void main(String[] args) {
		/**
		 * 在使用有界队列时，若有新的任务需要执行，如果线程池实际线程数小于corePoolSize，则优先创建线程，
		 * 若大于corePoolSize，则会将任务加入队列，
		 * 若队列已满，则在总线程数不大于maximumPoolSize的前提下，创建新的线程，
		 * 若线程数大于maximumPoolSize，则执行拒绝策略。或其他自定义方式。
		 * 
		 */	
		ThreadPoolExecutor pool = new ThreadPoolExecutor(
				1, 				//coreSize
				2, 				//MaxSize
				60, 			//60
				TimeUnit.SECONDS, 
				new ArrayBlockingQueue<Runnable>(3)			//指定一种队列 （有界队列）
				//new LinkedBlockingQueue<Runnable>()
				//, new MyRejected()
				//, new DiscardOldestPolicy()
				);
		
		MyTask mt1 = new MyTask(1, "任务1");
		MyTask mt2 = new MyTask(2, "任务2");
		MyTask mt3 = new MyTask(3, "任务3");
		MyTask mt4 = new MyTask(4, "任务4");
		MyTask mt5 = new MyTask(5, "任务5");
		MyTask mt6 = new MyTask(6, "任务6");
		
		pool.execute(mt1);
		pool.execute(mt2);
		pool.execute(mt3);
		pool.execute(mt4);
		pool.execute(mt5);
		pool.execute(mt6);
		
		pool.shutdown();
		
	}
}


```

使用无界队列  
LinkedBlockingQueue与有界队列相比，除非系统资源耗尽，否则无界队列不存在任务入队失败的情况，  
若系统的线程数小于corePoolSize时，则新建线程执行corePoolSize，当达到corePoolSize后，则把  
多余的任务放入队列中等待执行若任务的创建和处理的速速差异很大，无界队列会保持快速增长，直到耗  
尽系统内存为之，对于无界队列的线程池maximumPoolSize并无真实用处。  

```java
package com.bjsxt.height.concurrent018;

public class UseThreadPoolExecutor2 implements Runnable{

	private static AtomicInteger count = new AtomicInteger(0);
	
	@Override
	public void run() {
		try {
			int temp = count.incrementAndGet();
			System.out.println("任务" + temp);
			Thread.sleep(2000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) throws Exception{
		//System.out.println(Runtime.getRuntime().availableProcessors());
		BlockingQueue<Runnable> queue = 
				new LinkedBlockingQueue<Runnable>();
				//new ArrayBlockingQueue<Runnable>(10);
		ExecutorService executor  = new ThreadPoolExecutor(
					5, 		//core
					10, 	//max
					120L, 	//2fenzhong
					TimeUnit.SECONDS,
					queue);
		
		for(int i = 0 ; i < 20; i++){
			executor.execute(new UseThreadPoolExecutor2());
		}
		Thread.sleep(1000);
		System.out.println("queue size:" + queue.size());		//10
		Thread.sleep(2000);
	}


}
```

- 拒绝策略  

AbortPolicy
为java线程池默认的阻塞策略，不执行此任务，而且直接抛出一个运行时异常，  
切记ThreadPoolExecutor.execute需要try catch，否则程序会直接退出。  

DiscardPolicy  
直接抛弃，任务不执行，空方法  

DiscardOldestPolicy  
从队列里面抛弃head的一个任务，并再次execute 此task。   

CallerRunsPolicy  
在调用execute的线程里面执行此command，会阻塞入口  

用户自定义拒绝策略  
实现RejectedExecutionHandler，并自己定义策略模式  

```java
public class MyRejected implements RejectedExecutionHandler{

	
	public MyRejected(){
	}
	
	@Override
	public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
		System.out.println("自定义处理..");
		System.out.println("当前被拒绝任务为：" + r.toString());
		

	}

}
```

- Concurrent.util工具类讲解  

- CountDownLatch  
new CountDownLatch(2);需要countdown两次才能唤醒  

```java
package com.bjsxt.height.concurrent019;

import java.util.concurrent.CountDownLatch;

public class UseCountDownLatch {

	public static void main(String[] args) {
		
		final CountDownLatch countDown = new CountDownLatch(2);
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("进入线程t1" + "等待其他线程处理完成...");
					countDown.await();
					System.out.println("t1线程继续执行...");
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		},"t1");
		
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("t2线程进行初始化操作...");
					Thread.sleep(3000);
					System.out.println("t2线程初始化完毕，通知t1线程继续...");
					countDown.countDown();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("t3线程进行初始化操作...");
					Thread.sleep(4000);
					System.out.println("t3线程初始化完毕，通知t1线程继续...");
					countDown.countDown();
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		});
		
		t1.start();
		t2.start();
		t3.start();
	}
}

```

- CyclicBarrier  
new CyclicBarrier(3); 需要三次await  
当三个线程都await了，就可以同时唤醒三个线程  

```java
package com.bjsxt.height.concurrent019;
import java.io.IOException;  
import java.util.Random;  
import java.util.concurrent.BrokenBarrierException;  
import java.util.concurrent.CyclicBarrier;  
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors; 
public class UseCyclicBarrier {

	static class Runner implements Runnable {  
	    private CyclicBarrier barrier;  
	    private String name;  
	    
	    public Runner(CyclicBarrier barrier, String name) {  
	        this.barrier = barrier;  
	        this.name = name;  
	    }  
	    @Override  
	    public void run() {  
	        try {  
	            Thread.sleep(1000 * (new Random()).nextInt(5));  
	            System.out.println(name + " 准备OK.");  
	            barrier.await();  
	        } catch (InterruptedException e) {  
	            e.printStackTrace();  
	        } catch (BrokenBarrierException e) {  
	            e.printStackTrace();  
	        }  
	        System.out.println(name + " Go!!");  
	    }  
	} 
	
    public static void main(String[] args) throws IOException, InterruptedException {  
        CyclicBarrier barrier = new CyclicBarrier(3);  // 3 
        ExecutorService executor = Executors.newFixedThreadPool(3);  
        
        executor.submit(new Thread(new Runner(barrier, "zhangsan")));  
        executor.submit(new Thread(new Runner(barrier, "lisi")));  
        executor.submit(new Thread(new Runner(barrier, "wangwu")));  
  
        executor.shutdown();  
    }  
  
}  
```

- future  
call方法是future具体要去做的任务  
future1和future2新建两个线程，并发地去执行任务  
主程序执行其他业务逻辑  
future.get()获取处理结果  

```java
package com.bjsxt.height.concurrent019;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class UseFuture implements Callable<String>{
	private String para;
	
	public UseFuture(String para){
		this.para = para;
	}
	
	/**
	 * 这里是真实的业务逻辑，其执行可能很慢
	 */
	@Override
	public String call() throws Exception {
		//模拟执行耗时
		Thread.sleep(5000);
		String result = this.para + "处理完成";
		return result;
	}
	
	//主控制函数
	public static void main(String[] args) throws Exception {
		String queryStr = "query";
		//构造FutureTask，并且传入需要真正进行业务逻辑处理的类,该类一定是实现了Callable接口的类
		FutureTask<String> future = new FutureTask<String>(new UseFuture(queryStr));
		
		FutureTask<String> future2 = new FutureTask<String>(new UseFuture(queryStr));
		//创建一个固定线程的线程池且线程数为1,
		ExecutorService executor = Executors.newFixedThreadPool(2);
		//这里提交任务future,则开启线程执行RealData的call()方法执行
		//submit和execute的区别： 第一点是submit可以传入实现Callable接口的实例对象， 第二点是submit方法有返回值
		
		Future f1 = executor.submit(future);		//单独启动一个线程去执行的
		Future f2 = executor.submit(future2);
		System.out.println("请求完毕");
		
		try {
			//这里可以做额外的数据操作，也就是主程序执行其他业务逻辑
			System.out.println("处理实际的业务逻辑...");
			Thread.sleep(1000);
		} catch (Exception e) {
			e.printStackTrace();
		}
		//调用获取数据方法,如果call()方法没有执行完成,则依然会进行等待
		System.out.println("数据：" + future.get());
		System.out.println("数据：" + future2.get());
		
		executor.shutdown();
	}

}

```

- 信号量Semaphore  
设定只有五个线程可以访问 new Semaphore(5);  
获取许可  semp.acquire();  
访问完后，释放  semp.release();  

```java
package com.bjsxt.height.concurrent019;

import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
import java.util.concurrent.Semaphore;  
  
public class UseSemaphore {  
  
    public static void main(String[] args) {  
        // 线程池  
        ExecutorService exec = Executors.newCachedThreadPool();  
        // 只能5个线程同时访问  
        final Semaphore semp = new Semaphore(5);  
        // 模拟20个客户端访问  
        for (int index = 0; index < 20; index++) {  
            final int NO = index;  
            Runnable run = new Runnable() {  
                public void run() {  
                    try {  
                        // 获取许可  
                        semp.acquire();  
                        System.out.println("Accessing: " + NO);  
                        //模拟实际业务逻辑
                        Thread.sleep((long) (Math.random() * 10000));  
                        // 访问完后，释放  
                        semp.release();  
                    } catch (InterruptedException e) {  
                    }  
                }  
            };  
            exec.execute(run);  
        } 
        
        try {
			Thread.sleep(10);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
        
        //System.out.println(semp.getQueueLength());
        
        
        
        // 退出线程池  
        exec.shutdown();  
    }  
  
}  
```


- ReentrantLock  
lock.lock();  
lock.unlock();  

```java
package com.bjsxt.height.lock020;

import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class UseReentrantLock {
	
	private Lock lock = new ReentrantLock();
	
	public void method1(){
		try {
			lock.lock();
			System.out.println("当前线程:" + Thread.currentThread().getName() + "进入method1..");
			Thread.sleep(1000);
			System.out.println("当前线程:" + Thread.currentThread().getName() + "退出method1..");
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			
			lock.unlock();
		}
	}
	
	public void method2(){
		try {
			lock.lock();
			System.out.println("当前线程:" + Thread.currentThread().getName() + "进入method2..");
			Thread.sleep(2000);
			System.out.println("当前线程:" + Thread.currentThread().getName() + "退出method2..");
			Thread.sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		} finally {
			
			lock.unlock();
		}
	}
	
	public static void main(String[] args) {

		final UseReentrantLock ur = new UseReentrantLock();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				ur.method1();
				ur.method2();
			}
		}, "t1");

		t1.start();
		try {
			Thread.sleep(10);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		//System.out.println(ur.lock.getQueueLength());
	}
}
```

- Condition  
await  
signal  

```java
package com.bjsxt.height.lock020;

import java.util.concurrent.CopyOnWriteArrayList;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class UseCondition {

	private Lock lock = new ReentrantLock();
	private Condition condition = lock.newCondition();
	
	public void method1(){
		try {
			lock.lock();
			System.out.println("当前线程：" + Thread.currentThread().getName() + "进入等待状态..");
			Thread.sleep(3000);
			System.out.println("当前线程：" + Thread.currentThread().getName() + "释放锁..");
			condition.await();	// Object wait
			System.out.println("当前线程：" + Thread.currentThread().getName() +"继续执行...");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void method2(){
		try {
			lock.lock();
			System.out.println("当前线程：" + Thread.currentThread().getName() + "进入..");
			Thread.sleep(3000);
			System.out.println("当前线程：" + Thread.currentThread().getName() + "发出唤醒..");
			condition.signal();		//Object notify
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public static void main(String[] args) {
		
		final UseCondition uc = new UseCondition();
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				uc.method1();
			}
		}, "t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				uc.method2();
			}
		}, "t2");
		t1.start();

		t2.start();
	}
}
```

- 读写锁  
ReentrantReadWriteLock  
只有读可以并发进行，写写和读写都是互斥  

```java
package com.bjsxt.height.lock021;

import java.util.concurrent.locks.ReentrantReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock.ReadLock;
import java.util.concurrent.locks.ReentrantReadWriteLock.WriteLock;

public class UseReentrantReadWriteLock {

	private ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();
	private ReadLock readLock = rwLock.readLock();
	private WriteLock writeLock = rwLock.writeLock();
	
	public void read(){
		try {
			readLock.lock();
			System.out.println("当前线程:" + Thread.currentThread().getName() + "进入...");
			Thread.sleep(3000);
			System.out.println("当前线程:" + Thread.currentThread().getName() + "退出...");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			readLock.unlock();
		}
	}
	
	public void write(){
		try {
			writeLock.lock();
			System.out.println("当前线程:" + Thread.currentThread().getName() + "进入...");
			Thread.sleep(3000);
			System.out.println("当前线程:" + Thread.currentThread().getName() + "退出...");
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			writeLock.unlock();
		}
	}
	
	public static void main(String[] args) {
		
		final UseReentrantReadWriteLock urrw = new UseReentrantReadWriteLock();
		
		Thread t1 = new Thread(new Runnable() {
			@Override
			public void run() {
				urrw.read();
			}
		}, "t1");
		Thread t2 = new Thread(new Runnable() {
			@Override
			public void run() {
				urrw.read();
			}
		}, "t2");
		Thread t3 = new Thread(new Runnable() {
			@Override
			public void run() {
				urrw.write();
			}
		}, "t3");
		Thread t4 = new Thread(new Runnable() {
			@Override
			public void run() {
				urrw.write();
			}
		}, "t4");		
		
//		t1.start();
//		t2.start();
		
//		t1.start(); // R 
//		t3.start(); // W
		
		t3.start();
		t4.start();		
	}
}
```


- 锁的优化  
避免死锁  
减小锁的持有时间  
减小锁的粒度  
锁的分离(读写锁)  
无锁操作，如原子操作，volatile  


- disruptor框架  

无锁实现网络queue并发操作，性能远高于 blockingqueue容器，每秒传递600万订单  
可以看作最快的消息框架(轻量级的JMS)，观察者模式的实现，或者事件监听模式的实现  


入门程序流程总结  
通过五个参数创建Disruptor  
连接消费事件方法LongEventHandler  
启动Disruptor  
发布事件LongEventProducer的onData方法  
关闭disruptor以及线程池  


- disruptor的入门程序  
创建disruptor  
第一个参数为工厂类对象，用于创建一个个的LongEvent，LongEvent是实际的消费数据  
第二个参数是缓冲区大小  
第三个参数是线程池，进行disruptor的内部数据接收处理调度  
第四个参数是常量single或multi  
第五个参数是策略，见下面注释，推荐使用YieldingWaitStrategy  

```java
package bhz.base;

public class LongEventMain {

	public static void main(String[] args) throws Exception {
		//创建缓冲池
		ExecutorService  executor = Executors.newCachedThreadPool();
		//创建工厂
		LongEventFactory factory = new LongEventFactory();
		//创建bufferSize ,也就是RingBuffer大小，必须是2的N次方
		int ringBufferSize = 1024 * 1024; // 

		/*
		//BlockingWaitStrategy 是最低效的策略，但其对CPU的消耗最小并且在各种不同部署环境中能提供更加一致的性能表现
		WaitStrategy BLOCKING_WAIT = new BlockingWaitStrategy();
		//SleepingWaitStrategy 的性能表现跟BlockingWaitStrategy差不多，对CPU的消耗也类似，但其对生产者线程的影响最小，适合用于异步日志类似的场景
		WaitStrategy SLEEPING_WAIT = new SleepingWaitStrategy();
		//YieldingWaitStrategy 的性能是最好的，适合用于低延迟的系统。在要求极高性能且事件处理线数小于CPU逻辑核心数的场景中，推荐使用此策略；例如，CPU开启超线程的特性
		WaitStrategy YIELDING_WAIT = new YieldingWaitStrategy();
		*/
		
		//创建disruptor
		Disruptor<LongEvent> disruptor = 
				new Disruptor<LongEvent>(factory, ringBufferSize, executor, ProducerType.SINGLE, new YieldingWaitStrategy());
		// 连接消费事件方法
		disruptor.handleEventsWith(new LongEventHandler());
		
		// 启动
		disruptor.start();
		
		//Disruptor 的事件发布过程是一个两阶段提交的过程：
		//发布事件 
		RingBuffer<LongEvent> ringBuffer = disruptor.getRingBuffer();
		
		LongEventProducer producer = new LongEventProducer(ringBuffer); 
		//LongEventProducerWithTranslator producer = new LongEventProducerWithTranslator(ringBuffer);
		ByteBuffer byteBuffer = ByteBuffer.allocate(8);
		for(long l = 0; l<100; l++){
			byteBuffer.putLong(0, l);
			producer.onData(byteBuffer);
			//Thread.sleep(1000);
		}

		
		disruptor.shutdown();//关闭 disruptor，方法会堵塞，直至所有的事件都得到处理；
		executor.shutdown();//关闭 disruptor 使用的线程池；如果需要的话，必须手动关闭， disruptor 在 shutdown 时不会自动关闭；		
		
	}
}

```

LongEvent  
LongEvent和LongEventFactory用于封装事件，是disruptor创建的必要参数  

```java
package bhz.base;

//http://ifeve.com/disruptor-getting-started/
public class LongEvent { 
    private long value;
    public long getValue() { 
        return value; 
    } 
 
    public void setValue(long value) { 
        this.value = value; 
    } 
} 
```

LongEventFactory  

```java
package bhz.base;

import com.lmax.disruptor.EventFactory;
// 需要让disruptor为我们创建事件，我们同时还声明了一个EventFactory来实例化Event对象。
public class LongEventFactory implements EventFactory { 

    @Override 
    public Object newInstance() { 
        return new LongEvent(); 
    } 
} 
```

LongEventHandler  
事件处理器，事件消费者  

```java
package bhz.base;

import com.lmax.disruptor.EventHandler;

//我们还需要一个事件消费者，也就是一个事件处理器。这个事件处理器简单地把事件中存储的数据打印到终端：
public class LongEventHandler implements EventHandler<LongEvent>  {

	@Override
	public void onEvent(LongEvent longEvent, long l, boolean b) throws Exception {
		System.out.println(longEvent.getValue()); 		
	}

}

```

onData用来发布事件，每调用一次就发布一次事件  

四步固定  
得到下面一个事件槽  
用上面的索引取出一个空的事件用于填充  
获取要通过事件传递的业务数据  
发布事件  

```java
package bhz.base;

import java.nio.ByteBuffer;

import com.lmax.disruptor.RingBuffer;
/**
 * 很明显的是：当用一个简单队列来发布事件的时候会牵涉更多的细节，这是因为事件对象还需要预先创建。
 * 发布事件最少需要两步：获取下一个事件槽并发布事件（发布事件的时候要使用try/finnally保证事件一定会被发布）。
 * 如果我们使用RingBuffer.next()获取一个事件槽，那么一定要发布对应的事件。
 * 如果不能发布事件，那么就会引起Disruptor状态的混乱。
 * 尤其是在多个事件生产者的情况下会导致事件消费者失速，从而不得不重启应用才能会恢复。
 */
public class LongEventProducer {

	private final RingBuffer<LongEvent> ringBuffer;
	
	public LongEventProducer(RingBuffer<LongEvent> ringBuffer){
		this.ringBuffer = ringBuffer;
	}
	
	/**
	 * onData用来发布事件，每调用一次就发布一次事件
	 * 它的参数会用过事件传递给消费者
	 */
	public void onData(ByteBuffer bb){
		//1.可以把ringBuffer看做一个事件队列，那么next就是得到下面一个事件槽
		long sequence = ringBuffer.next();
		try {
			//2.用上面的索引取出一个空的事件用于填充（获取该序号对应的事件对象）
			LongEvent event = ringBuffer.get(sequence);
			//3.获取要通过事件传递的业务数据
			event.setValue(bb.getLong(0));
		} finally {
			//4.发布事件
			//注意，最后的 ringBuffer.publish 方法必须包含在 finally 中以确保必须得到调用；如果某个请求的 sequence 未被提交，将会堵塞后续的发布操作或者其它的 producer。
			ringBuffer.publish(sequence);
		}
	}
}

```

另一种发布方式LongEventProducerWithTranslator  
不用去获取空的槽  

```java
package bhz.base;
public class LongEventProducerWithTranslator {

	//一个translator可以看做一个事件初始化器，publicEvent方法会调用它
	//填充Event
	private static final EventTranslatorOneArg<LongEvent, ByteBuffer> TRANSLATOR = 
			new EventTranslatorOneArg<LongEvent, ByteBuffer>() {
				@Override
				public void translateTo(LongEvent event, long sequeue, ByteBuffer buffer) {
					event.setValue(buffer.getLong(0));
				}
			};
	
	private final RingBuffer<LongEvent> ringBuffer;
	
	public LongEventProducerWithTranslator(RingBuffer<LongEvent> ringBuffer) {
		this.ringBuffer = ringBuffer;
	}
	
	public void onData(ByteBuffer buffer){
		ringBuffer.publishEvent(TRANSLATOR, buffer);
	}

	
}

```

- Disruptor术语说明  

RingBuffer(循环队列): 被看作Disruptor最主要的组件，然而从3.0开始RingBuffer仅仅负责存储  
和更新在Disruptor中流通的数据。对一些特殊的使用场景能够被用户(使用其他数据结构)完全替代。  

Sequence(空的槽): Disruptor使用Sequence来表示一个特殊组件处理的序号。和Disruptor一样，每个消费者(EventProcessor)  
都维持着一个Sequence。大部分的并发代码依赖这些Sequence值的运转，因此Sequence支持多种当前为AtomicLong类的特性。  

Sequencer: 这是Disruptor真正的核心。实现了这个接口的两种生产者（单生产者和多生产者）均实现了所有的并发算法，  
为了在生产者和消费者之间进行准确快速的数据传递。

SequenceBarrier(生产者消费者平衡): 由Sequencer生成，并且包含了已经发布的Sequence的引用，这些的Sequence源于  
Sequencer和一些独立的消费者的Sequence。它包含了决定是否有供消费者来消费的Event的逻辑。  

WaitStrategy(队列满了的等待策略，上面有注释)：决定一个消费者将如何等待生产者将Event置入Disruptor。  

Event(事件)：从生产者到消费者过程中所处理的数据单元。Disruptor中没有代码表示Event，因为它完全是由用户定义的。  

EventProcessor(消费者，需要传入一个EventHandler)：主要事件循环，处理Disruptor中的Event，并且拥有消费者的Sequence。  
它有一个实现类是BatchEventProcessor，包含了event loop有效的实现，并且将回调到一个EventHandler接口的实现对象。 

EventHandler(处理事件的处理器)：由用户实现并且代表了Disruptor中的一个消费者的接口。  

Producer(生产者)：由用户实现，它调用RingBuffer来插入事件(Event)，在Disruptor中没有相应的实现代码，由用户实现。  

WorkProcessor()：确保每个sequence只被一个processor消费，在同一个WorkPool中的处理多个WorkProcessor  
不会消费同样的sequence。 

WorkerPool(类比于EventProcessor，需要WorkHandler)：一个WorkProcessor池，其中WorkProcessor将消费Sequence，  
所以任务可以在实现WorkHandler接口的worker吃间移交  

LifecycleAware：当BatchEventProcessor启动和停止时，于实现这个接口用于接收通知。  




- RingBuffer的使用  

Main1使用EventProcessor处理方式进行消费  

创建RingBuffer  
第一个参数叫EventFactory，从名字上理解就是"事件工厂"  
第二个参数是RingBuffer的大小，它必须是2的指数倍  
第三个参数是没有可用区块的时候的等待策略  

创建SequenceBarrier，sequenceBarrier做一个生产消费平衡  

调用ringBuffer.addGatingSequences  
消费者的位置信息引用注入到生产者，如果只有一个消费者的情况可以省略  

新建Callable接口子类对象相当于是生产者，生产数据放入ringBuffer并发布  

transProcessor是消费者，本例应用的是EventHandler消息处理方式  
需要三个参数ringBuffer, sequenceBarrier, TradeHandler  

```java
package bhz.generate1;

public class Main1 {  
   
	public static void main(String[] args) throws Exception {  
        int BUFFER_SIZE=1024;  
        int THREAD_NUMBERS=4;  
        /* 
         * createSingleProducer创建一个单生产者的RingBuffer， 
         * 第一个参数叫EventFactory，从名字上理解就是"事件工厂"，其实它的职责就是产生数据填充RingBuffer的区块。 
         * 第二个参数是RingBuffer的大小，它必须是2的指数倍 目的是为了将求模运算转为&运算提高效率 
         * 第三个参数是RingBuffer的生产都在没有可用区块的时候(可能是消费者（或者说是事件处理器） 太慢了)的等待策略 
         */  
        final RingBuffer<Trade> ringBuffer = RingBuffer.createSingleProducer(new EventFactory<Trade>() {  
            @Override  
            public Trade newInstance() {  
                return new Trade();  
            }  
        }, BUFFER_SIZE, new YieldingWaitStrategy());  
        
        //创建线程池  
        ExecutorService executors = Executors.newFixedThreadPool(THREAD_NUMBERS);  
        
        //创建SequenceBarrier  
        SequenceBarrier sequenceBarrier = ringBuffer.newBarrier();  
          
        //创建消息处理器  
        BatchEventProcessor<Trade> transProcessor = new BatchEventProcessor<Trade>(  
                ringBuffer, sequenceBarrier, new TradeHandler());  
          
        //这一步的目的就是把消费者的位置信息引用注入到生产者    如果只有一个消费者的情况可以省略 
        ringBuffer.addGatingSequences(transProcessor.getSequence());  
          
        //把消息处理器提交到线程池  
        executors.submit(transProcessor);  
        
        //如果存在多个消费者 那重复执行上面3行代码 把TradeHandler换成其它消费者类  
          
        Future<?> future= executors.submit(new Callable<Void>() {  
            @Override  
            public Void call() throws Exception {  
                long seq;  
                for(int i=0;i<10;i++){  
                    seq = ringBuffer.next();//占个坑 --ringBuffer一个可用区块  
                    ringBuffer.get(seq).setPrice(Math.random()*9999);//给这个区块放入数据 
                    ringBuffer.publish(seq);//发布这个区块的数据使handler(consumer)可见  
                }  
                return null;  
            }  
        }); 
        
        future.get();//等待生产者结束  
        Thread.sleep(1000);//等上1秒，等消费都处理完成  
        transProcessor.halt();//通知事件(或者说消息)处理器 可以结束了（并不是马上结束!!!）  
        executors.shutdown();//终止线程  
    }  
}  
```

TradeHandler  
实现了两种接口因为要做两种消息处理  
实际使用中使用Main1，Main2其中的一种就可以了  

```java
package bhz.generate1;

import java.util.UUID;

import com.lmax.disruptor.EventHandler;
import com.lmax.disruptor.WorkHandler;

public class TradeHandler implements EventHandler<Trade>, WorkHandler<Trade> {  
	  
    @Override  
    public void onEvent(Trade event, long sequence, boolean endOfBatch) throws Exception {  
        this.onEvent(event);  
    }  
  
    @Override  
    public void onEvent(Trade event) throws Exception {  
        //这里做具体的消费逻辑  
        event.setId(UUID.randomUUID().toString());//简单生成下ID  
        System.out.println(event.getId());  
        System.out.println(event.getPrice());//获取生产者生成的价格  
    }  
}  
```


Main2使用WorkerPool消息处理方式  
WorkerPool需要四个参数  
ringBuffer，sequenceBarrier，异常处理器IgnoreExceptionHandler，TradeHandler  
与EventProcessor方式不同的是，workerPool有start方法，不依赖于线程池启动  

```java
package bhz.generate1;
public class Main2 {  
    public static void main(String[] args) throws InterruptedException {  
        int BUFFER_SIZE=1024;  
        int THREAD_NUMBERS=4;  
        
        EventFactory<Trade> eventFactory = new EventFactory<Trade>() {  
            public Trade newInstance() {  
                return new Trade();  
            }  
        };  
        
        RingBuffer<Trade> ringBuffer = RingBuffer.createSingleProducer(eventFactory, BUFFER_SIZE);  
          
        SequenceBarrier sequenceBarrier = ringBuffer.newBarrier();  
          
        ExecutorService executor = Executors.newFixedThreadPool(THREAD_NUMBERS);  
          
        WorkHandler<Trade> handler = new TradeHandler();  

        WorkerPool<Trade> workerPool = new WorkerPool<Trade>(ringBuffer, sequenceBarrier, new IgnoreExceptionHandler(), handler);  
          
        workerPool.start(executor);  
          
        //下面这个生产8个数据
        for(int i=0;i<8;i++){  
            long seq=ringBuffer.next();  
            ringBuffer.get(seq).setPrice(Math.random()*9999);  
            ringBuffer.publish(seq);  
        }  
          
        Thread.sleep(1000);  
        workerPool.halt();  
        executor.shutdown();  
    }  
}  

```



- 在复杂场景下使用Disruptor以及RingBuffer  

Disruptor的创建还是那五个参数  

菱形操作  
希望P1生产的数据给C1、C2并行执行，最后C1、C2执行结束后C3执行  
顺序操作  
六边形操作  

并行执行handleEventsWith(new Handler1(), new Handler2());  
在之后执行handlerGroup.then(new Handler3());  
顺序执行使用链式调用  

```java
package bhz.generate2;
public class Main {  
    public static void main(String[] args) throws InterruptedException {  
       
    	long beginTime=System.currentTimeMillis();  
        int bufferSize=1024;  
        ExecutorService executor=Executors.newFixedThreadPool(8);  

        Disruptor<Trade> disruptor = new Disruptor<Trade>(new EventFactory<Trade>() {  
            @Override  
            public Trade newInstance() {  
                return new Trade();  
            }  
        }, bufferSize, executor, ProducerType.SINGLE, new BusySpinWaitStrategy());  
        
        /*菱形操作
        //使用disruptor创建消费者组C1,C2  
        EventHandlerGroup<Trade> handlerGroup = 
        disruptor.handleEventsWith(new Handler1(), new Handler2());
        //声明在C1,C2完事之后执行JMS消息发送操作 也就是流程走到C3 
        handlerGroup.then(new Handler3());
        */
        
        //顺序操作
        /*
        disruptor.handleEventsWith(new Handler1()).
        	handleEventsWith(new Handler2()).
        	handleEventsWith(new Handler3());
        */
        
        //六边形操作. 
        /*
        Handler1 h1 = new Handler1();
        Handler2 h2 = new Handler2();
        Handler3 h3 = new Handler3();
        Handler4 h4 = new Handler4();
        Handler5 h5 = new Handler5();
        disruptor.handleEventsWith(h1, h2);
        disruptor.after(h1).handleEventsWith(h4);
        disruptor.after(h2).handleEventsWith(h5);
        disruptor.after(h4, h5).handleEventsWith(h3);
        */
        
        
        
        disruptor.start();//启动  
        CountDownLatch latch=new CountDownLatch(1);  
        //生产者准备  
        executor.submit(new TradePublisher(latch, disruptor));
        
        latch.await();//等待生产者完事. 
       
        disruptor.shutdown();  
        executor.shutdown();  
        System.out.println("总耗时:"+(System.currentTimeMillis()-beginTime));  
    }  
}  
```

TradePublisher  
构造方法传入CountDownLatch和Disruptor  
类似于LongEventProducerWithTranslator的发布方式，不需要每次获取空的槽  
生产trade并setPrice，之后发布  

```java
package bhz.generate2;
import java.util.Random;
import java.util.concurrent.CountDownLatch;
import bhz.generate1.Trade;
import com.lmax.disruptor.EventTranslator;
import com.lmax.disruptor.dsl.Disruptor;
public class TradePublisher implements Runnable {  
	
    Disruptor<Trade> disruptor;  
    private CountDownLatch latch;  
    
    private static int LOOP=10;//模拟百万次交易的发生  
  
    public TradePublisher(CountDownLatch latch,Disruptor<Trade> disruptor) {  
        this.disruptor=disruptor;  
        this.latch=latch;  
    }  
  
    @Override  
    public void run() {  
    	TradeEventTranslator tradeTransloator = new TradeEventTranslator();  
        for(int i=0;i<LOOP;i++){  
            disruptor.publishEvent(tradeTransloator);  
        }  
        latch.countDown();  
    }  
      
}  
  
class TradeEventTranslator implements EventTranslator<Trade>{  
    
	private Random random=new Random();  
    
	@Override  
    public void translateTo(Trade event, long sequence) {  
        this.generateTrade(event);  
    }  
    
	private Trade generateTrade(Trade trade){  
        trade.setPrice(random.nextDouble()*9999);  
        return trade;  
    }  
	
}  
```

Handler1  

```java
package bhz.generate2;
import java.util.UUID;
import bhz.generate1.Trade;
import com.lmax.disruptor.EventHandler;
import com.lmax.disruptor.WorkHandler;

public class Handler1 implements EventHandler<Trade>,WorkHandler<Trade> {  
	  
    @Override  
    public void onEvent(Trade event, long sequence, boolean endOfBatch) throws Exception {  
        this.onEvent(event);  
    }  
  
    @Override  
    public void onEvent(Trade event) throws Exception {  
    	System.out.println("handler1: set name");
    	event.setName("h1");
    	Thread.sleep(1000);
    }  
}  
```

Handler2  

```java
package bhz.generate2;
import bhz.generate1.Trade;
import com.lmax.disruptor.EventHandler;

public class Handler2 implements EventHandler<Trade> {  
	  
    @Override  
    public void onEvent(Trade event, long sequence,  boolean endOfBatch) throws Exception {  
    	System.out.println("handler2: set price");
    	event.setPrice(17.0);
    	Thread.sleep(1000);
    }  
      
}  
```

Handler3，其余的Handler见代码  

```java
package bhz.generate2;
import bhz.generate1.Trade;
import com.lmax.disruptor.EventHandler;

public class Handler3 implements EventHandler<Trade> {
    @Override  
    public void onEvent(Trade event, long sequence,  boolean endOfBatch) throws Exception {  
    	System.out.println("handler3: name: " + event.getName() + " , price: " + event.getPrice() + ";  instance: " + event.toString());
    }  
}

```


多个生产者或消费者  



```java
package bhz.multi;
public class Main {
	
	public static void main(String[] args) throws Exception {

		//创建ringBuffer
		RingBuffer<Order> ringBuffer = 
				RingBuffer.create(ProducerType.MULTI, 
						new EventFactory<Order>() {  
				            @Override  
				            public Order newInstance() {  
				                return new Order();  
				            }  
				        }, 
				        1024 * 1024, 
						new YieldingWaitStrategy());
		
		SequenceBarrier barriers = ringBuffer.newBarrier();
		
		Consumer[] consumers = new Consumer[3];
		for(int i = 0; i < consumers.length; i++){
			consumers[i] = new Consumer("c" + i);
		}
		//三个消费者装进workerPool，barriers进行协调  
		WorkerPool<Order> workerPool = 
				new WorkerPool<Order>(ringBuffer, 
						barriers, 
						new IntEventExceptionHandler(),
						consumers);
		//把消费进度设置到ringBuffer中，进行协调  
        ringBuffer.addGatingSequences(workerPool.getWorkerSequences());  
        workerPool.start(Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors()));  
        
        final CountDownLatch latch = new CountDownLatch(1);
        for (int i = 0; i < 100; i++) {  
        	final Producer p = new Producer(ringBuffer);
        	new Thread(new Runnable() {
				@Override
				public void run() {
					try {
						latch.await();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					for(int j = 0; j < 100; j ++){
						p.onData(UUID.randomUUID().toString());
					}
				}
			}).start();
        } 
        Thread.sleep(2000);
        System.out.println("---------------开始生产-----------------");
        latch.countDown();
        Thread.sleep(5000);
        System.out.println("总数:" + consumers[0].getCount() );
	}
	
	static class IntEventExceptionHandler implements ExceptionHandler {  
	    public void handleEventException(Throwable ex, long sequence, Object event) {}  
	    public void handleOnStartException(Throwable ex) {}  
	    public void handleOnShutdownException(Throwable ex) {}  
	} 
}

```

Consumer  

```java
package bhz.multi;

import java.util.concurrent.atomic.AtomicInteger;

import com.lmax.disruptor.WorkHandler;

public class Consumer implements WorkHandler<Order>{
	
	private String consumerId;
	
	private static AtomicInteger count = new AtomicInteger(0);
	
	public Consumer(String consumerId){
		this.consumerId = consumerId;
	}

	@Override
	public void onEvent(Order order) throws Exception {
		System.out.println("当前消费者: " + this.consumerId + "，消费信息：" + order.getId());
		count.incrementAndGet();
	}
	
	public int getCount(){
		return count.get();
	}

}

```

Producer  

```java
package bhz.multi;

import java.nio.ByteBuffer;
import java.util.UUID;

import bhz.base.LongEvent;

import com.lmax.disruptor.EventTranslatorOneArg;
import com.lmax.disruptor.RingBuffer;

public class Producer {

	private final RingBuffer<Order> ringBuffer;
	
	public Producer(RingBuffer<Order> ringBuffer){
		this.ringBuffer = ringBuffer;
	}
	
	/**
	 * onData用来发布事件，每调用一次就发布一次事件
	 * 它的参数会用过事件传递给消费者
	 */
	public void onData(String data){
		//可以把ringBuffer看做一个事件队列，那么next就是得到下面一个事件槽
		long sequence = ringBuffer.next();
		try {
			//用上面的索引取出一个空的事件用于填充（获取该序号对应的事件对象）
			Order order = ringBuffer.get(sequence);
			//获取要通过事件传递的业务数据
			order.setId(data);
		} finally {
			//发布事件
			//注意，最后的 ringBuffer.publish 方法必须包含在 finally 中以确保必须得到调用；如果某个请求的 sequence 未被提交，将会堵塞后续的发布操作或者其它的 producer。
			ringBuffer.publish(sequence);
		}
	}
	
	
}

```


- 网络通信编程  

socket流程  
服务器监听  
客户端请求  
服务端确认连接  
客户端确认连接  

- BIO  
传统bio不能支持太多的客户端进行连接  

```java
package bhz.bio;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;


public class Server {

	final static int PROT = 8765;
	
	public static void main(String[] args) {
		
		ServerSocket server = null;
		try {
			server = new ServerSocket(PROT);
			System.out.println(" server start .. ");
			//进行阻塞
			Socket socket = server.accept();
			//新建一个线程执行客户端的任务
			new Thread(new ServerHandler(socket)).start();
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if(server != null){
				try {
					server.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
			server = null;
		}		
		
	}
	
}

```

bio改进  
添加一个线程池  

```java
package bhz.bio2;

import java.io.BufferedReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

public class Server {

	final static int PORT = 8765;

	public static void main(String[] args) {
		ServerSocket server = null;
		BufferedReader in = null;
		PrintWriter out = null;
		try {
			server = new ServerSocket(PORT);
			System.out.println("server start");
			Socket socket = null;
			//最大线程数50，队列容量是1000
			HandlerExecutorPool executorPool = new HandlerExecutorPool(50, 1000);
			while(true){
				socket = server.accept();
				executorPool.execute(new ServerHandler(socket));
			}
			
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if(in != null){
				try {
					in.close();
				} catch (Exception e1) {
					e1.printStackTrace();
				}
			}
			if(out != null){
				try {
					out.close();
				} catch (Exception e2) {
					e2.printStackTrace();
				}
			}
			if(server != null){
				try {
					server.close();
				} catch (Exception e3) {
					e3.printStackTrace();
				}
			}
			server = null;				
		}
	}
}

```

HandlerExecutorPool  

```java
package bhz.bio2;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class HandlerExecutorPool {

	private ExecutorService executor;
	public HandlerExecutorPool(int maxPoolSize, int queueSize){
		this.executor = new ThreadPoolExecutor(
				Runtime.getRuntime().availableProcessors(),
				maxPoolSize, 
				120L, 
				TimeUnit.SECONDS,
				new ArrayBlockingQueue<Runnable>(queueSize));
	}
	
	public void execute(Runnable task){
		this.executor.execute(task);
	}
}
```

ServerHandler  

```java
package bhz.bio2;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

public class ServerHandler implements Runnable {

	private Socket socket;
	public ServerHandler (Socket socket){
		this.socket = socket;
	}
	
	@Override
	public void run() {
		BufferedReader in = null;
		PrintWriter out = null;
		try {
			in = new BufferedReader(new InputStreamReader(this.socket.getInputStream()));
			out = new PrintWriter(this.socket.getOutputStream(), true);
			String body = null;
			while(true){
				body = in.readLine();
				if(body == null) break;
				System.out.println("Server:" + body);
				out.println("Server response");
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if(in != null){
				try {
					in.close();
				} catch (Exception e1) {
					e1.printStackTrace();
				}
			}
			if(out != null){
				try {
					out.close();
				} catch (Exception e2) {
					e2.printStackTrace();
				}
			}
			if(socket != null){
				try {
					socket.close();
				} catch (Exception e3) {
					e3.printStackTrace();
				}
			}
			socket = null;			
		}
	}
}
```

同步阻塞IO（BIO）  
我们熟知的Socket编程就是BIO，一个socket连接一个处理线程（这个线程负责这个Socket  
连接的一系列数据传输操作）。阻塞的原因在于：操作系统允许的线程数量是有限的，多个  
socket申请与服务端建立连接时，服务端不能提供相应数量的处理线程，没有分配到处理线  
程的连接就会阻塞等待或被拒绝。


同步非阻塞IO（NIO）  
Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，  
直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。 Java NIO的  
非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，如果  
目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取  
之前，该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，  
但不需要等待它完全写入，这个线程同时可以去做别的事情。   
线程通常将非阻塞IO的空闲时间用于在其它通道上执行IO操作，所以一个单独的线程现在可以  
管理多个输入和输出通道（channel）。

选择器（Selectors）  
Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，  
然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入   
的通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。   



异步非阻塞IO（AIO）  
NIO是同步的IO，是因为程序需要IO操作时，必须获得了IO权限后亲自进行IO操作才能进行下一  
步操作。AIO是对NIO的改进（所以AIO又叫NIO.2），它是基于Proactor模型的。每个socket连  
接在事件分离器注册 IO完成事件 和 IO完成事件处理器。程序需要进行IO时，向分离器发出IO  
请求并把所用的Buffer区域告知分离器，分离器通知操作系统进行IO操作，操作系统自己不断  
尝试获取IO权限并进行IO操作（数据保存在Buffer区），操作完成后通知分离器；分离器检测  
到 IO完成事件，则激活 IO完成事件处理器，处理器会通知程序说“IO已完成”，程序知道后就  
直接从Buffer区进行数据的读写。
也就是说：AIO是发出IO请求后，由操作系统自己去获取IO权限并进行IO操作；NIO则是发出IO  
请求后，由线程不断尝试获取IO权限，获取到后通知应用程序自己进行IO操作。  


- NIO  

nio.IntBuffer的使用  
别忘了用buf.flip()把position置为零，否则无法遍历  
flip会把position置为零，并且把limit赋值为原来的position  

```java
package bhz.nio.test;

import java.nio.IntBuffer;

public class TestBuffer {
	
	public static void main(String[] args) {
		
		// 1 基本操作
		
		//创建指定长度的缓冲区
		IntBuffer buf = IntBuffer.allocate(10);
		buf.put(13);// position位置：0 - > 1
		buf.put(21);// position位置：1 - > 2
		buf.put(35);// position位置：2 - > 3
		//把位置复位为0，也就是position位置：3 - > 0
		buf.flip();
		System.out.println("使用flip复位：" + buf);
		System.out.println("容量为: " + buf.capacity());	//容量一旦初始化后不允许改变（warp方法包裹数组除外）
		System.out.println("限制为: " + buf.limit());		//由于只装载了三个元素,所以可读取或者操作的元素为3 则limit=3
		
		
		System.out.println("获取下标为1的元素：" + buf.get(1));
		System.out.println("get(index)方法，position位置不改变：" + buf);
		buf.put(1, 4);
		System.out.println("put(index, change)方法，position位置不变：" + buf);;
		
		for (int i = 0; i < buf.limit(); i++) {
			//调用get方法会使其缓冲区位置（position）向后递增一位
			System.out.print(buf.get() + "\t");
		}
		System.out.println("buf对象遍历之后为: " + buf);
		
		
		// 2 wrap方法使用
		/*
		//  wrap方法会包裹一个数组: 一般这种用法不会先初始化缓存对象的长度，因为没有意义，最后还会被wrap所包裹的数组覆盖掉。 
		//  并且wrap方法修改缓冲区对象的时候，数组本身也会跟着发生变化。                     
		int[] arr = new int[]{1,2,5};
		IntBuffer buf1 = IntBuffer.wrap(arr);
		System.out.println(buf1);
		
		IntBuffer buf2 = IntBuffer.wrap(arr, 0 , 2);
		//这样使用表示容量为数组arr的长度，但是可操作的元素只有实际进入缓存区的元素长度
		System.out.println(buf2);
		*/
		
		
		// 3 其他方法
		/*
		IntBuffer buf1 = IntBuffer.allocate(10);
		int[] arr = new int[]{1,2,5};
		buf1.put(arr);
		System.out.println(buf1);
		//一种复制方法
		IntBuffer buf3 = buf1.duplicate();
		System.out.println(buf3);
		
		//设置buf1的位置属性
		//buf1.position(0);
		buf1.flip();
		System.out.println(buf1);
		
		System.out.println("可读数据为：" + buf1.remaining());
		
		int[] arr2 = new int[buf1.remaining()];
		//将缓冲区数据放入arr2数组中去
		buf1.get(arr2);
		for(int i : arr2){
			System.out.print(Integer.toString(i) + ",");
		}
		*/
		
	}
}

/*
使用flip复位：java.nio.HeapIntBuffer[pos=0 lim=3 cap=10]
容量为: 10
限制为: 3
获取下标为1的元素：21
get(index)方法，position位置不改变：java.nio.HeapIntBuffer[pos=0 lim=3 cap=10]
put(index, change)方法，position位置不变：java.nio.HeapIntBuffer[pos=0 lim=3 cap=10]
13	4	35	buf对象遍历之后为: java.nio.HeapIntBuffer[pos=3 lim=3 cap=10]
*/

```

Selector（选择器）的使用    
是JavaNIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好  
准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。  



代码解析:  

服务端  
通道注册到多路复用器上，并且监听阻塞事件OP_ACCEPT   
获取多路复用器已经选择的结果集  
判断状态是阻塞(accept)状态、可读状态、写数据，进行不同处理  
如果是阻塞(accept)则将客户端注册到多路复用器，并置为可读状态  
下次遍历时对缓冲区的数据进行读取  

客户端  
连接通道，创建缓冲区  
写入数据，清空缓冲区  


服务端  

```java
package bhz.nio;

public class Server implements Runnable{
	//1 多路复用器（管理所有的通道）
	private Selector seletor;
	//2 建立缓冲区
	private ByteBuffer readBuf = ByteBuffer.allocate(1024);
	//3 
	private ByteBuffer writeBuf = ByteBuffer.allocate(1024);
	public Server(int port){
		try {
			//1 打开路复用器
			this.seletor = Selector.open();
			//2 打开服务器通道
			ServerSocketChannel ssc = ServerSocketChannel.open();
			//3 设置服务器通道为非阻塞模式
			ssc.configureBlocking(false);
			//4 绑定地址
			ssc.bind(new InetSocketAddress(port));
			//5 把服务器通道注册到多路复用器上，并且监听阻塞事件
			ssc.register(this.seletor, SelectionKey.OP_ACCEPT);
			
			System.out.println("Server start, port :" + port);
			
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	@Override
	public void run() {
		while(true){
			try {
				//1 必须要让多路复用器开始监听
				this.seletor.select();
				//2 返回多路复用器已经选择的结果集
				Iterator<SelectionKey> keys = this.seletor.selectedKeys().iterator();
				//3 进行遍历
				while(keys.hasNext()){
					//4 获取一个选择的元素
					SelectionKey key = keys.next();
					//5 直接从容器中移除就可以了
					keys.remove();
					//6 如果是有效的
					if(key.isValid()){
						//7 如果为阻塞状态
						if(key.isAcceptable()){
							this.accept(key);
						}
						//8 如果为可读状态
						if(key.isReadable()){
							this.read(key);
						}
						//9 写数据
						if(key.isWritable()){
							//this.write(key); //ssc
						}
					}
					
				}
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}
	
	private void write(SelectionKey key){
		//ServerSocketChannel ssc =  (ServerSocketChannel) key.channel();
		//ssc.register(this.seletor, SelectionKey.OP_WRITE);
	}

	private void read(SelectionKey key) {
		try {
			//1 清空缓冲区旧的数据
			this.readBuf.clear();
			//2 获取之前注册的socket通道对象
			SocketChannel sc = (SocketChannel) key.channel();
			//3 读取数据
			int count = sc.read(this.readBuf);
			//4 如果没有数据
			if(count == -1){
				key.channel().close();
				key.cancel();
				return;
			}
			//5 有数据则进行读取 读取之前需要进行复位方法(把position 和limit进行复位)
			this.readBuf.flip();
			//6 根据缓冲区的数据长度创建相应大小的byte数组，接收缓冲区的数据
			byte[] bytes = new byte[this.readBuf.remaining()];
			//7 接收缓冲区数据
			this.readBuf.get(bytes);
			//8 打印结果
			String body = new String(bytes).trim();
			System.out.println("Server : " + body);
			
			// 9..可以写回给客户端数据 
			
		} catch (IOException e) {
			e.printStackTrace();
		}
		
	}

	private void accept(SelectionKey key) {
		try {
			//1 获取服务通道
			ServerSocketChannel ssc =  (ServerSocketChannel) key.channel();
			//2 执行阻塞方法
			SocketChannel sc = ssc.accept();
			//3 设置阻塞模式
			sc.configureBlocking(false);
			//4 注册到多路复用器上，并设置读取标识
			sc.register(this.seletor, SelectionKey.OP_READ);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		
		new Thread(new Server(8765)).start();;
	}
}
```

客户端代码  

```java
package bhz.nio;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.SocketChannel;

public class Client {

	//需要一个Selector 
	public static void main(String[] args) {
		
		//创建连接的地址
		InetSocketAddress address = new InetSocketAddress("127.0.0.1", 8765);
		
		//声明连接通道
		SocketChannel sc = null;
		
		//建立缓冲区
		ByteBuffer buf = ByteBuffer.allocate(1024);
		
		try {
			//打开通道
			sc = SocketChannel.open();
			//进行连接
			sc.connect(address);
			
			while(true){
				//定义一个字节数组，然后使用系统录入功能：
				byte[] bytes = new byte[1024];
				System.in.read(bytes);
				
				//把数据放到缓冲区中
				buf.put(bytes);
				//对缓冲区进行复位
				buf.flip();
				//写出数据
				sc.write(buf);
				//清空缓冲区数据
				buf.clear();
			}
		} catch (IOException e) {
			e.printStackTrace();
		} finally {
			if(sc != null){
				try {
					sc.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
		
	}
	
}

```

aio  
通道的创建需要一个线程组  
assc.accept是异步调用方法，如果没有sleep服务会停止  
accept两个参数  
第一个是参数this是server对象  
第二个参数ServerCompletionHandler具体处理读写操作  

```java
package bhz.aio;

import java.net.InetSocketAddress;
import java.nio.channels.AsynchronousChannelGroup;
import java.nio.channels.AsynchronousServerSocketChannel;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class Server {
	//线程池
	private ExecutorService executorService;
	//线程组
	private AsynchronousChannelGroup threadGroup;
	//服务器通道
	public AsynchronousServerSocketChannel assc;
	
	public Server(int port){
		try {
			//创建一个缓存池
			executorService = Executors.newCachedThreadPool();
			//创建线程组
			threadGroup = AsynchronousChannelGroup.withCachedThreadPool(executorService, 1);
			//创建服务器通道
			assc = AsynchronousServerSocketChannel.open(threadGroup);
			//进行绑定
			assc.bind(new InetSocketAddress(port));
			
			System.out.println("server start , port : " + port);
			//进行阻塞
			assc.accept(this, new ServerCompletionHandler());
			//一直阻塞 不让服务器停止
			Thread.sleep(Integer.MAX_VALUE);
			
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public static void main(String[] args) {
		Server server = new Server(8765);
	}
	
}

```

ServerCompletionHandler，实现异步处理  
要重写completed和failed  
read方法完成读数据，在read里调用write响应给客户端  
asc.read方法读成功后才会执行completed方法实现异步  
asc.write方法返回一个future对象，新开线程实现写也是异步  

```java
package bhz.aio;

import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.nio.channels.CompletionHandler;
import java.util.concurrent.ExecutionException;

public class ServerCompletionHandler implements CompletionHandler<AsynchronousSocketChannel, Server> {

	@Override
	public void completed(AsynchronousSocketChannel asc, Server attachment) {
		//当有下一个客户端接入的时候 直接调用Server的accept方法，这样反复执行下去，保证多个客户端都可以阻塞
		attachment.assc.accept(attachment, this);
		read(asc);
	}

	private void read(final AsynchronousSocketChannel asc) {
		//读取数据
		ByteBuffer buf = ByteBuffer.allocate(1024);
		asc.read(buf, buf, new CompletionHandler<Integer, ByteBuffer>() {
			@Override
			public void completed(Integer resultSize, ByteBuffer attachment) {
				//进行读取之后,重置标识位
				attachment.flip();
				//获得读取的字节数
				System.out.println("Server -> " + "收到客户端的数据长度为:" + resultSize);
				//获取读取的数据
				String resultData = new String(attachment.array()).trim();
				System.out.println("Server -> " + "收到客户端的数据信息为:" + resultData);
				String response = "服务器响应, 收到了客户端发来的数据: " + resultData;
				write(asc, response);
			}
			@Override
			public void failed(Throwable exc, ByteBuffer attachment) {
				exc.printStackTrace();
			}
		});
	}
	
	private void write(AsynchronousSocketChannel asc, String response) {
		try {
			ByteBuffer buf = ByteBuffer.allocate(1024);
			buf.put(response.getBytes());
			buf.flip();
			asc.write(buf).get();
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}
	
	@Override
	public void failed(Throwable exc, Server attachment) {
		exc.printStackTrace();
	}

}

```

aio的client  

```java
package bhz.aio;

import java.io.UnsupportedEncodingException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.AsynchronousSocketChannel;
import java.util.concurrent.ExecutionException;

public class Client implements Runnable{

	private AsynchronousSocketChannel asc ;
	
	public Client() throws Exception {
		asc = AsynchronousSocketChannel.open();
	}
	
	public void connect(){
		asc.connect(new InetSocketAddress("127.0.0.1", 8765));
	}
	
	public void write(String request){
		try {
			asc.write(ByteBuffer.wrap(request.getBytes())).get();
			read();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	private void read() {
		ByteBuffer buf = ByteBuffer.allocate(1024);
		try {
			asc.read(buf).get();
			buf.flip();
			byte[] respByte = new byte[buf.remaining()];
			buf.get(respByte);
			System.out.println(new String(respByte,"utf-8").trim());
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}
	}
	
	@Override
	public void run() {
		while(true){
			
		}
	}
	
	public static void main(String[] args) throws Exception {
		Client c1 = new Client();
		c1.connect();
		
		Client c2 = new Client();
		c2.connect();
		
		Client c3 = new Client();
		c3.connect();
		
		new Thread(c1, "c1").start();
		new Thread(c2, "c2").start();
		new Thread(c3, "c3").start();
		
		Thread.sleep(1000);
		
		c1.write("c1 aaa");
		c2.write("c2 bbbb");
		c3.write("c3 ccccc");
	}
	
}
```

- NIO和IO的主要区别  
IO            NIO
面向流        面向缓冲
阻塞IO        非阻塞IO
无            选择器

nio的优点，减少socket连接时线程数量  

阻塞与非阻塞IO  
Java IO的各种流是阻塞的。这意味着，当一个线程调用read() 或 write()时，该线程被阻塞，直到有一些数据被读取，或数据完全写入。该线程在此期间不能再干任何事情了。  
Java NIO的非阻塞模式，使一个线程从某通道发送请求读取数据，但是它仅能得到目前可用的数据，  
如果目前没有数据可用时，就什么都不会获取。而不是保持线程阻塞，所以直至数据变的可以读取之前，  
该线程可以继续做其他的事情。 非阻塞写也是如此。一个线程请求写入一些数据到某通道，但不需要等待  
它完全写入，这个线程同时可以去做别的事情。 线程通常将非阻塞IO的空闲时间用于在其它通道上执行  
IO操作，所以一个单独的线程现在可以管理多个输入和输出通道（channel）。  

选择器（Selectors）  
Java NIO的选择器允许一个单独的线程来监视多个输入通道，你可以注册多个通道使用一个选择器，  
然后使用一个单独的线程来“选择”通道：这些通道里已经有可以处理的输入，或者选择已准备写入的  
通道。这种选择机制，使得一个单独的线程很容易来管理多个通道。  



- netty入门  


DISCARD服务(丢弃服务，指的是会忽略所有接收的数据的一种协议)  
世界上最简单的协议不是“Hello,World!”，是DISCARD，他是一种丢弃了所有接  
受到的数据，并不做有任何的响应的协议。为了实现DISCARD协议，你唯一需要做  
的就是忽略所有收到的数据。让我们从处理器的实现开始，处理器是由Netty生成  
用来处理I/O事件的  

1、DisCardServerHandler 继承自 ChannelHandlerAdapter，这个类实现了  
ChannelHandler接口，ChannelHandler提供了许多事件处理的接口方法，然后  
你可以覆盖这些方法。现在仅仅只需要继承ChannelHandlerAdapter类而不是你  
自己去实现接口方法。  

2、这里我们覆盖了chanelRead()事件处理方法。每当从客户端收到新的数据时，  
这个方法会在收到消息时被调用，这个例子中，收到的消息的类型是ByteBuf  

3、为了实现DISCARD协议，处理器不得不忽略所有接受到的消息。ByteBuf是一个引  
用计数对象，这个对象必须显示地调用release()方法来释放。请记住处理器的  
职责是释放所有传递到处理器的引用计数对象。  

4、exceptionCaught()事件处理方法是当出现Throwable对象才会被调用，即当  
Netty由于IO错误或者处理器在处理事件时抛出的异常时。在大部分情况下，捕获  
的异常应该被记录下来并且把关联的channel给关闭掉。然而这个方法的处理方式  
会在遇到不同异常的情况下有不同的实现，比如你可能想在关闭连接之前发送一个  
错误码的响应消息。  

DisCardServerHandler  

```java
package io.netty.example.discard;

import io.netty.buffer.ByteBuf;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelHandlerAdapter;

/**
 * Handles a server-side channel.
 */
public class DiscardServerHandler extends ChannelHandlerAdapter { // (1)

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { // (2)
        // Discard the received data silently.
        ((ByteBuf) msg).release(); // (3)
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) { // (4)
        // Close the connection when an exception is raised.
        cause.printStackTrace();
        ctx.close();
    }
}
```

我们已经实现了DISCARD服务的一半功能，剩下的需要编写一个main()方法来启动  
服务端的DiscardServerHandler  

NioEventLoopGroup 是用来处理I/O操作的多线程事件循环器，第一个被叫做‘boss’，  
用来接收进来的连接。第二个被叫做‘worker’，用来处理已经被接收的连接，一旦‘boss’    
射到已经创建的Channels上都需要依赖于EventLoopGroup的实现，并且可以通过构造  
函数来配置他们的关系。  




三次握手的总结  
服务器端TCP内核模块有两个队列  
客户端进行连接  
服务端进行确认  
将客户端连接加入A队列  
然后服务器收到客户端发来的ACK时  
将客户端从A队列移动到B队列，连接完成，应用程序的accept会返回  
accept从B队列中取出完成三次握手的连接  
当A、B队列的长度之和大于backlog时，新连接将会被TCP内核拒绝  

Netty 实现通信的步骤：  
1.创建两个NIO线程组，一个专门用于网络事件处理（接受客户端的连接），另一个则进行网络通信读写。  
2.创建一个ServerBootstrap对象，配置Netty的一系列参数，例如接受传出数据的缓存大小等。  
3.创建一个实际处理数据的类ChannelInitializer，进行初始化的准备工作，比如设置接受传出数据的字符、格式以及实际处理数据的接口  
4.绑定接口，执行同步阻塞方法等待服务器启动即可。  
如此简单的四个步骤，我们的服务器端就编写完成了，几十行代码 就可以把之前NIO的代码代替。  

Server  

```java

public class Server {

	public static void main(String[] args) throws Exception {
		//1 创建线两个程组 
		//一个是用于处理服务器端接收客户端连接的
		//一个是进行网络通信的（网络读写的）
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		//2 创建辅助工具类，用于服务器通道的一系列配置
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)		//绑定俩个线程组
		.channel(NioServerSocketChannel.class)		//指定NIO的模式

		 /**
         * 服务器端TCP内核模块维护两个队列，我们称之为A和B
         * 客户端向服务器端connect的时候 会发送带有SYN标志的包(第一次握手)
         * 服务器收到客户端发来的SYN时，向客户端发送SYN ACK确认(第二次握手)
         * 此时TCP内核模块将客户端连接加入A队列中，然后服务器收到客户端发来的ACK时(第三次握手)
         * TCP内核模块将客户端从A队列移动到B队列，连接完成，应用程序的accept会返回。
         * 也就是说accept从B队列中取出完成三次握手的连接
         * A队列和B队列的长度之和是backlog。当A、B队列的长度之和大于backlog时，新连接将会被TCP内核拒绝
         * 所以，如果backlog过小，可能会出现accept速度跟不上，A、B队列满了，导致新的客户无法连接。
         * 要注意的是：backlog对程序的连接数并无影响，backlog影响的知识还没有被accept取出的连接。
         */
		.option(ChannelOption.SO_BACKLOG, 1024)		//设置tcp缓冲区
		.option(ChannelOption.SO_SNDBUF, 32*1024)	//设置发送缓冲大小
		.option(ChannelOption.SO_RCVBUF, 32*1024)	//这是接收缓冲大小
		.option(ChannelOption.SO_KEEPALIVE, true)	//保持连接
		.childHandler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				//3 在这里配置具体数据接收方法的处理
				sc.pipeline().addLast(new ServerHandler());
			}
		});
		
		//4 进行绑定 
		ChannelFuture cf1 = b.bind(8765).sync();
		//ChannelFuture cf2 = b.bind(8764).sync();
		//5 等待关闭
		cf1.channel().closeFuture().sync();
		//cf2.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
	}
}


}
```

DiscardServerHandler改造成ServerHandler  
我们使用NIO发送消息时不是调用java.nio.ByteBuffer.flip()吗？BuyteBf之所以没有这个方法  
因为有两个指针，一个对应读操作一个对应写操作。当你向ByteBuf里写入数据的时候写指针的索引  
就会增加，同时读指针的索引没有变化。读指针索引和写指针索引分别代表了消息的开始和结束。  

ServerHandler可以读或写数据，写出推荐使用writeAndFlush  

细节问题：由于服务端使用了写操作，可以省略release(msg)释放操作  

如果使用了addListener(ChannelFutureListener.CLOSE);客户端接收到响应数据后会断开与服务器的连接  

```java
public class ServerHandler extends ChannelHandlerAdapter {

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println("server channel active... ");
	}


	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg)
			throws Exception {
			ByteBuf buf = (ByteBuf) msg;
			byte[] req = new byte[buf.readableBytes()];
			buf.readBytes(req);
			String body = new String(req, "utf-8");
			System.out.println("Server :" + body );
			String response = "进行返回给客户端的响应：" + body ;
			ctx.writeAndFlush(Unpooled.copiedBuffer(response.getBytes()));
			//.addListener(ChannelFutureListener.CLOSE);
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx)
			throws Exception {
		System.out.println("读完了");
		ctx.flush();
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable t)
			throws Exception {
		ctx.close();
	}

}

```

客户端  
只有workgroup一个线程组  
使用的channel是NioSocketChannel  
处理方法还是调handler  
通道写出777  

```java
public class Client {

	public static void main(String[] args) throws Exception{
		
		EventLoopGroup workgroup = new NioEventLoopGroup();
		Bootstrap b = new Bootstrap();
		b.group(workgroup)
		.channel(NioSocketChannel.class)
		.handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(new ClientHandler());
			}
		});
		
		ChannelFuture cf1 = b.connect("127.0.0.1", 8765).sync();

		cf1.channel().write(Unpooled.copiedBuffer("777".getBytes()));
		cf1.channel().flush();

		cf1.channel().closeFuture().sync();
		group.shutdownGracefully();
		
		
		
	}
}

```

ClientHandler  
读取服务端相应的数据  

```java
public class ClientHandler extends ChannelHandlerAdapter { 

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) { 
    		try {
			ByteBuf buf = (ByteBuf) msg;
			
			byte[] req = new byte[buf.readableBytes()];
			buf.readBytes(req);
			
			String body = new String(req, "utf-8");
			System.out.println("Client :" + body );
			String response = "收到服务器端的返回信息：" + body;
		} finally {
			ReferenceCountUtil.release(msg);
		}
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
        cause.printStackTrace();
        ctx.close();
    }
}
```

- 粘包问题的解决策略(v0106003)  
由于底层的TCP无法理解上层的业务逻辑，所以在底层是无法确保数据包不被拆分和重组的，  
这个问题只能通过上层的应用协议栈设计来解决，根据业界的主流协议的解决方案，归纳如下：  

1.消息定长，例如每个报文的大小为固定长度200字节,如果不够，空位补空格；  
2.在包尾增加回车换行符进行分割，例如FTP协议；  
3.将消息分为消息头和消息体，消息头中包含表示消息总长度（或者消息体长度）的字段，  
通常设计思路是消息头的第一个字段用int来表示消息的总长度；  


在末尾增加标识位  
Server  

```java
package bhz.netty.ende1;
public class Server {

	public static void main(String[] args) throws Exception{
		//1 创建2个线程，一个是负责接收客户端的连接。一个是负责进行数据传输的
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		//2 创建服务器辅助类
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)
		 .channel(NioServerSocketChannel.class)
		 .option(ChannelOption.SO_BACKLOG, 1024)
		 .option(ChannelOption.SO_SNDBUF, 32*1024)
		 .option(ChannelOption.SO_RCVBUF, 32*1024)
		 .childHandler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				//设置特殊分隔符
				ByteBuf buf = Unpooled.copiedBuffer("$_".getBytes());
				sc.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, buf));
				//设置字符串形式的解码
				sc.pipeline().addLast(new StringDecoder());
				sc.pipeline().addLast(new ServerHandler());
			}
		});
		
		//4 绑定连接
		ChannelFuture cf = b.bind(8765).sync();
		
		//等待服务器监听端口关闭
		cf.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
		
	}
	
}

```

ServerHandler  

```java
package bhz.netty.ende1;

public class ServerHandler extends ChannelHandlerAdapter {


	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		System.out.println(" server channel active... ");
	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		String request = (String)msg;
		System.out.println("Server :" + msg);
		String response = "服务器响应：" + msg + "$_";
		ctx.writeAndFlush(Unpooled.copiedBuffer(response.getBytes()));
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable t) throws Exception {
		ctx.close();
	}

}
```

Client  

```java
package bhz.netty.ende1;
public class Client {

	public static void main(String[] args) throws Exception {
		
		EventLoopGroup group = new NioEventLoopGroup();
		
		Bootstrap b = new Bootstrap();
		b.group(group)
		 .channel(NioSocketChannel.class)
		 .handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				//
				ByteBuf buf = Unpooled.copiedBuffer("$_".getBytes());
				sc.pipeline().addLast(new DelimiterBasedFrameDecoder(1024, buf));
				sc.pipeline().addLast(new StringDecoder());
				sc.pipeline().addLast(new ClientHandler());
			}
		});
		
		ChannelFuture cf = b.connect("127.0.0.1", 8765).sync();
		
		cf.channel().writeAndFlush(Unpooled.wrappedBuffer("bbbb$_".getBytes()));
		cf.channel().writeAndFlush(Unpooled.wrappedBuffer("cccc$_".getBytes()));
		
		
		//等待客户端端口关闭
		cf.channel().closeFuture().sync();
		group.shutdownGracefully();
		
	}
}

```

- 报文固定长度  
Server  
FixedLengthFrameDecoder使用固定长度  

```java
public class Server {

	public static void main(String[] args) throws Exception{
		//1 创建2个线程，一个是负责接收客户端的连接。一个是负责进行数据传输的
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		//2 创建服务器辅助类
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)
		 .channel(NioServerSocketChannel.class)
		 .option(ChannelOption.SO_BACKLOG, 1024)
		 .option(ChannelOption.SO_SNDBUF, 32*1024)
		 .option(ChannelOption.SO_RCVBUF, 32*1024)
		 .childHandler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				//设置定长字符串接收
				sc.pipeline().addLast(new FixedLengthFrameDecoder(5));
				//设置字符串形式的解码
				sc.pipeline().addLast(new StringDecoder());
				sc.pipeline().addLast(new ServerHandler());
			}
		});
		
		//4 绑定连接
		ChannelFuture cf = b.bind(8765).sync();
		
		//等待服务器监听端口关闭
		cf.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
		
	}
	
}

```

Client  
FixedLengthFrameDecoder使用固定长度  
发了七个c会丢失两个  

```java
public class Client {

	public static void main(String[] args) throws Exception {
		
		EventLoopGroup group = new NioEventLoopGroup();
		
		Bootstrap b = new Bootstrap();
		b.group(group)
		 .channel(NioSocketChannel.class)
		 .handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(new FixedLengthFrameDecoder(5));
				sc.pipeline().addLast(new StringDecoder());
				sc.pipeline().addLast(new ClientHandler());
			}
		});
		
		ChannelFuture cf = b.connect("127.0.0.1", 8765).sync();
		
		cf.channel().writeAndFlush(Unpooled.wrappedBuffer("aaaaabbbbb".getBytes()));
		cf.channel().writeAndFlush(Unpooled.copiedBuffer("ccccccc".getBytes()));
		
		//等待客户端端口关闭
		cf.channel().closeFuture().sync();
		group.shutdownGracefully();
		
	}
}

```

- netty编解码技术(v0107002)  

server  
LoggingHandler，是netty用来打印日志  
MarshallingCodeCFactory指定Decoder和Encoder  

```java
package bhz.netty.serial;
public class Server {

	public static void main(String[] args) throws Exception{
		
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)
		 .channel(NioServerSocketChannel.class)
		 .option(ChannelOption.SO_BACKLOG, 1024)
		 //设置日志
		 .handler(new LoggingHandler(LogLevel.INFO))
		 .childHandler(new ChannelInitializer<SocketChannel>() {
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
				sc.pipeline().addLast(new ServerHandler());
			}
		});
		
		ChannelFuture cf = b.bind(8765).sync();
		
		cf.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
		
	}
}

```

client  
MarshallingCodeCFactory指定Decoder和Encoder  

```java
package bhz.netty.serial;
public class Client {

	
	public static void main(String[] args) throws Exception{
		
		EventLoopGroup group = new NioEventLoopGroup();
		Bootstrap b = new Bootstrap();
		b.group(group)
		 .channel(NioSocketChannel.class)
		 .handler(new ChannelInitializer<SocketChannel>() {
			@Override
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
				sc.pipeline().addLast(new ClientHandler());
			}
		});
		
		ChannelFuture cf = b.connect("127.0.0.1", 8765).sync();
		
		for(int i = 0; i < 5; i++ ){
			Req req = new Req();
			req.setId("" + i);
			req.setName("pro" + i);
			req.setRequestMessage("数据信息" + i);	
			String path = System.getProperty("user.dir") + File.separatorChar + "sources" +  File.separatorChar + "001.jpg";
			File file = new File(path);
	        FileInputStream in = new FileInputStream(file);  
	        byte[] data = new byte[in.available()];  
	        in.read(data);  
	        in.close(); 
			req.setAttachment(GzipUtils.gzip(data));
			cf.channel().writeAndFlush(req);
		}

		cf.channel().closeFuture().sync();
		group.shutdownGracefully();
	}
}

```

ServerHandler  

```java
package bhz.netty.serial;

public class ServerHandler extends ChannelHandlerAdapter{

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {

	}

	@Override
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		Req req = (Req)msg;
		System.out.println("Server : " + req.getId() + ", " + req.getName() + ", " + req.getRequestMessage());
		byte[] attachment = GzipUtils.ungzip(req.getAttachment());
		
		String path = System.getProperty("user.dir") + File.separatorChar + "receive" +  File.separatorChar + "001.jpg";
        FileOutputStream fos = new FileOutputStream(path);
        fos.write(attachment);
        fos.close();
		
		Resp resp = new Resp();
		resp.setId(req.getId());
		resp.setName("resp" + req.getId());
		resp.setResponseMessage("响应内容" + req.getId());
		ctx.writeAndFlush(resp);//.addListener(ChannelFutureListener.CLOSE);
	}

	@Override
	public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
		
	}

	@Override
	public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
		ctx.close();
	}	
}
```

MarshallingCodeCFactory工具类，直接用就行  

```java
package bhz.netty.serial;
public final class MarshallingCodeCFactory {

    /**
     * 创建Jboss Marshalling解码器MarshallingDecoder
     * @return MarshallingDecoder
     */
    public static MarshallingDecoder buildMarshallingDecoder() {
    	//首先通过Marshalling工具类的精通方法获取Marshalling实例对象 参数serial标识创建的是java序列化工厂对象。
		final MarshallerFactory marshallerFactory = Marshalling.getProvidedMarshallerFactory("serial");
		//创建了MarshallingConfiguration对象，配置了版本号为5 
		final MarshallingConfiguration configuration = new MarshallingConfiguration();
		configuration.setVersion(5);
		//根据marshallerFactory和configuration创建provider
		UnmarshallerProvider provider = new DefaultUnmarshallerProvider(marshallerFactory, configuration);
		//构建Netty的MarshallingDecoder对象，俩个参数分别为provider和单个消息序列化后的最大长度
		MarshallingDecoder decoder = new MarshallingDecoder(provider, 1024 * 1024 * 1);
		return decoder;
    }

    /**
     * 创建Jboss Marshalling编码器MarshallingEncoder
     * @return MarshallingEncoder
     */
    public static MarshallingEncoder buildMarshallingEncoder() {
		final MarshallerFactory marshallerFactory = Marshalling.getProvidedMarshallerFactory("serial");
		final MarshallingConfiguration configuration = new MarshallingConfiguration();
		configuration.setVersion(5);
		MarshallerProvider provider = new DefaultMarshallerProvider(marshallerFactory, configuration);
		//构建Netty的MarshallingEncoder对象，MarshallingEncoder用于实现序列化接口的POJO对象序列化为二进制数组
		MarshallingEncoder encoder = new MarshallingEncoder(provider);
		return encoder;
    }
}

```

- 压缩和解压(v0107002)  
上面的server和client实现了文件压缩传输  
gzip和ungzip用于压缩和解压  

```java
package bhz.utils;
public class GzipUtils {

    public static byte[] gzip(byte[] data) throws Exception{
    	ByteArrayOutputStream bos = new ByteArrayOutputStream();
    	GZIPOutputStream gzip = new GZIPOutputStream(bos);
    	gzip.write(data);
    	gzip.finish();
    	gzip.close();
    	byte[] ret = bos.toByteArray();
    	bos.close();
    	return ret;
    }
    
    public static byte[] ungzip(byte[] data) throws Exception{
    	ByteArrayInputStream bis = new ByteArrayInputStream(data);
    	GZIPInputStream gzip = new GZIPInputStream(bis);
    	byte[] buf = new byte[1024];
    	int num = -1;
    	ByteArrayOutputStream bos = new ByteArrayOutputStream();
    	while((num = gzip.read(buf, 0 , buf.length)) != -1 ){
    		bos.write(buf, 0, num);
    	}
    	gzip.close();
    	bis.close();
    	byte[] ret = bos.toByteArray();
    	bos.flush();
    	bos.close();
    	return ret;
    }
    
    public static void main(String[] args) throws Exception{
		
    	//读取文件
    	String readPath = System.getProperty("user.dir") + File.separatorChar + "sources" +  File.separatorChar + "006.jpg";
        File file = new File(readPath);  
        FileInputStream in = new FileInputStream(file);  
        byte[] data = new byte[in.available()];  
        in.read(data);  
        in.close();  
        
        System.out.println("文件原始大小:" + data.length);
        //测试压缩
        
        byte[] ret1 = GzipUtils.gzip(data);
        System.out.println("压缩之后大小:" + ret1.length);
        
        byte[] ret2 = GzipUtils.ungzip(ret1);
        System.out.println("还原之后大小:" + ret2.length);
        
        //写出文件
        String writePath = System.getProperty("user.dir") + File.separatorChar + "receive" +  File.separatorChar + "006.jpg";
        FileOutputStream fos = new FileOutputStream(writePath);
        fos.write(ret2);
        fos.close();    		
	}
}
```

复习SystemProperty  

```java
package bhz.utils;

public class TestSystemProperty {
	
	public static void main(String[] args) {
	
		System.out.println("java版本号：" + System.getProperty("java.version")); // java版本号
		System.out.println("Java提供商名称：" + System.getProperty("java.vendor")); // Java提供商名称
		System.out.println("Java提供商网站：" + System.getProperty("java.vendor.url")); // Java提供商网站
		System.out.println("jre目录：" + System.getProperty("java.home")); // Java，哦，应该是jre目录
		System.out.println("Java虚拟机规范版本号：" + System.getProperty("java.vm.specification.version")); // Java虚拟机规范版本号
		System.out.println("Java虚拟机规范提供商：" + System.getProperty("java.vm.specification.vendor")); // Java虚拟机规范提供商
		System.out.println("Java虚拟机规范名称：" + System.getProperty("java.vm.specification.name")); // Java虚拟机规范名称
		System.out.println("Java虚拟机版本号：" + System.getProperty("java.vm.version")); // Java虚拟机版本号
		System.out.println("Java虚拟机提供商：" + System.getProperty("java.vm.vendor")); // Java虚拟机提供商
		System.out.println("Java虚拟机名称：" + System.getProperty("java.vm.name")); // Java虚拟机名称
		System.out.println("Java规范版本号：" + System.getProperty("java.specification.version")); // Java规范版本号
		System.out.println("Java规范提供商：" + System.getProperty("java.specification.vendor")); // Java规范提供商
		System.out.println("Java规范名称：" + System.getProperty("java.specification.name")); // Java规范名称
		System.out.println("Java类版本号：" + System.getProperty("java.class.version")); // Java类版本号
		System.out.println("Java类路径：" + System.getProperty("java.class.path")); // Java类路径
		System.out.println("Java lib路径：" + System.getProperty("java.library.path")); // Java lib路径
		System.out.println("Java输入输出临时路径：" + System.getProperty("java.io.tmpdir")); // Java输入输出临时路径
		System.out.println("Java编译器：" + System.getProperty("java.compiler")); // Java编译器
		System.out.println("Java执行路径：" + System.getProperty("java.ext.dirs")); // Java执行路径
		System.out.println("操作系统名称：" + System.getProperty("os.name")); // 操作系统名称
		System.out.println("操作系统的架构：" + System.getProperty("os.arch")); // 操作系统的架构
		System.out.println("操作系统版本号：" + System.getProperty("os.version")); // 操作系统版本号
		System.out.println("文件分隔符：" + System.getProperty("file.separator")); // 文件分隔符
		System.out.println("路径分隔符：" + System.getProperty("path.separator")); // 路径分隔符
		System.out.println("直线分隔符：" + System.getProperty("line.separator")); // 直线分隔符
		System.out.println("操作系统用户名：" + System.getProperty("user.name")); // 用户名
		System.out.println("操作系统用户的主目录：" + System.getProperty("user.home")); // 用户的主目录
		System.out.println("当前程序所在目录：" + System.getProperty("user.dir")); // 当前程序所在目录
	}
}
```


- Netty的使用(v0107003)  


我们需要考虑的问题是两台机器(甚至多台)使用Netty的怎样进行通信，大体上分为三种：  

第一种，使用长连接通道不断开的形式进行通信，也就是服务器和客户端的通道一直处于开  
启状态，如果服务器性能足够好，并且我们的客户端数量也比较少的情况下，我还是推荐这  
种方式的。
第二种，一次性批量提交数据，采用短连接的方式，也就是我们会把数据保存在本地临时缓  
冲区或者临时表里，当达到临界值时进行一次性批量提交，又或者根据定时任务轮询提交，  
这种情况弊端是做不到实时性传输，在对实时性不高的应用程序中可以推荐使用。  
第三种，我们可以使用一种特殊的长连接，在指定某一时间之内，服务器与某台客户端没有  
任何通信，则断开连接，下次连接则是客户端想服务器发送请求的时候，再次建立连接，但  
是这种模式需要考虑2个因素：
(1)如何在超时(即服务器和客户端每天任何通信)后关闭通道？关闭通道后，又如何再次建  
立连接？  
(2)客户端宕机时，我们无需考虑，下次客户端重启之后我们就可以与服务器简历连接，但  
是服务器宕机时，客户端如何与服务器进行连接呢？  


第三种模式的示例  
Server  
服务器客户端都通过超时处理器关闭通道  
addLast(new ReadTimeoutHandler(5));  

```java
package bhz.netty.runtime;
public class Server {

	public static void main(String[] args) throws Exception{
		
		EventLoopGroup pGroup = new NioEventLoopGroup();
		EventLoopGroup cGroup = new NioEventLoopGroup();
		
		ServerBootstrap b = new ServerBootstrap();
		b.group(pGroup, cGroup)
		 .channel(NioServerSocketChannel.class)
		 .option(ChannelOption.SO_BACKLOG, 1024)
		 //设置日志
		 .handler(new LoggingHandler(LogLevel.INFO))
		 .childHandler(new ChannelInitializer<SocketChannel>() {
			protected void initChannel(SocketChannel sc) throws Exception {
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
				sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
				sc.pipeline().addLast(new ReadTimeoutHandler(5)); 
				sc.pipeline().addLast(new ServerHandler());
			}
		});
		
		ChannelFuture cf = b.bind(8765).sync();
		
		cf.channel().closeFuture().sync();
		pGroup.shutdownGracefully();
		cGroup.shutdownGracefully();
		
	}
}
```

Client  
getChannelFuture方法判断cf是否为null或者isActive  
如果不是则重新连接  

```java
package bhz.netty.runtime;
public class Client {
	
	private static class SingletonHolder {
		static final Client instance = new Client();
	}
	
	public static Client getInstance(){
		return SingletonHolder.instance;
	}
	
	private EventLoopGroup group;
	private Bootstrap b;
	private ChannelFuture cf ;
	
	private Client(){
			group = new NioEventLoopGroup();
			b = new Bootstrap();
			b.group(group)
			 .channel(NioSocketChannel.class)
			 .handler(new LoggingHandler(LogLevel.INFO))
			 .handler(new ChannelInitializer<SocketChannel>() {
					@Override
					protected void initChannel(SocketChannel sc) throws Exception {
						sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingDecoder());
						sc.pipeline().addLast(MarshallingCodeCFactory.buildMarshallingEncoder());
						//超时handler（当服务器端与客户端在指定时间以上没有任何进行通信，则会关闭响应的通道，主要为减小服务端资源占用）
						sc.pipeline().addLast(new ReadTimeoutHandler(5)); 
						sc.pipeline().addLast(new ClientHandler());
					}
		    });
	}
	
	public void connect(){
		try {
			this.cf = b.connect("127.0.0.1", 8765).sync();
			System.out.println("远程服务器已经连接, 可以进行数据交换..");				
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	public ChannelFuture getChannelFuture(){
		
		if(this.cf == null){
			this.connect();
		}
		if(!this.cf.channel().isActive()){
			this.connect();
		}
		
		return this.cf;
	}
	
	public static void main(String[] args) throws Exception{
		final Client c = Client.getInstance();
		//c.connect();
		
		ChannelFuture cf = c.getChannelFuture();
		for(int i = 1; i <= 3; i++ ){
			Request request = new Request();
			request.setId("" + i);
			request.setName("pro" + i);
			request.setRequestMessage("数据信息" + i);
			cf.channel().writeAndFlush(request);
			TimeUnit.SECONDS.sleep(4);
		}

		cf.channel().closeFuture().sync();
		
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				try {
					System.out.println("进入子线程...");
					ChannelFuture cf = c.getChannelFuture();
					System.out.println(cf.channel().isActive());
					System.out.println(cf.channel().isOpen());
					
					//再次发送数据
					Request request = new Request();
					request.setId("" + 4);
					request.setName("pro" + 4);
					request.setRequestMessage("数据信息" + 4);
					cf.channel().writeAndFlush(request);					
					cf.channel().closeFuture().sync();
					System.out.println("子线程结束.");
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}
		}).start();
		
		System.out.println("断开连接,主线程结束..");
	}
}
```

- netty的应用    
netty实现文件上传下载见视频  

netty实现http协议  

```java
package bhz.netty.http;
public final class HttpHelloWorldServer {
  
      static final boolean SSL = System.getProperty("ssl") != null;
      static final int PORT = Integer.parseInt(System.getProperty("port", SSL? "8443" : "8080"));
  
      public static void main(String[] args) throws Exception {
          // Configure SSL.
          final SslContext sslCtx;
          if (SSL) {
              SelfSignedCertificate ssc = new SelfSignedCertificate();
              sslCtx = SslContext.newServerContext(ssc.certificate(), ssc.privateKey());
          } else {
              sslCtx = null;
          }
  
          // Configure the server.
          EventLoopGroup bossGroup = new NioEventLoopGroup(1);
          EventLoopGroup workerGroup = new NioEventLoopGroup();
          try {
              ServerBootstrap b = new ServerBootstrap();
              b.option(ChannelOption.SO_BACKLOG, 1024);
              b.group(bossGroup, workerGroup)
               .channel(NioServerSocketChannel.class)
               .handler(new LoggingHandler(LogLevel.INFO))
               .childHandler(new HttpHelloWorldServerInitializer(sslCtx));
  
              Channel ch = b.bind(PORT).sync().channel();
  
              System.err.println("Open your web browser and navigate to " +
                      (SSL? "https" : "http") + "://127.0.0.1:" + PORT + '/');
  
              ch.closeFuture().sync();
          } finally {
              bossGroup.shutdownGracefully();
              workerGroup.shutdownGracefully();
          }
      }     
}
```

HttpServerCodec和HttpHelloWorldServerHandler是netty封装的实现http协议的对象  

```java
  package bhz.netty.http;
  public class HttpHelloWorldServerInitializer extends ChannelInitializer<SocketChannel> {
      private final SslContext sslCtx;
  
      public HttpHelloWorldServerInitializer(SslContext sslCtx) {
         this.sslCtx = sslCtx;
      }
  
      @Override
      public void initChannel(SocketChannel ch) {
          ChannelPipeline p = ch.pipeline();
          if (sslCtx != null) {
              p.addLast(sslCtx.newHandler(ch.alloc()));
          }
          p.addLast(new HttpServerCodec());
          p.addLast(new HttpHelloWorldServerHandler());
      }
  }
```

向浏览器响应Hello World  

```java
package bhz.netty.http;
public class HttpHelloWorldServerHandler extends ChannelHandlerAdapter {
      private static final byte[] CONTENT = { 'H', 'e', 'l', 'l', 'o', ' ', 'W', 'o', 'r', 'l', 'd' };
  
      @Override
      public void channelReadComplete(ChannelHandlerContext ctx) {
          ctx.flush();
      }
      @Override
      public void channelRead(ChannelHandlerContext ctx, Object msg) {
          if (msg instanceof HttpRequest) {
              HttpRequest req = (HttpRequest) msg;
  
              if (HttpHeaderUtil.is100ContinueExpected(req)) {
                  ctx.write(new DefaultFullHttpResponse(HTTP_1_1, CONTINUE));
              }
              boolean keepAlive = HttpHeaderUtil.isKeepAlive(req);
              FullHttpResponse response = new DefaultFullHttpResponse(HTTP_1_1, OK, Unpooled.wrappedBuffer(CONTENT));
              response.headers().set(CONTENT_TYPE, "text/plain");
              response.headers().setInt(CONTENT_LENGTH, response.content().readableBytes());
  
              if (!keepAlive) {
                  ctx.write(response).addListener(ChannelFutureListener.CLOSE);
              } else {
                  response.headers().set(CONNECTION, HttpHeaderValues.KEEP_ALIVE);
                  ctx.write(response);
              }
          }
      }
  
      @Override
      public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) {
          cause.printStackTrace();
          ctx.close();
      }
  }
```

- sigar的使用  
sigar-amd64-winnt.dll要放到jdk中  

```java
package bhz.utils;

import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.Map;
import java.util.Properties;

import org.hyperic.sigar.Cpu;
import org.hyperic.sigar.CpuInfo;
import org.hyperic.sigar.CpuPerc;
import org.hyperic.sigar.FileSystem;
import org.hyperic.sigar.FileSystemUsage;
import org.hyperic.sigar.Mem;
import org.hyperic.sigar.NetFlags;
import org.hyperic.sigar.NetInterfaceConfig;
import org.hyperic.sigar.NetInterfaceStat;
import org.hyperic.sigar.OperatingSystem;
import org.hyperic.sigar.Sigar;
import org.hyperic.sigar.SigarException;
import org.hyperic.sigar.Swap;
import org.hyperic.sigar.Who;

public class TestSigar {
    public static void main(String[] args) {
        try {
            // System信息，从jvm获取
            property();
            System.out.println("----------------------------------");
            // cpu信息
            cpu();
            System.out.println("----------------------------------");
            // 内存信息
            memory();
            System.out.println("----------------------------------");
            // 操作系统信息
            os();
            System.out.println("----------------------------------");
            // 用户信息
            who();
            System.out.println("----------------------------------");
            // 文件系统信息
            file();
            System.out.println("----------------------------------");
            // 网络信息
            net();
            System.out.println("----------------------------------");
            // 以太网信息
            ethernet();
            System.out.println("----------------------------------");
        } catch (Exception e1) {
            e1.printStackTrace();
        }
    }

    private static void property() throws UnknownHostException {
        Runtime r = Runtime.getRuntime();
        Properties props = System.getProperties();
        InetAddress addr;
        addr = InetAddress.getLocalHost();
        String ip = addr.getHostAddress();
        Map<String, String> map = System.getenv();
        String userName = map.get("USERNAME");// 获取用户名
        String computerName = map.get("COMPUTERNAME");// 获取计算机名
        String userDomain = map.get("USERDOMAIN");// 获取计算机域名
        System.out.println("用户名:    " + userName);
        System.out.println("计算机名:    " + computerName);
        System.out.println("计算机域名:    " + userDomain);
        System.out.println("本地ip地址:    " + ip);
        System.out.println("本地主机名:    " + addr.getHostName());
        System.out.println("JVM可以使用的总内存:    " + r.totalMemory());
        System.out.println("JVM可以使用的剩余内存:    " + r.freeMemory());
        System.out.println("JVM可以使用的处理器个数:    " + r.availableProcessors());
        System.out.println("Java的运行环境版本：    " + props.getProperty("java.version"));
        System.out.println("Java的运行环境供应商：    " + props.getProperty("java.vendor"));
        System.out.println("Java供应商的URL：    " + props.getProperty("java.vendor.url"));
        System.out.println("Java的安装路径：    " + props.getProperty("java.home"));
        System.out.println("Java的虚拟机规范版本：    " + props.getProperty("java.vm.specification.version"));
        System.out.println("Java的虚拟机规范供应商：    " + props.getProperty("java.vm.specification.vendor"));
        System.out.println("Java的虚拟机规范名称：    " + props.getProperty("java.vm.specification.name"));
        System.out.println("Java的虚拟机实现版本：    " + props.getProperty("java.vm.version"));
        System.out.println("Java的虚拟机实现供应商：    " + props.getProperty("java.vm.vendor"));
        System.out.println("Java的虚拟机实现名称：    " + props.getProperty("java.vm.name"));
        System.out.println("Java运行时环境规范版本：    " + props.getProperty("java.specification.version"));
        System.out.println("Java运行时环境规范供应商：    " + props.getProperty("java.specification.vender"));
        System.out.println("Java运行时环境规范名称：    " + props.getProperty("java.specification.name"));
        System.out.println("Java的类格式版本号：    " + props.getProperty("java.class.version"));
        System.out.println("Java的类路径：    " + props.getProperty("java.class.path"));
        System.out.println("加载库时搜索的路径列表：    " + props.getProperty("java.library.path"));
        System.out.println("默认的临时文件路径：    " + props.getProperty("java.io.tmpdir"));
        System.out.println("一个或多个扩展目录的路径：    " + props.getProperty("java.ext.dirs"));
        System.out.println("操作系统的名称：    " + props.getProperty("os.name"));
        System.out.println("操作系统的构架：    " + props.getProperty("os.arch"));
        System.out.println("操作系统的版本：    " + props.getProperty("os.version"));
        System.out.println("文件分隔符：    " + props.getProperty("file.separator"));
        System.out.println("路径分隔符：    " + props.getProperty("path.separator"));
        System.out.println("行分隔符：    " + props.getProperty("line.separator"));
        System.out.println("用户的账户名称：    " + props.getProperty("user.name"));
        System.out.println("用户的主目录：    " + props.getProperty("user.home"));
        System.out.println("用户的当前工作目录：    " + props.getProperty("user.dir"));
    }

    private static void memory() throws SigarException {
        Sigar sigar = new Sigar();
        Mem mem = sigar.getMem();
        // 内存总量
        System.out.println("内存总量:    " + mem.getTotal() / 1024L + "K av");
        // 当前内存使用量
        System.out.println("当前内存使用量:    " + mem.getUsed() / 1024L + "K used");
        // 当前内存剩余量
        System.out.println("当前内存剩余量:    " + mem.getFree() / 1024L + "K free");
        Swap swap = sigar.getSwap();
        // 交换区总量
        System.out.println("交换区总量:    " + swap.getTotal() / 1024L + "K av");
        // 当前交换区使用量
        System.out.println("当前交换区使用量:    " + swap.getUsed() / 1024L + "K used");
        // 当前交换区剩余量
        System.out.println("当前交换区剩余量:    " + swap.getFree() / 1024L + "K free");
    }

    private static void cpu() throws SigarException {
        Sigar sigar = new Sigar();
        CpuInfo infos[] = sigar.getCpuInfoList();
        CpuPerc cpuList[] = null;

        System.out.println("cpu 总量参数情况:" + sigar.getCpu());
        System.out.println("cpu 总百分比情况:" + sigar.getCpuPerc());
        
        cpuList = sigar.getCpuPercList();
        for (int i = 0; i < infos.length; i++) {// 不管是单块CPU还是多CPU都适用
            CpuInfo info = infos[i];
            System.out.println("第" + (i + 1) + "块CPU信息");
            System.out.println("CPU的总量MHz:    " + info.getMhz());// CPU的总量MHz
            System.out.println("CPU生产商:    " + info.getVendor());// 获得CPU的卖主，如：Intel
            System.out.println("CPU类别:    " + info.getModel());// 获得CPU的类别，如：Celeron
            System.out.println("CPU缓存数量:    " + info.getCacheSize());// 缓冲存储器数量
            printCpuPerc(cpuList[i]);
        }
    }

    private static void printCpuPerc(CpuPerc cpu) {
        System.out.println("CPU用户使用率:    " + CpuPerc.format(cpu.getUser()));// 用户使用率
        System.out.println("CPU系统使用率:    " + CpuPerc.format(cpu.getSys()));// 系统使用率
        System.out.println("CPU当前等待率:    " + CpuPerc.format(cpu.getWait()));// 当前等待率
        System.out.println("CPU当前错误率:    " + CpuPerc.format(cpu.getNice()));//
        System.out.println("CPU当前空闲率:    " + CpuPerc.format(cpu.getIdle()));// 当前空闲率
        System.out.println("CPU总的使用率:    " + CpuPerc.format(cpu.getCombined()));// 总的使用率
    }

    private static void os() {
        OperatingSystem OS = OperatingSystem.getInstance();
        // 操作系统内核类型如： 386、486、586等x86
        System.out.println("操作系统:    " + OS.getArch());
        System.out.println("操作系统CpuEndian():    " + OS.getCpuEndian());//
        System.out.println("操作系统DataModel():    " + OS.getDataModel());//
        // 系统描述
        System.out.println("操作系统的描述:    " + OS.getDescription());
        // 操作系统类型
        // System.out.println("OS.getName():    " + OS.getName());
        // System.out.println("OS.getPatchLevel():    " + OS.getPatchLevel());//
        // 操作系统的卖主
        System.out.println("操作系统的卖主:    " + OS.getVendor());
        // 卖主名称
        System.out.println("操作系统的卖主名:    " + OS.getVendorCodeName());
        // 操作系统名称
        System.out.println("操作系统名称:    " + OS.getVendorName());
        // 操作系统卖主类型
        System.out.println("操作系统卖主类型:    " + OS.getVendorVersion());
        // 操作系统的版本号
        System.out.println("操作系统的版本号:    " + OS.getVersion());
    }

    private static void who() throws SigarException {
        Sigar sigar = new Sigar();
        Who who[] = sigar.getWhoList();
        if (who != null && who.length > 0) {
            for (int i = 0; i < who.length; i++) {
                // System.out.println("当前系统进程表中的用户名" + String.valueOf(i));
                Who _who = who[i];
                System.out.println("用户控制台:    " + _who.getDevice());
                System.out.println("用户host:    " + _who.getHost());
                // System.out.println("getTime():    " + _who.getTime());
                // 当前系统进程表中的用户名
                System.out.println("当前系统进程表中的用户名:    " + _who.getUser());
            }
        }
    }

    private static void file() throws Exception {
        Sigar sigar = new Sigar();
        FileSystem fslist[] = sigar.getFileSystemList();
        for (int i = 0; i < fslist.length; i++) {
            System.out.println("分区的盘符名称" + i);
            FileSystem fs = fslist[i];
            // 分区的盘符名称
            System.out.println("盘符名称:    " + fs.getDevName());
            // 分区的盘符名称
            System.out.println("盘符路径:    " + fs.getDirName());
            System.out.println("盘符标志:    " + fs.getFlags());//
            // 文件系统类型，比如 FAT32、NTFS
            System.out.println("盘符类型:    " + fs.getSysTypeName());
            // 文件系统类型名，比如本地硬盘、光驱、网络文件系统等
            System.out.println("盘符类型名:    " + fs.getTypeName());
            // 文件系统类型
            System.out.println("盘符文件系统类型:    " + fs.getType());
            FileSystemUsage usage = null;
            usage = sigar.getFileSystemUsage(fs.getDirName());
            switch (fs.getType()) {
            case 0: // TYPE_UNKNOWN ：未知
                break;
            case 1: // TYPE_NONE
                break;
            case 2: // TYPE_LOCAL_DISK : 本地硬盘
                // 文件系统总大小
                System.out.println(fs.getDevName() + "总大小:    " + usage.getTotal() + "KB");
                // 文件系统剩余大小
                System.out.println(fs.getDevName() + "剩余大小:    " + usage.getFree() + "KB");
                // 文件系统可用大小
                System.out.println(fs.getDevName() + "可用大小:    " + usage.getAvail() + "KB");
                // 文件系统已经使用量
                System.out.println(fs.getDevName() + "已经使用量:    " + usage.getUsed() + "KB");
                double usePercent = usage.getUsePercent() * 100D;
                // 文件系统资源的利用率
                System.out.println(fs.getDevName() + "资源的利用率:    " + usePercent + "%");
                break;
            case 3:// TYPE_NETWORK ：网络
                break;
            case 4:// TYPE_RAM_DISK ：闪存
                break;
            case 5:// TYPE_CDROM ：光驱
                break;
            case 6:// TYPE_SWAP ：页面交换
                break;
            }
            System.out.println(fs.getDevName() + "读出：    " + usage.getDiskReads());
            System.out.println(fs.getDevName() + "写入：    " + usage.getDiskWrites());
        }
        return;
    }

    private static void net() throws Exception {
        Sigar sigar = new Sigar();
        String ifNames[] = sigar.getNetInterfaceList();
        for (int i = 0; i < ifNames.length; i++) {
            String name = ifNames[i];
            NetInterfaceConfig ifconfig = sigar.getNetInterfaceConfig(name);
            System.out.println("网络设备名:    " + name);// 网络设备名
            System.out.println("IP地址:    " + ifconfig.getAddress());// IP地址
            System.out.println("子网掩码:    " + ifconfig.getNetmask());// 子网掩码
            if ((ifconfig.getFlags() & 1L) <= 0L) {
                System.out.println("!IFF_UP...skipping getNetInterfaceStat");
                continue;
            }
            NetInterfaceStat ifstat = sigar.getNetInterfaceStat(name);
            System.out.println(name + "接收的总包裹数:" + ifstat.getRxPackets());// 接收的总包裹数
            System.out.println(name + "发送的总包裹数:" + ifstat.getTxPackets());// 发送的总包裹数
            System.out.println(name + "接收到的总字节数:" + ifstat.getRxBytes());// 接收到的总字节数
            System.out.println(name + "发送的总字节数:" + ifstat.getTxBytes());// 发送的总字节数
            System.out.println(name + "接收到的错误包数:" + ifstat.getRxErrors());// 接收到的错误包数
            System.out.println(name + "发送数据包时的错误数:" + ifstat.getTxErrors());// 发送数据包时的错误数
            System.out.println(name + "接收时丢弃的包数:" + ifstat.getRxDropped());// 接收时丢弃的包数
            System.out.println(name + "发送时丢弃的包数:" + ifstat.getTxDropped());// 发送时丢弃的包数
        }
    }

    private static void ethernet() throws SigarException {
        Sigar sigar = null;
        sigar = new Sigar();
        String[] ifaces = sigar.getNetInterfaceList();
        for (int i = 0; i < ifaces.length; i++) {
            NetInterfaceConfig cfg = sigar.getNetInterfaceConfig(ifaces[i]);
            if (NetFlags.LOOPBACK_ADDRESS.equals(cfg.getAddress()) || (cfg.getFlags() & NetFlags.IFF_LOOPBACK) != 0
                    || NetFlags.NULL_HWADDR.equals(cfg.getHwaddr())) {
                continue;
            }
            System.out.println(cfg.getName() + "IP地址:" + cfg.getAddress());// IP地址
            System.out.println(cfg.getName() + "网关广播地址:" + cfg.getBroadcast());// 网关广播地址
            System.out.println(cfg.getName() + "网卡MAC地址:" + cfg.getHwaddr());// 网卡MAC地址
            System.out.println(cfg.getName() + "子网掩码:" + cfg.getNetmask());// 子网掩码
            System.out.println(cfg.getName() + "网卡描述信息:" + cfg.getDescription());// 网卡描述信息
            System.out.println(cfg.getName() + "网卡类型" + cfg.getType());//
        }
    }
}

```


- netty实现心跳检测  

ClienHeartBeattHandler  
在channelActive通道刚激活时，发送一个证书auth  
与服务端认证通过之后使用一个ScheduledThreadPool定时发送心跳信息  
心跳信息是本机的的一些信息，通过ChannelHandlerContext ctx对象发送  

```java
package bhz.netty.heartBeat;
public class ClienHeartBeattHandler extends ChannelHandlerAdapter {

    private ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
    
    private ScheduledFuture<?> heartBeat;
	//主动向服务器发送认证信息
    private InetAddress addr ;
    
    private static final String SUCCESS_KEY = "auth_success_key";

	@Override
	public void channelActive(ChannelHandlerContext ctx) throws Exception {
		addr = InetAddress.getLocalHost();
        String ip = addr.getHostAddress();
		String key = "1234";
		//证书
		String auth = ip + "," + key;
		ctx.writeAndFlush(auth);
	}
	
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    	try {
        	if(msg instanceof String){
        		String ret = (String)msg;
        		if(SUCCESS_KEY.equals(ret)){
        	    	// 握手成功，主动发送心跳消息
        	    	this.heartBeat = this.scheduler.scheduleWithFixedDelay(new HeartBeatTask(ctx), 0, 2, TimeUnit.SECONDS);
        		    System.out.println(msg);    			
        		}
        		else {
        			System.out.println(msg);
        		}
        	}
		} finally {
			ReferenceCountUtil.release(msg);
		}
    }

    private class HeartBeatTask implements Runnable {
    	private final ChannelHandlerContext ctx;

		public HeartBeatTask(final ChannelHandlerContext ctx) {
		    this.ctx = ctx;
		}
	
		@Override
		public void run() {
			try {
			    RequestInfo info = new RequestInfo();
			    //ip
			    info.setIp(addr.getHostAddress());
		        Sigar sigar = new Sigar();
		        //cpu prec
		        CpuPerc cpuPerc = sigar.getCpuPerc();
		        HashMap<String, Object> cpuPercMap = new HashMap<String, Object>();
		        cpuPercMap.put("combined", cpuPerc.getCombined());
		        cpuPercMap.put("user", cpuPerc.getUser());
		        cpuPercMap.put("sys", cpuPerc.getSys());
		        cpuPercMap.put("wait", cpuPerc.getWait());
		        cpuPercMap.put("idle", cpuPerc.getIdle());
		        // memory
		        Mem mem = sigar.getMem();
				HashMap<String, Object> memoryMap = new HashMap<String, Object>();
				memoryMap.put("total", mem.getTotal() / 1024L);
				memoryMap.put("used", mem.getUsed() / 1024L);
				memoryMap.put("free", mem.getFree() / 1024L);
				info.setCpuPercMap(cpuPercMap);
			    info.setMemoryMap(memoryMap);
			    ctx.writeAndFlush(info);
			    
			} catch (Exception e) {
				e.printStackTrace();
			}
		}

	    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
	    	cause.printStackTrace();
			if (heartBeat != null) {
			    heartBeat.cancel(true);
			    heartBeat = null;
			}
			ctx.fireExceptionCaught(cause);
	    }
	    
	}
}

```

ServerHeartBeatHandler  
先通过证书进行认证，然后返回SUCCESS_KEY  
获取心跳信息的内容并打印  
最后返回info received  

```java

package bhz.netty.heartBeat;

import io.netty.channel.ChannelFutureListener;
import io.netty.channel.ChannelHandlerAdapter;
import io.netty.channel.ChannelHandlerContext;

import java.util.HashMap;

public class ServerHeartBeatHandler extends ChannelHandlerAdapter {
    
	/** key:ip value:auth */
	private static HashMap<String, String> AUTH_IP_MAP = new HashMap<String, String>();
	private static final String SUCCESS_KEY = "auth_success_key";
	
	static {
		AUTH_IP_MAP.put("192.168.1.200", "1234");
	}
	
	private boolean auth(ChannelHandlerContext ctx, Object msg){
			//System.out.println(msg);
			String [] ret = ((String) msg).split(",");
			String auth = AUTH_IP_MAP.get(ret[0]);
			if(auth != null && auth.equals(ret[1])){
				ctx.writeAndFlush(SUCCESS_KEY);
				return true;
			} else {
				ctx.writeAndFlush("auth failure !").addListener(ChannelFutureListener.CLOSE);
				return false;
			}
	}
	
	@Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
		if(msg instanceof String){
			auth(ctx, msg);
		} else if (msg instanceof RequestInfo) {
			
			RequestInfo info = (RequestInfo) msg;
			System.out.println("--------------------------------------------");
			System.out.println("当前主机ip为: " + info.getIp());
			System.out.println("当前主机cpu情况: ");
			HashMap<String, Object> cpu = info.getCpuPercMap();
			System.out.println("总使用率: " + cpu.get("combined"));
			System.out.println("用户使用率: " + cpu.get("user"));
			System.out.println("系统使用率: " + cpu.get("sys"));
			System.out.println("等待率: " + cpu.get("wait"));
			System.out.println("空闲率: " + cpu.get("idle"));
			
			System.out.println("当前主机memory情况: ");
			HashMap<String, Object> memory = info.getMemoryMap();
			System.out.println("内存总量: " + memory.get("total"));
			System.out.println("当前内存使用量: " + memory.get("used"));
			System.out.println("当前内存剩余量: " + memory.get("free"));
			System.out.println("--------------------------------------------");
			
			ctx.writeAndFlush("info received!");
		} else {
			ctx.writeAndFlush("connect failure!").addListener(ChannelFutureListener.CLOSE);
		}
    }


}

```


- java虚拟机结构和组成  

1、类加载子系统负责从文件系统或者网络中加载Class信息，加载的类信息存放于一块称为  
方法区的内存空间。

2、除了类的信息外，方法区中可能还会存放运行时常量池信息，包括字符串  
字面量和数字常量（这部分常量信息是Class文件中常量池部分的内存映射）。  

3、java堆在虚拟机启动的时候建立，它是java程序最主要的内存工作区域。几乎所有的java  
对象实例都存放在java堆中。堆空间是所有线程共享的，这是一块与java应用密切相关的内存  
空间。  

4、java的NIO库允许java程序使用直接内存。直接内存是在java堆外的、直接向系统申请的内  
存空间。通常访问直接内存的速度会优于java堆。因此出于性能的考虑，读写频繁的场合可能会  
考虑使用直接内存。由于直接内存在java堆外，因此它的大小不会直接受限于Xmx指定的最大堆  
大小，但是系统内存是有限的，java堆和直接内存的总和依然受限于操作系统能给出的最大内存。  

5、垃圾回收系统是java虚拟机的重要组成部分，垃圾回收器可以对方法区、java堆和直接内  
存进行回收。其中，java堆是垃圾收集器的工作重点。和C/C++不同，java中所有的对象空间  
释放都是隐式的，也就是说，java中没有类似free()或者delete()这样的函数释放指定的内  
存区域。对于不再使用的垃圾对象，垃圾回收系统会在后台默默工作，默默查找、标识并释放  
垃圾对象，完成包括java堆、方法区和直接内存中的全自动化管理。  

6、每一个java虚拟机线程都有一个私有的java栈，一个线程的java栈在线程创建的时候被创  
建，java栈中保存着帧信息，java栈中保存着局部变量、方法参数，同时和java方法的调用、  
返回密切相关。

7、本地方法栈和java栈非常类似，最大的不同在于java栈用于方法的调用，而本地方法栈则  
用于本地方法的调用，作为对java虚拟机的重要扩展，java虚拟机允许java直接调用本地方法  
（通常使用C编写）

8、PC（Program Counter）寄存器也是每一个线程私有的空间，java虚拟机会为每一个java  
线程创建PC寄存器。在任意时刻，一个java线程总是在执行一个方法，这个正在被执行的方法  
称为当前方法。如果当前方法不是本地方法，PC寄存器就会指向当前正在被执行的指令。如果当  
前方法是本地方法，那么PC寄存器的值就是undefined  

9、执行引擎是java虚拟机的最核心组件之一，它负责执行虚拟机的字节码，现代虚拟机为了提  
高执行效率，会使用即时编译技术将方法编译成机器码后再执行  


- 堆、栈、方法区概念和联系  

堆解决的事数据存储的问题，即数据怎么放，放在哪  
栈解决程序的运营问题，即程序如何执行，如何处理数据  
方法区则是辅助堆栈的永久区，解决堆栈信息的产生，是先决条件。  
比如我 我们创建一个新的对象，User:那么User类的一些信息（类信息，静态信息都存在于方法区中）  
而User类被实例化出来之后，被存储到Java堆中，一块内存空间 当我们去使用的时候，都是使用User  
对象的引用，如User user=new User();  
这里的user就是存放在Java栈中的，即User真实对象的一个引用  


- java堆  
java堆和java应用程序关系最密切的内存空间，几乎所有对象都存放在其中，并且Java堆完全是自动化  
管理的，通过垃圾回收机制，垃圾对象会自动清理，不需要显示地释放  

根据垃圾回收机制不同，java堆有可能拥有不用的结构，最为常见的将整个Java堆分为新生代和老年代。  
新生代分为eden区，s0区，s1区 ，  
s1和s0也被称为to和from区域，他们是两块大小相等并且可以互换角色的空间。  
绝大多数情况下，对象先分配在eden区，在一次新生代回收后，如果对象还存活，则会进入s0或s1区，  
之后每经过一次新生代回收，如果对象存活则他的年龄就加1，当对象达到一定年龄后，则进入老年代。  
（s0 和 s1 采用复制算法 相互复制回收不使用的对象）  

- java栈  
Java栈是一块线程私有的空间，一个栈，一般由3部分组成：局部变量表，操作数据栈，和帧数据区。  
局部变量表：用于报错函数的参数及局部变量  
操作数据栈：主要保存计算过程的中间结果，同时作为计算过程中的变量临时的存储空间。  
帧数据区： 除了局部变量表和操作数据栈以外，栈还需要一些数据来支持常量池的解析，这里帧数据区  
保存着访问常量池的指针，方便程序访问常量池，另外当函数返回或出现异常时，虚拟机必须有一个异常  
处理表，方便发送异常的时候找到异常的代码，因此异常处理表也是帧数据区的一部分。  

- java方法区  
Java方法区和堆一样，方法区是一块所有线程共享的内存区域，他保存系统的类信息，比如类的字段，  
方法，常量池等，方法区的大小决定系统可以保存多少个类，如果系统定义太多个类，导致方法区溢出，  
虚拟机同样会抛出内存溢出的错误，方法区可以理解为永久区。  


- java堆分配参数  

-XX:PrintGC：使用这个参数，虚拟机启动后，只要遇到GC就会打印日志  
-XX:+UserSerialGC：配置串行器  
-XX:PrintGCDetails：可以查看详细信息，包括各个区的情况  
-Xms：设置java程序启动时初始堆大小  
-Xmx：设置java程序能获得的最大堆大小  
-Xmx20m -Xms5m -XX:PrintCommandLineFlags：可以将隐式或者显示传给虚拟机的参数输出  

将注释的内容配置到VM argument，就会生效  

```java
package com.bjsxt.base001;
public class Test01 {

	public static void main(String[] args) {
		//-Xms5m -Xmx20m -XX:+PrintGCDetails -XX:+UseSerialGC -XX:+PrintCommandLineFlags
		//查看GC信息
		System.out.println("max memory:" + Runtime.getRuntime().maxMemory());
		System.out.println("free memory:" + Runtime.getRuntime().freeMemory());
		System.out.println("total memory:" + Runtime.getRuntime().totalMemory());
		
		byte[] b1 = new byte[1*1024*1024];
		System.out.println("分配了1M");
		System.out.println("max memory:" + Runtime.getRuntime().maxMemory());
		System.out.println("free memory:" + Runtime.getRuntime().freeMemory());
		System.out.println("total memory:" + Runtime.getRuntime().totalMemory());
		
		byte[] b2 = new byte[4*1024*1024];
		System.out.println("分配了4M");
		System.out.println("max memory:" + Runtime.getRuntime().maxMemory());
		System.out.println("free memory:" + Runtime.getRuntime().freeMemory());
		System.out.println("total memory:" + Runtime.getRuntime().totalMemory());
	}
}
```


堆内存配置参数二  

-Xmn：设置新生代的大小，设置一个比较大的新生代会减少老年代的大小，这个参数对系统性能  
以及GC行为有很大的影响，新生代大小一般会设置整个堆空间的1/3到1/4左右。  

-XX:NewRatio=老年代/新生代：设置新生代和老年代的比例  

-XX:SurvivorRatio：用来设置新生代中eden空间和from/to空间的比例  
含义:-XX:SurvivorRatio=eden/from=eden/to  

```java
package com.bjsxt.base001;

public class Test02 {

	public static void main(String[] args) {
		
		//第一次配置
		//-Xms20m -Xmx20m -Xmn1m -XX:SurvivorRatio=2 -XX:+PrintGCDetails -XX:+UseSerialGC
		
		//第二次配置
		//-Xms20m -Xmx20m -Xmn7m -XX:SurvivorRatio=2 -XX:+PrintGCDetails -XX:+UseSerialGC
		
		//第三次配置
		//-XX:NewRatio=老年代/新生代
		//-Xms20m -Xmx20m -XX:SurvivorRatio=2 -XX:+PrintGCDetails -XX:+UseSerialGC
		
		byte[] b = null;
		//连续向系统申请10MB空间
		for(int i = 0 ; i <10; i ++){
			b = new byte[1*1024*1024];
		}
	}
}

```

堆内存溢出  

```java
package com.bjsxt.base001;

import java.util.Vector;

public class Test03 {

	public static void main(String[] args) {
		
		//-Xms1m -Xmx1m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=d:/Test03.dump
		//堆内存溢出
		Vector v = new Vector();
		for(int i=0; i < 5; i ++){
			v.add(new Byte[1*1024*1024]);
		}
		
	}
}

```

线程最大的栈空间  

```java
package com.bjsxt.base001;

public class Test04 {

	//-Xss1m  
	//-Xss5m
	
	//栈调用深度
	private static int count;
	
	public static void recursion(){
		count++;
		recursion();
	}
	public static void main(String[] args){
		try {
			recursion();
		} catch (Throwable t) {
			System.out.println("调用最大深入：" + count);
			t.printStackTrace();
		}
	}
}

```

- 垃圾回收算法  
垃圾回收有很多种算法，如引用计数法，标记压缩法、复制算法、分带、分区的思想。

引用计数法  
这是一个比较古老而经典的垃圾回收算法，其核心是在对象被其他所引用时计数器加1，  
而当引用失效时则减一，但这种方法有非常严重的问题，无法处理循环引用的问题，还  
有就是每次进行加减操作比较浪费系统性能。  

标记清除法  
就是分别标记和清除两个阶段进行处理内存中的对象，当然这种方式也有很大的弊端，  
就是空间碎片问题，垃圾回收后的空间不是连续的，不连续的内存空间的工作效率要低  
于连续的内存空间。  

复制算法（新生代）  
其核心思想将内存空间分为两块，每次只使用其中一块，在垃圾回收时将正在使用的内  
存中的残留对象复制到未被使用的内存块中，之后去清除之前正在使用的内存块中所有  
的对象，反复去交换两个内存的角色，完成垃圾回收。（java中新生代的from和to空间  
就是使用了这个算法）  

标记压缩法（老年代）  
标记压缩法在标记清除法基础上做了优化，把存活的对象压缩到内存的一端，而后进行  
垃圾清理。  

为什么新生代和老年代使用不同的算法？  
新生代对象被回收的几率要大于老年代中对象被回收的概率，老年代中的对象都较为稳  
定。采用不同的算法是为了性能优化考虑。  

分代算法  
根据对象的特点即将内存分为N块，而后根据内存的特点使用不同的算法。   
对于新生代和老生代来说，新生代回收频率较高，但是每次回收耗时都很短，而老年代  
回收频率较低，但是耗时会相对较长，所以应该尽量减少老年代的GC。  

分区算法  
分区算法：其主要就是将整个内存分为N多个独立空间，每个小空间都可以独立使用，这  
样细粒度的控制一次回收多个小空间而不是针对整个空间进行GC，从而提升性能，并减少  
GC的停顿时间。  



- 垃圾回收时的停顿现象  
垃圾回收的任务是识别和回收垃圾对象进行内存清理，为了让垃圾回收器可以更高效的执行，  
大部分情况下，会要求系统进如一个停顿的状态。停顿的目的是为了终止所有的应用线程，只  
有这样的系统才不会有新垃圾的产生。同时停顿保证了系统状态在某一个瞬间的一致性，也有  
利于更好的标记垃圾对象。因此在垃圾回收时，都会产生应用程序的停顿。  




- 对象如何进入老年代
一般而言对象首次创建会被放置在新生代的eden区，如果没有GC介入，则对象不会离开eden区，  
那么eden区的对象如何进入老年代那呢？一般来说只要对象的年龄达到一定的大小，就会自动  
离开年轻代进入老年代，对象年龄是由对象经历数次GC决定的，在新生代每次GC后，如果没有  
被回收则年龄加1.虚拟机提供一个参数来控制新生代对象的最大年龄，当超过这个年龄范围就  
会晋升老年代，使用-XX:MaxTenuringThreshold进行配置多少次gc进入老年代  

```java
package com.bjsxt.base001;

import java.util.HashMap;
import java.util.Map;

public class Test05 {

	public static void main(String[] args) {
		//初始的对象在eden区
		//参数：-Xmx64M -Xms64M -XX:+PrintGCDetails
		for(int i=0; i< 5; i++){
			byte[] b = new byte[1024*1024];
		}
		
		
		
		//测试进入老年代的对象
		//
		//参数：-Xmx1024M -Xms1024M -XX:+UseSerialGC -XX:MaxTenuringThreshold=15 -XX:+PrintGCDetails 
		//-XX:+PrintHeapAtGC
/*		
		Map<Integer, byte[]> m = new HashMap<Integer, byte[]>();
		for(int i =0; i <5 ; i++) {
			byte[] b = new byte[1024*1024];
			m.put(i, b);
		}
		
		for(int k = 0; k<20; k++) {
			for(int j = 0; j<300; j++){
				byte[] b = new byte[1024*1024]; 
			}
		}
*/	
	
	}
}

```

根据设置MaxTenuringThreshold参数可以指定新生代对象经过多少次回收后进入老年代。  
另外，大对象(新生代eden区无法装入时，也会直接进入老年代)。JVM有个参数可以设置对  
象的大小超过指定的大小之后，直接晋升老年代。 使用-XX:PretenureSizeThreshold  

虚拟机对于体积不大的对象 会优先把数据分配到TLAB区域中，因此就失去了在老年代分配的机会  
使用-XX:-UseTLAB会禁用TLAB区域  

```java
package com.bjsxt.base001;

import java.util.HashMap;
import java.util.Map;

public class Test06 {

	public static void main(String[] args) {
		
		//参数：-Xmx30M -Xms30M -XX:+UseSerialGC -XX:+PrintGCDetails -XX:PretenureSizeThreshold=1000
		//这种现象原因为：虚拟机对于体积不大的对象 会优先把数据分配到TLAB区域中，因此就失去了在老年代分配的机会
		//参数：-Xmx30M -Xms30M -XX:+UseSerialGC -XX:+PrintGCDetails -XX:PretenureSizeThreshold=1000 -XX:-UseTLAB
		Map<Integer, byte[]> m = new HashMap<Integer, byte[] >();
		for(int i=0; i< 5*1024; i++){
			byte[] b = new byte[1024];
			m.put(i, b);
		}
	}
}

```


TLAB

TLAB全称是Thread Local Allocation Buffer即本地分配缓存，从名字上看是一个线程  
专用的内存分配区域，是为了加速对象分配而生的。每一个线程会产生一个TLAB，该线程独  
享的一个工作区域，java虚拟机使用这种TLAB来避免多线程冲突问题。提高了对象分配的效  
率。TLAB空间一般不会太大，当大对象无法在TLAB分配时，则会直接分配在堆上。   
-XX:+UseTLAB 使用TLAB  
-XX:+TLABSize设置TLAB大小  
-XX:TLABRefillWasterFraction 设置维护进入TLAB空间的单个对象大小，它是一个比例  
值，默认64，即如果对象大于整个空间的1/64,则在堆创建对象。  
-XX:+PrintTLAB查看TLAB信息  
-XX:ResizeTLAB自调整TLABRefillWasteFraction阈值。  
  
![](http://i1.bvimg.com/567571/3947acc73907f3f5.png)  

```java
package com.bjsxt.base001;
public class Test07 {

	public static void alloc(){
		byte[] b = new byte[2];
	}
	public static void main(String[] args) {
		
		//TLAB分配
		//参数：-XX:+UseTLAB -XX:+PrintTLAB -XX:+PrintGC -XX:TLABSize=102400 -XX:-ResizeTLAB -XX:TLABRefillWasteFraction=100 -XX:-DoEscapeAnalysis -server
		for(int i=0; i<10000000;i++){
			alloc();
		}
	}
}

```


- Java有四种类型的垃圾回收器：  

串行垃圾回收器（Serial Garbage Collector）  
并行垃圾回收器（Parallel Garbage Collector）  
并发标记扫描垃圾回收器（CMS Garbage Collector）  
G1垃圾回收器（G1 Garbage Collector）  
 
串行垃圾收集器  
串行回收器是指使用单线程进行垃圾回收的回收器。每次回收时，串行回收器只有一个  
工作线程，对于并行能力较弱的计算机来说，串行回收器的专注性和独占性往往有更好  
的性能表现。串行回收器可以在新生代和老年代使用。根据作用于不同的对空间分为新生  
代串行回收器和老年代串行回收器。   
-XX：+UseSerialGC参数可以设置使用新生代串行回收器和老年代串行回收器。  

并行垃圾收集器  
并行的垃圾回收器在串行回收器的基础上做了改进，他可以使用多个线程同时进行垃圾  
回收，对于计算能力强的计算机而言，可以有效的缩短垃圾回收所需的实际时间。  

1、ParNew回收器是一个工作在新生代的垃圾回收器，他只是简单的将串行回收器多线程化，  
它的回收策略和算法和串行回收器一样。   
使用-XX:+UseParNewGC 新生代使用ParNew回收器，老年代则使用串行回收器。   
ParNew回收器工作时的线程数量可以使用-XX：ParallelGCThreads参数指定，一般  
最好和计算机的CPU相当，避免过多的线程影响性能。  

2、新生代ParallelGC回收器，使用了复制算法的回收器，也是多线程独占形式的回收器，  
但ParallelGC回收器有一个很重要的特点，就是它非常关注吞吐量。   
提供了两个参数控制系统的吞吐量 
-XX: MaxGCPauseMillis:设置最大垃圾收集停顿时间，可用于把虚拟机在GC停顿的时  
间控制在MaxGCPauseMillis范围内，如果希望减少GC停顿时间可以将MaxGCPauseMillis  
设置的很小，但是会导致GC频繁，从而增加GC的总时间，降低了吞吐量。所以要根据实际  
情况设置该值。 
-XX: GCTimeRatio:设置吞吐量的大小，它是一个0到100之间的整数，默认情况下它的  
取值是99，那么系统将花费不超过1/(1+n)的时间用于垃圾回收，也就是1/(1+99)=1%的时间。   
另外还可以指定-XX：+UseAdaptiveSizePolicy打开自适应模式，在这种模式下，新生  
代的大小、eden、from/to的比例，以及晋升老年代的对象的年龄参数将被自动调整，以  
达到在堆大小、吞吐量和停顿时间之间的平衡点。  

3、老年代ParallelOldGC回收器也是一种多线程的回收器，和新生代的ParallelGC回收器一样，  
也是一种关注吞吐量的回收器，它使用标记压缩算法实现。   
-XX:+UseParallelOldGC进行设置  
-XX:+ParallelGCThreads也可以设置垃圾收集时的线程数量。  


CMS回收器  
CMS全称为：Current Mark Sweep意为并发标记清除，他使用的是标记清除法，  
主要关注系统的停顿时间。   
使用-XX:+UseConcMarkSweepGC进行设置  
使用-XX:+ConcGCThreads设置并发线程数量  

CMS并不是独占的回收器，也就是说CMS回收的过程中，应用程序仍在不断的运行，又会有新的  
垃圾不断的产生，所以在使用CMS的过程中应该确保应用程序的内存足够用。CMS不会等到应用  
程序饱和的时候才去回收垃圾，而是在某一阈值的时候开始回收，回收阈值可以用指定的参数  
进行配置，-XX:CMSInitiatingOccupancyFraction来指定，默认值为68，也就是说当老年  
代的空间使用率达到68%的时候，会执行CMS回收。如果内存使用率增长很快，在CMS执行的过  
程中，已经出现内存不足的情况，此时CMS回收就会失败，虚拟机将启动老年代串行回收器进  
行垃圾回收，这会使应用程序中断，直到垃圾回收完成后才会正常工作。这个过程GC停顿的时  
间可能比较长，所以-XX:CMSInitiatingOccupancyFraction的设置要根据实际情况。  

之前我们在学习算法的时候说过，标记清除法有个缺点就是内存碎片的问题，那么CMS有个参数  
设置-XX:+UseCMSCompactAtFullCollection可以使CMS回收完成之后进行一次碎片整理，  
-XX:CMSFullGCsBeforeCompaction参数可以设置进行多少次CMS回收之后对内存进行一次压缩。  


G1回收器 
G1回收器(Garbage-First)是在jdk1.7中提出的垃圾回收器，从长期目标来看是为了取代CMS  
回收器，G1回收器拥有独特的垃圾回收策略，G1属于分代垃圾回收器，区分新生代和老年代，  
依然有eden和from/to区,并不要求整个eden区或新生代、老年代空间都连续，它使用的是分区算法。   
并行性： G1回收期间可多线程同时工作。   
并发性： G1拥有与应用程序交替执行的能力，部分工作可与应用程序同时执行，在整个GC期间  
不会完全阻塞应用程序。   
分代GC：  
G1依然是一个分代收集器，但是它是兼顾新生代和老年代一起工作的，之前的垃圾收集器或者在  
新生代工作或者在老年代工作，因此这是一个很大的不同。   
空间整理： G1在回收过程中，不会像CMS那样在若干次GC后进行碎片整理，G1采用有效复制对象  
的方式减少空间碎片。  
可预见性：由于分区的原因，G1可以只选取部分区域进行回收，缩小了回收的范围，提高了性能。  
使用-XX:+UseG1GC应用G1收集器  
使用-XX:MaxGCPauseMillis指定最大的停顿时间  
使用-XX:ParallelGCThreads设置并行回收的线程数量  


配置Tomcat参数（一）  
使用串行垃圾回收器  
-XX:+PrintGCDetails -Xmx32m -Xms32m  
-XX:+HeapDumpOnOutOfMemoryError  
-XX:+UseSerialGC  
-XX:PermSize=32m  

打印信息、堆最大32m、初始32m  
打印堆溢出异常  
使用串行化gc  
方法区32m  
测试结果显示吞吐量为:871.8/sec 87KB/sec  


配置Tomcat参数（二）  
扩大内存提高系统性能  
-XX:+PrintGCDetails -Xmx512m -Xms32m  
-XX:+HeapDumpOnOutOfMemoryError  
-XX:+UseSerialGC  
-XX:PermSize=32m  
-Xloggc:d:/gc.log  
测试结果显示吞吐量为:1383.31/sec 138KB/sec

配置Tomcat参数（三）  
调整初始堆大小  
-XX:+PrintGCDetails -Xmx512m -Xms128m  
-XX:+HeapDumpOnOutOfMemoryError  
-XX:+UseSerialGC  
-XX:PermSize=32m  
-Xloggc:d:/gc.log  
测试结果显示吞吐量为:1501.3/sec 149.8KB/sec  

配置Tomcat参数（四）  
测试ParNew回收器的表现  
-XX:+PrintGCDetails -Xmx512m -Xms128m  
-XX:+HeapDumpOnOutOfMemoryError  
-XX:+UseParNewGC  
-XX:PermSize=32m  
-Xloggc:d:/gc.log  
测试结果显示吞吐量为:2130/sec 212KB/sec  

配置Tomcat参数（五）  
测试ParallelOldGC回收器的表现  
-XX:+PrintGCDetails -Xmx512m -Xms128m  
-XX:+HeapDumpOnOutOfMemoryError  
-XX:+UseParallelGC  
-XX:+UseParallelOldGC  
-XX:ParallelGCThreads=8  
-XX:PermSize=32M  
-Xloggc:d:/gc.log  
测试结果显示吞吐量为:2236/sec 223KB/sec  

配置Tomcat参数（五）  
测试CMS回收器的表现  
-XX:+PrintGCDetails -Xmx512m -Xms128m  
-XX:+HeapDumpOnOutOfMemoryError  
-XX:+UseConcMarkSweepGC  
-XX:ConcGCThreads=8  
-XX:PermSize=32M  
-Xloggc:d:/gc.log  
测试结果显示吞吐量为:1813/sec 181KB/sec  

# RocketMQ  


- 选用理由：  
强调集群无单点，可扩展，任意一点高可用，水平可扩展。  
海量消息堆积能力，消息堆积后，写入低延迟。  
支持上万个队列  
消息失败重试机制  
消息可查询  
开源社区活跃  
成熟度（经过双十一考验）  

单个Master  
这种方式风险较大，一旦Broker重启或者宕机时，会导致整个服务不可用，不建议线上环境使用。  

多Master模式  
一个集群无Slave，全是Master，例如2个Master或者3个Master  
优点：配置简单，单个Master 宕机或重启维护对应用无影响，在磁盘配置为RAID10 时，  
即使机器宕机不可恢复情况下，由于RAID10磁盘非常可靠，消息也不会丢（异步刷盘丢失  
少量消息，同步刷盘一条不丢）。性能最高。  
缺点：单台机器宕机期间，这台机器上未被消费的消息在机器恢复之前不可订  
阅，消息实时性会受到受到影响。  
 ### 先启动 NameServer  
 ### 在机器 A，启动第一个 Master  
 ### 在机器 B，启动第二个 Master  

多Master 多Slave模式，异步复制  
每个 Master 配置一个 Slave，有多对Master-Slave，HA  
采用异步复制方式，主备有短暂消息延迟，毫秒级。  
优点：即使磁盘损坏，消息丢失的非常少，且消息实时性不会受影响，因为  
Master 宕机后，消费者仍然可以从 Slave消费，此过程对应用透明。不需要  
人工干预。性能同多 Master 模式几乎一样。  
缺点：Master 宕机，磁盘损坏情况，会丢失少量消息。  
 ### 先启动 NameServer  
 ### 在机器 A，启动第一个 Master  
 ### 在机器 B，启动第二个 Master  
 ### 在机器 C，启动第一个 Slave  
 ### 在机器 D，启动第二个 Slave  

多Master多Slave模式，同步双写  
每个Master配置一个 Slave，有多对Master-Slave，HA  
采用同步双写方式，主备都写成功，向应用返回成功。  
优点：数据与服务都无单点，Master宕机情况下，消息无延迟，服务可用性与  
数据可用性都非常高  
缺点：性能比异步复制模式略低，大约低 10%左右，发送单个消息的RT  
会略高。目前主宕机后，备机不能自动切换为主机，后续会支持自动切换功能。  


- RocketMQ部署【双Master方式】  

总结：先启动NameServer再启动BrokerServer  

1.服务器环境  

序号  | IP             | 用户名  |角色                      | 模式  
-|-|-|-|-
1     |192.168.100.24 |root   |nameServer1,brokerServer1 |Master1  
2     |192.168.100.25 |root   |nameServer2,brokerServer2 |Master2  

2.Hosts添加信息  
`# vi /etc/hosts`  
IP NAME  
192.168.100.24 rocketmq-nameserver1  
192.168.100.24 rocketmq-master1  
192.168.100.25 rocketmq-nameserver2  
192.168.100.25 rocketmq-master2  

3.上传解压【两台机器】  

```
# 上传alibaba-rocketmq-3.2.6.tar.gz文件至/usr/local
# tar -zxvf alibaba-rocketmq-3.2.6.tar.gz -C /usr/local
# mv alibaba-rocketmq alibaba-rocketmq-3.2.6  #改名
# ln -s alibaba-rocketmq-3.2.6 rocketmq   #软连接
  ll /usr/local
```

4.创建存储路径【两台机器】  

```
# mkdir /usr/local/rocketmq/store
# mkdir /usr/local/rocketmq/store/commitlog
# mkdir /usr/local/rocketmq/store/consumequeue
# mkdir /usr/local/rocketmq/store/index
```

5.RocketMQ配置文件【两台机器】  

```
# vim /usr/local/rocketmq/conf/2m-noslave/broker-a.properties
# vim /usr/local/rocketmq/conf/2m-noslave/broker-b.properties
```

```
#所属集群名字
brokerClusterName=rocketmq-cluster
#broker名字，注意此处不同的配置文件填写的不一样
brokerName=broker-a|broker-b
#0 表示 Master，>0 表示 Slave
brokerId=0
#nameServer地址，分号分割
namesrvAddr=rocketmq-nameserver1:9876;rocketmq-nameserver2:9876
#在发送消息时，自动创建服务器不存在的topic，默认创建的队列数
defaultTopicQueueNums=4
#是否允许 Broker 自动创建Topic，建议线下开启，线上关闭
autoCreateTopicEnable=true
#是否允许 Broker 自动创建订阅组，建议线下开启，线上关闭
autoCreateSubscriptionGroup=true
#Broker 对外服务的监听端口
listenPort=10911
#删除文件时间点，默认凌晨 4点
deleteWhen=04
#文件保留时间，默认 48 小时
fileReservedTime=120
#commitLog每个文件的大小默认1G
mapedFileSizeCommitLog=1073741824
#ConsumeQueue每个文件默认存30W条，根据业务情况调整
mapedFileSizeConsumeQueue=300000
#destroyMapedFileIntervalForcibly=120000
#redeleteHangedFileInterval=120000
#检测物理文件磁盘空间
diskMaxUsedSpaceRatio=88
#存储路径
storePathRootDir=/usr/local/rocketmq/store
#commitLog 存储路径
storePathCommitLog=/usr/local/rocketmq/store/commitlog
#消费队列存储路径存储路径
storePathConsumeQueue=/usr/local/rocketmq/store/consumequeue
#消息索引存储路径
storePathIndex=/usr/local/rocketmq/store/index
#checkpoint 文件存储路径
storeCheckpoint=/usr/local/rocketmq/store/checkpoint
#abort 文件存储路径
abortFile=/usr/local/rocketmq/store/abort
#限制的消息大小
maxMessageSize=65536
#flushCommitLogLeastPages=4
#flushConsumeQueueLeastPages=2
#flushCommitLogThoroughInterval=10000
#flushConsumeQueueThoroughInterval=60000
#Broker 的角色
#- ASYNC_MASTER 异步复制Master
#- SYNC_MASTER 同步双写Master
#- SLAVE
brokerRole=ASYNC_MASTER
#刷盘方式
#- ASYNC_FLUSH 异步刷盘
#- SYNC_FLUSH 同步刷盘
flushDiskType=ASYNC_FLUSH
#checkTransactionMessageEnable=false
#发消息线程池数量
#sendMessageThreadPoolNums=128
#拉消息线程池数量
#pullMessageThreadPoolNums=128
```

6.修改日志配置文件【两台机器】  

```
# mkdir -p /usr/local/rocketmq/logs
# cd /usr/local/rocketmq/conf && sed -i 's#${user.home}#/usr/local/rocketmq#g' 
*.xml
```

7.修改启动脚本参数【两台机器】  

```
# vim /usr/local/rocketmq/bin/runbroker.sh
```

```
#============================================================
# 开发环境JVM Configuration
#============================================================
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m -
XX:PermSize=128m -XX:MaxPermSize=320m"
# vim /usr/local/rocketmq/bin/runserver.sh
JAVA_OPT="${JAVA_OPT} -server -Xms1g -Xmx1g -Xmn512m -
XX:PermSize=128m -XX:MaxPermSize=320m"
```

8.启动NameServer【两台机器】  

```
# cd /usr/local/rocketmq/bin
# nohup sh mqnamesrv &
```

9.启动BrokerServer A【192.168.100.24】  

```
# cd /usr/local/rocketmq/bin
# nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-a.properties >/dev/null 2>&1 &
# netstat -ntlp
# jps
# tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
# tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log
```

10.启动BrokerServer B【192.168.100.25】  

```
# cd /usr/local/rocketmq/bin
# nohup sh mqbroker -c /usr/local/rocketmq/conf/2m-noslave/broker-b.properties >/dev/null 2>&1 &
# netstat -ntlp
# jps
# tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/broker.log
# tail -f -n 500 /usr/local/rocketmq/logs/rocketmqlogs/namesrv.log
```

11.RocketMQ Console  
在tomcat中部署rocketmq-console.war  

12.数据清理  

```
# cd /usr/local/rocketmq/bin
# sh mqshutdown broker
# sh mqshutdown namesrv
# --等待停止
# rm -rf /usr/local/rocketmq/store
# mkdir /usr/local/rocketmq/store
# mkdir /usr/local/rocketmq/store/commitlog
# mkdir /usr/local/rocketmq/store/consumequeue
# mkdir /usr/local/rocketmq/store/index
# --按照上面步骤重启NameServer与BrokerServer
```

Producer  
发送的内容包括 topic ， tag ， body  
quickstart_producer 名称在服务器中应该唯一  
设置一个 setNamesrvAddr  

```java
package com.alibaba.rocketmq.example.quickstart;

import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;
import com.alibaba.rocketmq.client.producer.SendResult;
import com.alibaba.rocketmq.common.message.Message;
/**
 * Producer，发送消息
 * 
 */
public class Producer {
    public static void main(String[] args) throws MQClientException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("quickstart_producer");
        producer.setNamesrvAddr("192.168.1.121:9876";"192.168.1.122:9876");

        producer.start();

        for (int i = 0; i < 1000; i++) {
            try {
                Message msg = new Message("TopicTest",// topic
                    "TagA",// tag
                    ("Hello RocketMQ " + i).getBytes()// body
                        );
                SendResult sendResult = producer.send(msg);
                System.out.println(sendResult);
            }
            catch (Exception e) {
                e.printStackTrace();
                Thread.sleep(1000);
            }
        }

        producer.shutdown();
    }
}


```

consumer  
consumer也要设置setNamesrvAddr  
setConsumeFromWhere 设置从哪开始消费  
subscribe 订阅主题和标签  

```java
package com.alibaba.rocketmq.example.quickstart;
import java.util.List;
import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;
import com.alibaba.rocketmq.common.message.MessageExt;
/**
 * Consumer，订阅消息
 */
public class Consumer {

    public static void main(String[] args) throws InterruptedException, MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("quickstart_consumer");
        consumer.setNamesrvAddr("192.168.1.121:9876";"192.168.1.122:9876");
        consumer.setConsumeMessageBatchMaxSize(10);  
        /** 
         * 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费<br> 
         * 如果非第一次启动，那么按照上次消费的位置继续消费 
         */  
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);  
  
        consumer.subscribe("TopicTest", "*");  
  
        consumer.registerMessageListener(new MessageListenerConcurrently() {  
  
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
                  
                try {  
                    System.out.println("msgs的长度" + msgs.size());  
                    System.out.println(Thread.currentThread().getName() + " Receive New Messages: " + msgs);  
                } catch (Exception e) {  
                    e.printStackTrace();  
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;  
                }  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
            }  
        });  
  
        consumer.start();  
  
        System.out.println("Consumer Started.");  
    }
}

```

Consumer 批量消费  
可以通过consumer.setConsumeMessageBatchMaxSize(10);//每次拉取10条  

如果是Consumer先启动，所以他回去轮询MQ上是否有订阅队列的消息，  
由于每次producer插入一条，Consumer就拿一条所以测试结果如下（每次size都是1）  

Consumer端后启动，也就是Producer先启动  
由于这里是Consumer后启动，所以MQ上也就堆积了一堆数据，Consumer每次拉取10条    



消息重试机制：消息重试分为2种：Producer端重试、Consumer端重试  

1、Producer端重试  
也就是Producer往MQ上发消息没有发送成功，我们可以设置发送失败重试的次数  
producer.setRetryTimesWhenSendFailed(10);//失败的 情况发送10次  

```java
package quickstart;  
  
import com.alibaba.rocketmq.client.exception.MQClientException;  
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;  
import com.alibaba.rocketmq.client.producer.SendResult;  
import com.alibaba.rocketmq.common.message.Message;  
  
/** 
 * Producer，发送消息 
 *  
 */  
public class Producer {  
    public static void main(String[] args) throws MQClientException, InterruptedException {  
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");  
        producer.setNamesrvAddr("192.168.100.145:9876;192.168.100.146:9876");  
        producer.setRetryTimesWhenSendFailed(10);//失败的 情况发送10次  
        producer.start();  
  
        for (int i = 0; i < 1000; i++) {  
            try {  
                Message msg = new Message("TopicTest",// topic  
                        "TagA",// tag  
                        ("Hello RocketMQ " + i).getBytes()// body  
                );  
                SendResult sendResult = producer.send(msg);  
                System.out.println(sendResult);  
            } catch (Exception e) {  
                e.printStackTrace();  
                Thread.sleep(1000);  
            }  
        }  
  
        producer.shutdown();  
    }  
}  

```

Consumer端重试  
2.1、exception的情况，一般重复16次 10s、30s、1分钟、2分钟、3分钟等等  
代码中消费异常的情况返回  
return ConsumeConcurrentlyStatus.RECONSUME_LATER;//重试  
正常则返回：  
return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;//成功  

```java
package quickstart;  
  
  
import java.util.List;  
  
import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;  
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;  
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;  
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;  
import com.alibaba.rocketmq.client.exception.MQClientException;  
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;  
import com.alibaba.rocketmq.common.message.MessageExt;  
  
/** 
 * Consumer，订阅消息 
 */  
public class Consumer {  
  
    public static void main(String[] args) throws InterruptedException, MQClientException {  
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_4");  
        consumer.setNamesrvAddr("192.168.100.145:9876;192.168.100.146:9876");  
        consumer.setConsumeMessageBatchMaxSize(10);  
        /** 
         * 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费<br> 
         * 如果非第一次启动，那么按照上次消费的位置继续消费 
         */  
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);  
  
        consumer.subscribe("TopicTest", "*");  
  
        consumer.registerMessageListener(new MessageListenerConcurrently() {  
  
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
  
                try {  
                    // System.out.println("msgs的长度" + msgs.size());  
                    System.out.println(Thread.currentThread().getName() + " Receive New Messages: " + msgs);  
                    for (MessageExt msg : msgs) {  
                        String msgbody = new String(msg.getBody(), "utf-8");  
                        if (msgbody.equals("Hello RocketMQ 4")) {  
                            System.out.println("======错误=======");  
                            int a = 1 / 0;  
                        }  
                    }  
  
                } catch (Exception e) {  
                    e.printStackTrace();  
                    if(msgs.get(0).getReconsumeTimes()==3){  
                        //记录日志  
                          
                        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;// 成功  
                    }else{  
                          
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;// 重试  
                    }  
                }  
  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;// 成功  
            }  
        });  
  
        consumer.start();  
  
        System.out.println("Consumer Started.");  
    }  
}  
```

2.2超时的情况，这种情况MQ会无限制的发送给消费端。  

就是由于网络的情况，MQ发送数据之后，Consumer端并没有收到导致超时。也就是消费端  
没有给我返回return 任何状态，这样的就认为没有到达Consumer端。  
这里模拟Producer只发送一条数据。consumer端暂停1分钟并且不发送接收状态给MQ  

```java
package model;  
  
import java.util.List;  
  
import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;  
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;  
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;  
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;  
import com.alibaba.rocketmq.client.exception.MQClientException;  
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;  
import com.alibaba.rocketmq.common.message.MessageExt;  
  
/** 
 * Consumer，订阅消息 
 */  
public class Consumer {  
  
    public static void main(String[] args) throws InterruptedException, MQClientException {  
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("message_consumer");  
        consumer.setNamesrvAddr("192.168.100.145:9876;192.168.100.146:9876");  
        consumer.setConsumeMessageBatchMaxSize(10);  
        /** 
         * 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费<br> 
         * 如果非第一次启动，那么按照上次消费的位置继续消费 
         */  
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);  
  
        consumer.subscribe("TopicTest", "*");  
  
        consumer.registerMessageListener(new MessageListenerConcurrently() {  
  
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
  
                try {  
  
                    // 表示业务处理时间  
                    System.out.println("=========开始暂停===============");  
                    Thread.sleep(60000);  
  
                    for (MessageExt msg : msgs) {  
                        System.out.println(" Receive New Messages: " + msg);  
                    }  
  
                } catch (Exception e) {  
                    e.printStackTrace();  
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;// 重试  
                }  
  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;// 成功  
            }  
        });  
  
        consumer.start();  
  
        System.out.println("Consumer Started.");  
    }  
}  
```

三、消费模式  

1、集群消费  
2、广播消费  

集群消费  
不同的consumer的 message_consumer 要相同，相当于同组  
rocketMQ默认是集群消费,我们可以通过在Consumer来支持广播消费  
consumer.setMessageModel(MessageModel.BROADCASTING);// 广播消费  

```java
package model;  
  
import java.util.List;  
  
import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;  
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;  
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;  
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;  
import com.alibaba.rocketmq.client.exception.MQClientException;  
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;  
import com.alibaba.rocketmq.common.message.MessageExt;  
import com.alibaba.rocketmq.common.protocol.heartbeat.MessageModel;  
  
/** 
 * Consumer，订阅消息 
 */  
public class Consumer2 {  
  
    public static void main(String[] args) throws InterruptedException, MQClientException {  
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("message_consumer");  
        consumer.setNamesrvAddr("192.168.100.145:9876;192.168.100.146:9876");  
        consumer.setConsumeMessageBatchMaxSize(10);  
        consumer.setMessageModel(MessageModel.BROADCASTING);// 广播消费  
      
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);  
  
        consumer.subscribe("TopicTest", "*");  
  
        consumer.registerMessageListener(new MessageListenerConcurrently() {  
  
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
  
                try {  
  
                    for (MessageExt msg : msgs) {  
                        System.out.println(" Receive New Messages: " + msg);  
                    }  
  
                } catch (Exception e) {  
                    e.printStackTrace();  
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;// 重试  
                }  
  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;// 成功  
            }  
        });  
  
        consumer.start();  
  
        System.out.println("Consumer Started.");  
    }  
}  
```

消费模式  
广播消费：rocketMQ默认是集群消费,我们可以通过在Consumer来支持广播消费  
consumer.setMessageModel(MessageModel.BROADCASTING);// 广播消费  

```java
package model;  
  
import java.util.List;  
  
import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;  
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;  
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;  
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;  
import com.alibaba.rocketmq.client.exception.MQClientException;  
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;  
import com.alibaba.rocketmq.common.message.MessageExt;  
import com.alibaba.rocketmq.common.protocol.heartbeat.MessageModel;  
  
/** 
 * Consumer，订阅消息 
 */  
public class Consumer2 {  
  
    public static void main(String[] args) throws InterruptedException, MQClientException {  
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("message_consumer");  
        consumer.setNamesrvAddr("192.168.100.145:9876;192.168.100.146:9876");  
        consumer.setConsumeMessageBatchMaxSize(10);  
        consumer.setMessageModel(MessageModel.BROADCASTING);// 广播消费  
      
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);  
  
        consumer.subscribe("TopicTest", "*");  
  
        consumer.registerMessageListener(new MessageListenerConcurrently() {  
  
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
  
                try {  
  
                    for (MessageExt msg : msgs) {  
                        System.out.println(" Receive New Messages: " + msg);  
                    }  
  
                } catch (Exception e) {  
                    e.printStackTrace();  
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;// 重试  
                }  
  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;// 成功  
            }  
        });  
  
        consumer.start();  
  
        System.out.println("Consumer Started.");  
    }  
}  
```

四、刷盘方式  

同步刷盘：在消息到达MQ后，RocketMQ需要将数据持久化，同步刷盘是指数据到达内存之后，  
必须刷到commitlog日志之后才算成功，然后返回producer数据已经发送成功。  

异步刷盘：异步刷盘是指数据到达内存之后,返回producer说数据已经发送成功。，然后再写入commitlog日志。  

commitlog：  
commitlog就是来存储所有的元信息，包含消息体，类似于Mysql、Oracle的redolog,所以主要有CommitLog在，  
Consume Queue即使数据丢失，仍然可以恢复出来。  
consumequeue：记录数据的位置,以便Consume快速通过consumequeue找到commitlog中的数据  



- Producer(v0316001)  
Producer的用途大家都很清楚，主要就是生产消息，那么分布式模式下与单队列模式不一样，  
如何能够充分利用分布式的优势，将生产的消息分布到不同的队列下呢？Rocket-MQ提供了3种  
不同模式的Producer：  
1.NormalProducer  
2.OrderProducer  
3.TransactionProducer  

普通模式：不保证消息顺序一致性  

顺序模式：  
topic下有多个队列，队列也可以分别放到不同的Broker下，可以并行的被多个consumer消费，  
但每个队列下所有的订单一定是顺序消费的  

Producer  
要把需要顺序消费的消息放入同一个队列  

```java
import java.io.IOException;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.List;
 
import com.alibaba.rocketmq.client.exception.MQBrokerException;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;
import com.alibaba.rocketmq.client.producer.MessageQueueSelector;
import com.alibaba.rocketmq.client.producer.SendResult;
import com.alibaba.rocketmq.common.message.Message;
import com.alibaba.rocketmq.common.message.MessageQueue;
import com.alibaba.rocketmq.remoting.exception.RemotingException;
 
 
/**
 * Producer，发送顺序消息
 */
public class Producer {
    public static void main(String[] args) throws IOException {
        try {
            DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
 
            producer.setNamesrvAddr("192.168.0.104:9876");
 
            producer.start();
 
            //String[] tags = new String[] { "TagA", "TagC", "TagD" };
 
            Date date = new Date();
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            String dateStr = sdf.format(date);
            for (int i = 0; i < 10; i++) {
                // 加个时间后缀
                String body = dateStr + "order_1" + i;
                Message msg = new Message("TopicTest", "order_1", "KEY" + i, body.getBytes());
 
                SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                    @Override
                    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                        Integer id = (Integer) arg;
                        return mqs.get(id);
                    }
                }, 0);//0是队列的下标
 
                System.out.println(sendResult + ", body:" + body);
            }

            for (int i = 0; i < 10; i++) {
                // 加个时间后缀
                String body = dateStr + "order_2" + i;
                Message msg = new Message("TopicTest", "order_2", "KEY" + i, body.getBytes());
 
                SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                    @Override
                    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                        Integer id = (Integer) arg;
                        return mqs.get(id);
                    }
                }, 1);//0是队列的下标
 
                System.out.println(sendResult + ", body:" + body);
            }

            for (int i = 0; i < 10; i++) {
                // 加个时间后缀
                String body = dateStr + "order_3" + i;
                Message msg = new Message("TopicTest", "order_3", "KEY" + i, body.getBytes());
 
                SendResult sendResult = producer.send(msg, new MessageQueueSelector() {
                    @Override
                    public MessageQueue select(List<MessageQueue> mqs, Message msg, Object arg) {
                        Integer id = (Integer) arg;
                        return mqs.get(id);
                    }
                }, 2);//0是队列的下标
 
                System.out.println(sendResult + ", body:" + body);
            }

             
            producer.shutdown();
        } catch (MQClientException e) {
            e.printStackTrace();
        } 
        System.in.read();
    }
}
```

consumer  
使用的不同于普通模式的MessageListenerConcurrently，变成了 MessageListenerOrderly  
这样就能保证同一个线程，去访问同一个队列，实现顺序消费  

```java
import java.util.List;
import java.util.Random;
import java.util.concurrent.TimeUnit;
 
import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeOrderlyContext;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeOrderlyStatus;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerOrderly;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;
import com.alibaba.rocketmq.common.message.MessageExt;
/**
 * 顺序消息消费，带事务方式（应用可控制Offset什么时候提交）
 */
public class Consumer {
 
    public static void main(String[] args) throws MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("please_rename_unique_group_name_3");
        consumer.setNamesrvAddr("192.168.0.104:9876");
        /**
         * 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费
         * 如果非第一次启动，那么按照上次消费的位置继续消费
         */
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);
 
        consumer.subscribe("TopicTest", "*");
 
        consumer.registerMessageListener(new MessageListenerOrderly() {
 
            Random random = new Random();
 
            @Override
            public ConsumeOrderlyStatus consumeMessage(List<MessageExt> msgs, ConsumeOrderlyContext context) {
                context.setAutoCommit(true);
                System.out.print(Thread.currentThread().getName() + " Receive New Messages: " );
                for (MessageExt msg: msgs) {
                    System.out.println(msg + ", content:" + new String(msg.getBody()));
                }
                try {
                    //模拟业务逻辑处理中...
                    TimeUnit.SECONDS.sleep(random.nextInt(10));
                } catch (Exception e) {
                    e.printStackTrace();
                }
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
 
        consumer.start();
 
        System.out.println("Consumer Started.");
    }
 
}
```


- 事物模式的应用，避免分布式事物  

分布式事物的实现一种可行的方法：两阶段提交协议(2PC)，通信时间长，性能不好  

分布式开放消息系统[简书链接](http://www.jianshu.com/p/453c6e7ff81c)  


RocketMQ第一阶段发送Prepared消息时，会拿到消息的地址，第二阶段执行本地事物，  
第三阶段通过第一阶段拿到的地址去访问消息，并修改消息的状态。  
细心的你可能又发现问题了，如果确认消息发送失败了怎么办？RocketMQ会定期扫描消  
息集群中的事物消息，如果发现了Prepared消息，它会向消息发送端(生产者)确认，Bob  
的钱到底是减了还是没减呢？如果减了是回滚还是继续发送确认消息呢？RocketMQ会根据  
发送端设置的策略来决定是回滚还是继续发送确认消息。这样就保证了消息发送与本地事务  
同时成功或同时失败。  

Producer  
使用sendMessageInTransaction发送事物型消息  
使用LocalTransactionExecuter的子类处理本地事物  

```java
import java.util.concurrent.TimeUnit;

import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.LocalTransactionExecuter;
import com.alibaba.rocketmq.client.producer.LocalTransactionState;
import com.alibaba.rocketmq.client.producer.SendResult;
import com.alibaba.rocketmq.client.producer.TransactionCheckListener;
import com.alibaba.rocketmq.client.producer.TransactionMQProducer;
import com.alibaba.rocketmq.common.message.Message;
import com.alibaba.rocketmq.common.message.MessageExt;

public class Producer {

    public static void main(String[] args) throws MQClientException, InterruptedException {
        /**
         * 一个应用创建一个Producer，由应用来维护此对象，可以设置为全局对象或者单例
         * 注意：ProducerGroupName需要由应用来保证唯一,一类Producer集合的名称，这类Producer通常发送一类消息，
         * 且发送逻辑一致
         * ProducerGroup这个概念发送普通的消息时，作用不大，但是发送分布式事务消息时，比较关键，
         * 因为服务器会回查这个Group下的任意一个Producer
         */
        final TransactionMQProducer producer = new TransactionMQProducer("ProducerGroupName");
        // nameserver服务
        producer.setNamesrvAddr("192.168.1.121:9876";"192.168.1.122:9876");
        producer.setInstanceName("Producer");

        /**
         * Producer对象在使用之前必须要调用start初始化，初始化一次即可
         * 注意：切记不可以在每次发送消息时，都调用start方法
         */
        producer.start();
        // 服务器回调Producer，检查本地事务分支成功还是失败
        producer.setTransactionCheckListener(new TransactionCheckListener() {

            public LocalTransactionState checkLocalTransactionState(
                    MessageExt msg) {
                System.out.println("checkLocalTransactionState --" + new String(msg.getBody()));
                return LocalTransactionState.COMMIT_MESSAGE;
            }
        });



        for (int i = 0; i < 2; i++) {
            try {
                {
                    Message msg = new Message("TopicTransaction", // topic
                            "Transaction"+i,                         // tag
                            "key",                   // key消息关键词，多个Key用KEY_SEPARATOR隔开（查询消息使用）
                            ("Hello Rocket").getBytes());   // body
                    SendResult sendResult = producer.sendMessageInTransaction(msg, tranExecutor ,"tq");
                    System.out.println(sendResult);
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
            TimeUnit.MILLISECONDS.sleep(1000);
        }

        /**
         * 应用退出时，要调用shutdown来清理资源，关闭网络连接，从MetaQ服务器上注销自己
         * 注意：我们建议应用在JBOSS、Tomcat等容器的退出钩子里调用shutdown方法
         */
        // producer.shutdown();
        Runtime.getRuntime().addShutdownHook(new Thread(new Runnable() {
            public void run() {
                producer.shutdown();
            }
        }));
        System.exit(0);
    } // 执行本地事务，由客户端回调

}
```

TransactionExecuter  
失败返回ROLLBACK_MESSAGE  
正常返回COMMIT_MESSAGE  

```java
public class TransactionExecuter implements LocalTransactionExecuter(){

            public LocalTransactionState executeLocalTransactionBranch(Message msg, Object arg) {
                System.out.println("msg=" + new String(msg.getBody()));
                System.out.println("arg=" + arg);
                Sting tag = msg.getTags();
                if(tag.equals(Transaction1)){
                	System.out.println(这里处理业务逻辑失败回滚);
                	return LocalTransactionState.ROLLBACK_MESSAGE;
                }

                return LocalTransactionState.COMMIT_MESSAGE;
            }
        },



```

Consumer  
注册的Listener只需实现正常的MessageListenerConcurrently，和原来一样  
订阅的Topic是 TopicTransaction  
结果就是只能接收Transaction2消息，因为第一条被回滚了  

```java

import java.util.List;
import com.alibaba.rocketmq.client.consumer.DefaultMQPushConsumer;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
import com.alibaba.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
import com.alibaba.rocketmq.client.consumer.listener.MessageListenerConcurrently;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.common.consumer.ConsumeFromWhere;
import com.alibaba.rocketmq.common.message.MessageExt;
/**
 * Consumer，订阅消息
 */
public class Consumer {

    public static void main(String[] args) throws InterruptedException, MQClientException {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("group_name");
        consumer.setNamesrvAddr("192.168.1.121:9876";"192.168.1.122:9876");
        consumer.setConsumeMessageBatchMaxSize(10);  
        /** 
         * 设置Consumer第一次启动是从队列头部开始消费还是队列尾部开始消费<br> 
         * 如果非第一次启动，那么按照上次消费的位置继续消费 
         */  
        consumer.setConsumeFromWhere(ConsumeFromWhere.CONSUME_FROM_FIRST_OFFSET);  
  
        consumer.subscribe("TopicTransaction", "*");  
  
        consumer.registerMessageListener(new MessageListenerConcurrently() {  
  
            public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {  
                  
                try {  
                    System.out.println("msgs的长度" + msgs.size());  
                    System.out.println(Thread.currentThread().getName() + " Receive New Messages: " + msgs);  
                } catch (Exception e) {  
                    e.printStackTrace();  
                    return ConsumeConcurrentlyStatus.RECONSUME_LATER;  
                }  
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;  
            }  
        });  
  
        consumer.start();  
  
        System.out.println("Consumer Started.");  
    }
}

```

- consumer详解(v0321001)  

push方式里，consumer把轮询过程封装了，并注册MessageListener监听器，  
取到消息后，唤醒MessageListener的consumeMessage()来消费，对用户而  
言，感觉消息是被推送过来的。  

pull方式里，取消息的过程需要用户自己写，首先通过打算消费的Topic拿到  
MessageQueue的集合，遍历MessageQueue集合，然后针对每个MessageQueue  
批量取消息，一次取完后，记录该队列下一次要取的开始offset，直到取完了，  
再换另一个MessageQueue。  

Producer，发送消息  

```java
package com.zoo.quickstart;
 
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.client.producer.DefaultMQProducer;
import com.alibaba.rocketmq.client.producer.SendResult;
import com.alibaba.rocketmq.common.message.Message;
 
 
/**
 * Producer，发送消息
 * 
 */
public class Producer {
    public static void main(String[] args) throws MQClientException, InterruptedException {
        DefaultMQProducer producer = new DefaultMQProducer("please_rename_unique_group_name");
        producer.setNamesrvAddr("192.168.0.104:9876");
        producer.start();
 
        for (int i = 0; i < 5; i++) {
            try {
                Message msg = new Message("TopicTest",// topic
                    "TagA",// tag
                    ("Hello RocketMQ " + i).getBytes()// body
                        );
                SendResult sendResult = producer.send(msg);
                System.out.println(sendResult);
                Thread.sleep(6000);
            }
            catch (Exception e) {
                e.printStackTrace();
                Thread.sleep(3000);
            }
        }
 
        producer.shutdown();
    }
}
```

PullConsumer  
拉取式消费，但是可以重复消费  

```java
package com.zoo.quickstart.pull;
 
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
 
import com.alibaba.rocketmq.client.consumer.DefaultMQPullConsumer;
import com.alibaba.rocketmq.client.consumer.PullResult;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.common.message.MessageQueue;
 
 
/**
 * PullConsumer，订阅消息
 */
public class PullConsumer {
    private static final Map<MessageQueue, Long> offseTable = new HashMap<MessageQueue, Long>();
 
 
    public static void main(String[] args) throws MQClientException {
        DefaultMQPullConsumer consumer = new DefaultMQPullConsumer("please_rename_unique_group_name_5");
        consumer.setNamesrvAddr("192.168.0.104:9876");
        consumer.start();
 
        Set<MessageQueue> mqs = consumer.fetchSubscribeMessageQueues("TopicTest");
        for (MessageQueue mq : mqs) {
            System.out.println("Consume from the queue: " + mq);
            SINGLE_MQ: while (true) {
                try {
                    PullResult pullResult =
                            consumer.pullBlockIfNotFound(mq, null, getMessageQueueOffset(mq), 32);
                    System.out.println(pullResult);
                    putMessageQueueOffset(mq, pullResult.getNextBeginOffset());
                    switch (pullResult.getPullStatus()) {
                    case FOUND:
                        // TODO
                        break;
                    case NO_MATCHED_MSG:
                        break;
                    case NO_NEW_MSG:
                        break SINGLE_MQ;
                    case OFFSET_ILLEGAL:
                        break;
                    default:
                        break;
                    }
                }
                catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
 
        consumer.shutdown();
    }
 
 
    private static void putMessageQueueOffset(MessageQueue mq, long offset) {
        offseTable.put(mq, offset);
    }
 
 
    private static long getMessageQueueOffset(MessageQueue mq) {
        Long offset = offseTable.get(mq);
        if (offset != null)
            return offset;
 
        return 0;
    }
 
}
```

PullScheduleService多用这种方式  
实现定时拉取  

```java
package com.zoo.quickstart.pull;
 
import com.alibaba.rocketmq.client.consumer.MQPullConsumer;
import com.alibaba.rocketmq.client.consumer.MQPullConsumerScheduleService;
import com.alibaba.rocketmq.client.consumer.PullResult;
import com.alibaba.rocketmq.client.consumer.PullTaskCallback;
import com.alibaba.rocketmq.client.consumer.PullTaskContext;
import com.alibaba.rocketmq.client.exception.MQClientException;
import com.alibaba.rocketmq.common.message.MessageQueue;
import com.alibaba.rocketmq.common.protocol.heartbeat.MessageModel;
 
 
public class PullScheduleService {
 
    public static void main(String[] args) throws MQClientException {
        final MQPullConsumerScheduleService scheduleService = new MQPullConsumerScheduleService("GroupName1");
        scheduleService.getDefaultMQPullConsumer().setNamesrvAddr("192.168.0.104:9876");
        scheduleService.setMessageModel(MessageModel.CLUSTERING);
        scheduleService.registerPullTaskCallback("TopicTest", new PullTaskCallback() {
 
            @Override
            public void doPullTask(MessageQueue mq, PullTaskContext context) {
                MQPullConsumer consumer = context.getPullConsumer();
                try {
                    // 获取从哪里拉取
                    long offset = consumer.fetchConsumeOffset(mq, false);
                    if (offset < 0)
                        offset = 0;
 
                    PullResult pullResult = consumer.pull(mq, "*", offset, 32);
                    System.out.println(offset + "\t" + mq + "\t" + pullResult);
                    switch (pullResult.getPullStatus()) {
                    case FOUND:
                        break;
                    case NO_MATCHED_MSG:
                        break;
                    case NO_NEW_MSG:
                    case OFFSET_ILLEGAL:
                        break;
                    default:
                        break;
                    }
 
                    // 存储Offset，客户端每隔5s会定时刷新到Broker
                    consumer.updateConsumeOffset(mq, pullResult.getNextBeginOffset());
 
                    // 设置再过100ms后重新拉取
                    context.setPullNextDelayTimeMillis(100);
                }
                catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
 
        scheduleService.start();
    }
}
```


# dubbo  

- 启动时检查  
Dubbo缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止Spring初始化完成，  
以便上线时，能及早发现问题，默认check=true。  

如果你的Spring容器是懒加载的，或者通过API编程延迟引用服务，请关闭check，否则服务临时  
不可用时，会抛出异常，拿到null引用，如果check=false，总是会返回引用，当服务恢复时，  
能自动连上。


- 集群容错模式(一般常用的就是retries="0"，重试设为0，相当于快速失败)  
可以自行扩展集群容错策略，参见：集群扩展  

Failover Cluster  
失败自动切换，当出现失败，重试其它服务器。(缺省)  
通常用于读操作，但重试会带来更长延迟。  
可通过retries="2"来设置重试次数(不含第一次)。  
Failfast Cluster  
快速失败，只发起一次调用，失败立即报错。  
通常用于非幂等性的写操作，比如新增记录。  
Failsafe Cluster  
失败安全，出现异常时，直接忽略。  
通常用于写入审计日志等操作。  
Failback Cluster  
失败自动恢复，后台记录失败请求，定时重发。  
通常用于消息通知操作。  
Forking Cluster  
并行调用多个服务器，只要一个成功即返回。  
通常用于实时性要求较高的读操作，但需要浪费更多服务资源。  
可通过forks="2"来设置最大并行数。  
Broadcast Cluster  
广播调用所有提供者，逐个调用，任意一台报错则报错。(2.1.0开始支持)  
通常用于通知所有提供者更新缓存或日志等本地资源信息。  

- 负载均衡  
配置LoadBalance  

集群负载均衡时，Dubbo提供了多种均衡策略，缺省为random随机调用。  
Random LoadBalance  
随机，按权重设置随机概率。  
在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。  
RoundRobin LoadBalance  
轮循，按公约后的权重设置轮循比率。  
存在慢的提供者累积请求问题，比如：第二台机器很慢，但没挂，当请求调到第二台时就卡在那，久而久之，所有请求都卡在调到第二台上。  
LeastActive LoadBalance  
最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。  
使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。  
ConsistentHash LoadBalance  
一致性Hash，相同参数的请求总是发到同一提供者。  
比较少用，详情见网页  

- 线程模型  
Dispatcher，一般默认就行了  
all 所有消息都派发到线程池，包括请求，响应，连接事件，断开事件，心跳等。  
direct 所有消息都不派发到线程池，全部在IO线程上直接执行。  
message 只有请求响应消息派发到线程池，其它连接断开事件，心跳等消息，直接在IO线程上执行。  
execution 只请求消息派发到线程池，不含响应，响应和其它连接断开事件，心跳等消息，直接在IO线程上执行。  
connection 在IO线程上，将连接断开事件放入队列，有序逐个执行，其它消息派发到线程池。  
ThreadPool  
fixed 固定大小线程池，启动时建立线程，不关闭，一直持有。(缺省)  
cached 缓存线程池，空闲一分钟自动删除，需要时重建。  
limited 可伸缩线程池，但池中的线程数只会增长不会收缩。(为避免收缩时突然来了大流量引起的性能问题)。  
配置如：  

```xml
<dubbo:protocol name="dubbo" dispatcher="all" threadpool="fixed" threads="100" />  
```

只订阅  
问题  
为方便开发测试，经常会在线下共用一个所有服务可用的注册中心，这时，  
如果一个正在开发中的服务提供者注册，可能会影响消费者不能正常运行。  
解决方案  
可以让服务提供者开发方，只订阅服务(开发的服务可能依赖其它服务)，  
而不注册正在开发的服务，通过直连测试正在开发的服务。  


只注册  
问题  
如果有两个镜像环境，两个注册中心，有一个服务只在其中一个注册中心有部署，  
另一个注册中心还没来得及部署，而两个注册中心的其它应用都需要依赖此服务，  
所以需要将服务同时注册到两个注册中心，但却不能让此服务同时依赖两个注册  
中心的其它服务。  
解决方案  
可以让服务提供者方，只注册服务到另一注册中心，而不从另一注册中心订阅服务。  

静态服务  
有时候希望人工管理服务提供者的上线和下线，此时需将注册中心标识为非动态管理模式。  
<dubbo:registry address="10.20.141.150:9090" dynamic="false" />  

多协议、多注册见网站  

服务分组
<dubbo:service group="feedback" interface="com.xxx.IndexService" />  
<dubbo:service group="member" interface="com.xxx.IndexService" />  

多版本
在低压力时间段，先升级一半提供者为新版本  
再将所有消费者升级为新版本  
然后将剩下的一半提供者升级为新版本  
<dubbo:service interface="com.foo.BarService" version="1.0.0" />  
<dubbo:service interface="com.foo.BarService" version="2.0.0" />  


- dubbox  
支持REST风格远程调用（HTTP + JSON/XML)：基于非常成熟的JBoss RestEasy框架，  
在dubbo中实现了REST风格（HTTP + JSON/XML）的远程调用，以显著简化企业内部的  
跨语言交互，同时显著简化企业对外的Open API、无线API甚至AJAX服务端等等的开发。  
事实上，这个REST调用也使得Dubbo可以对当今特别流行的“微服务”架构提供基础性支持。   
另外，REST调用也达到了比较高的性能，在基准测试下，HTTP + JSON与Dubbo 2.x默认  
的RPC协议（即TCP + Hessian2二进制序列化）之间只有1.5倍左右的差距，详见文档中  
的基准测试报告。  

支持基于Kryo和FST的Java高效序列化实现：基于当今比较知名的Kryo和FST高性能序列化库，  
为Dubbo默认的RPC协议添加新的序列化实现，并优化调整了其序列化体系，比较显著的提高了  
Dubbo RPC的性能，详见文档中的基准测试报告。  

支持基于Jackson的JSON序列化：基于业界应用最广泛的Jackson序列化库，为Dubbo默认的  
RPC协议添加新的JSON序列化实现。

支持基于嵌入式Tomcat的HTTP remoting体系：基于嵌入式tomcat实现dubbo的HTTP remoting  
体系（即dubbo-remoting-http），用以逐步取代Dubbo中旧版本的嵌入式Jetty，可以显著的  
提高REST等的远程调用性能，并将Servlet API的支持从2.5升级到3.1。（注：除了REST，dubbo  
中的WebServices、Hessian、HTTP Invoker等协议都基于这个HTTP remoting体系）。  

升级Spring：将dubbo中Spring由2.x升级到目前最常用的3.x版本，减少版本冲突带来的麻烦。  
升级ZooKeeper客户端：将dubbo中的zookeeper客户端升级到最新的版本，以修正老版本中包含的bug。  
支持完全基于Java代码的Dubbo配置：基于Spring的Java Config，实现完全无XML的纯Java代码方式来配置dubbo  

调整Demo应用：暂时将dubbo的demo应用调整并改写以主要演示REST功能、Dubbo协议的新序列化方式、  
基于Java代码的Spring配置等等。  


dubbox producer端  
/dubbox-provider/src/dubbo-provider.xml  
配置文件中开启注解  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd">
	
	
	<!-- 引入zc-com通用类属性注入配置文件 -->
	<util:properties id="zcparams" location="classpath:params.properties"></util:properties>
	
    <dubbo:application name="provider" owner="programmer" organization="dubbox"/>

	<!-- zookeeper注册中心 -->
    <dubbo:registry address="zookeeper://192.168.1.121:2181?backup=192.168.1.122:2181,192.168.1.123:2181"/>

    <dubbo:annotation package="bhz.service" />
    
    <!-- kryo实现序列化  serialization="kryo" optimizer="bhz.utils.SerializationOptimizerImpl"-->
    <dubbo:protocol name="dubbo"  />


	
</beans>
```

/dubbox-provider/test/bhz/test/Provider.java  

```java
package bhz.test;


import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Provider {

	public static void main(String[] args) throws Exception {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(
				new String[] { "dubbo-provider.xml" });
		context.start();
		System.in.read(); // 为保证服务一直开着，利用输入流的阻塞来模拟
	}
}
```

配置spring和dubbo的service  
配置协议为dubbo，重试次数为0  

```java
package bhz.service.impl;

import java.util.HashMap;
import java.util.Map;

import org.springframework.stereotype.Service;

import bhz.entity.Simple;
import bhz.service.SimpleService;

@Service("simpleService")
@com.alibaba.dubbo.config.annotation.Service(interfaceClass=bhz.service.SimpleService.class, protocol={"dubbo"}, retries=0)
public class SimpleServiceImpl implements SimpleService{

	@Override
	public String sayHello(String name) {
		return "hello" + name;
	}

	@Override
	public Simple getSimple() {
        Map<String,Integer> map = new HashMap<String, Integer>(2);  
        map.put("zhang0", 1);  
        map.put("zhang1", 2);  
        return new Simple("zhang3", 21, map);
	}

}

```


consumer  
consuer中有Simple类实现序列化接口，也有SimpleService接口但没有实现类  

/dubbox-consumer/test/bhz/test/Consumer.java  

```java
package bhz.test;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import bhz.entity.Simple;
import bhz.service.SimpleService;
import bhz.service.UserService;

public class Consumer {

	public static void main(String[] args) throws Exception {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(
				new String[] { "dubbo-consumer.xml" });
		context.start();

		SimpleService ss = (SimpleService) context.getBean("simpleService");
		String hello = ss.sayHello("tom");
		System.out.println(hello);
		Simple simple = ss.getSimple();
		System.out.println(simple.getName());
		System.out.println(simple.getAge());
		System.out.println(simple.getMap().get("zhang1"));
			
	}

}
```

/dubbox-consumer/src/dubbo-consumer.xml  
序列化方式不用kryo，用序列化接口  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
	http://code.alibabatech.com/schema/dubbo http://code.alibabatech.com/schema/dubbo/dubbo.xsd
	http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd">
	
	<!-- 引入zc-com通用类属性注入配置文件 -->
	<util:properties id="zcparams" location="classpath:params.properties"></util:properties>

    <dubbo:application name="consumer" owner="programmer" organization="dubbox"/>
    
    <dubbo:registry address="zookeeper://192.168.1.121:2181?backup=192.168.1.122:2181,192.168.1.123:2181"/>
    <!-- kryo实现序列化   serialization="kryo" optimizer="bhz.utils.SerializationOptimizerImpl"-->
    <dubbo:protocol name="dubbo" />
    
    <!-- 生成远程服务代理，可以像使用本地bean -->
	<dubbo:reference interface="bhz.service.UserService" id="userService" check="false" />
    <!-- 生成远程服务代理，可以像使用本地bean -->
	<dubbo:reference interface="bhz.service.SimpleService" id="simpleService" />
	
</beans>
```

REST方式  
/dubbox-provider/src/bhz/service/UserService.java  

```java
package bhz.service;

import javax.ws.rs.Consumes;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import com.alibaba.dubbo.rpc.protocol.rest.support.ContentType;

import bhz.entity.User;
@Path("/userService")
@Consumes({MediaType.APPLICATION_JSON, MediaType.TEXT_XML})
@Produces({ContentType.APPLICATION_JSON_UTF_8, ContentType.TEXT_XML_UTF_8})
public interface UserService {
	@GET
    @Path("/testget")
	public void testget();
    @GET
    @Path("/getUser")
	public User getUser();
	@GET
	@Path("/get/{id : \\d+}")
	public User getUser(@PathParam(value = "id") Integer id);
	@GET
	@Path("/get/{id : \\d+}/{name : [a-zA-Z][0-9]}")
	public User getUser(@PathParam(value = "id") Integer id, @PathParam(value = "name") String name);
    @POST
    @Path("/testpost")
	public void testpost();
    @POST
    @Path("/postUser")
	public User postUser(User user);
	@POST
	@Path("/post/{id}")
	public User postUser(@PathParam(value = "id") String id);

}

```

/dubbox-provider/src/bhz/service/impl/UserServiceImpl.java  

```java
package bhz.service.impl;

import javax.ws.rs.Consumes;
import javax.ws.rs.DELETE;
import javax.ws.rs.GET;
import javax.ws.rs.POST;
import javax.ws.rs.PUT;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;
import org.springframework.stereotype.Service;
import bhz.entity.User;
import bhz.service.UserService;
import com.alibaba.dubbo.rpc.protocol.rest.support.ContentType;

//这里是spring的注解
@Service("userService")
//这个是dubbo的注解（同时提供dubbo本地，和rest方式）
@com.alibaba.dubbo.config.annotation.Service(interfaceClass=bhz.service.UserService.class, protocol = {"rest", "dubbo"}, retries=0)
public class UserServiceImpl implements UserService{

	public void testget() {
		//http://localhost:8888/provider/userService/testget
		System.out.println("测试...get");
	}

	public User getUser() {
    	User user = new User();
    	user.setId("1001");
    	user.setName("张三");
    	return user;
	}
	public User getUser(Integer id) {
		System.out.println("测试传入int类型的id: " + id);
    	User user = new User();
    	user.setId("1001");
    	user.setName("张三");
    	return user;
	}
	public User getUser(Integer id, String name) {

		System.out.println("测试俩个参数：");
		System.out.println("id: " + id);
		System.out.println("name: " + name);
    	User user = new User();
    	user.setId("1001");
    	user.setName("张三");
    	return user;
	}
	

	public void testpost() {
    	System.out.println("测试...post");
	}
    

	public User postUser(User user) {
    	System.out.println(user.getName());
    	System.out.println("测试...postUser");
    	User user1 = new User();
    	user1.setId("1001");
    	user1.setName("张三");
    	return user1;
	}


	public User postUser(String id) {
		System.out.println(id);
		System.out.println("测试...get");
    	User user = new User();
    	user.setId("1001");
    	user.setName("张三");
    	return user;
	}
}
```

- 序列化

在dubbo RPC中，同时支持多种序列化方式，例如：  
dubbo序列化：阿里尚未开发成熟的高效java序列化实现，阿里不建议在生产环境使用它  
hessian2序列化：hessian是一种跨语言的高效二进制序列化方式。但这里实际不是原生  
的hessian2序列化，而是阿里修改过的hessian lite，它是dubbo RPC默认启用的序列化方式  
json序列化：目前有两种实现，一种是采用的阿里的fastjson库，另一种是采用dubbo  
中自己实现的简单json库，但其实现都不是特别成熟，而且json这种文本序列化性能一般  
不如上面两种二进制序列化。  
java序列化：主要是采用JDK自带的Java序列化实现，性能很不理想。  
在通常情况下，这四种主要序列化方式的性能从上到下依次递减。对于dubbo RPC这种追求  
高性能的远程调用方式来说，实际上只有1、2两种高效序列化方式比较般配，而第1个dubbo  
序列化由于还不成熟，所以实际只剩下2可用，所以dubbo RPC默认采用hessian2序列化。  


最近几年，各种新的高效序列化方式层出不穷，不断刷新序列化性能的上限，最典型的包括：  
专门针对Java语言的：Kryo，FST等等  
跨语言的：Protostuff，ProtoBuf，Thrift，Avro，MsgPack等等  
这些序列化方式的性能多数都显著优于hessian2（甚至包括尚未成熟的dubbo序列化）。  

有鉴于此，我们为dubbo引入Kryo和FST这两种高效Java序列化实现，来逐步取代hessian2。  
其中，Kryo是一种非常成熟的序列化实现，已经在Twitter、Groupon、Yahoo以及多个著名开源项目  
（如Hive、Storm）中广泛的使用。而FST是一种较新的序列化实现，目前还缺乏足够多的成熟使用案例，  
但我认为它还是非常有前途的。在面向生产环境的应用中，我建议目前更优先选择Kryo。  

启用Kryo和FST  
使用Kryo和FST非常简单，只需要在dubbo RPC的XML配置中添加一个属性即可：  

<dubbo:protocol name="dubbo" serialization="kryo"/>  
<dubbo:protocol name="dubbo" serialization="fst"/>  