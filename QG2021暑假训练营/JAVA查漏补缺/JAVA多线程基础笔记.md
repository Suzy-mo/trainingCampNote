# JAVA多线程基础笔记

## 一、多线程概述

线程与进程

* 运行中的应用就是进程，轮流占有cpu的资源 由于切换过快感觉是同时进行的
* 并发：同一时刻多个在执行

![](E:\QG移动组\QG2021暑假训练营\JAVA查漏补缺\图片\进程与线程.jpg)

* 每个进程中包含多个线程 每个任务对应一个线程--多线程
* 多线程具有随机性 谁先抢到资源就先执行，执行任务的完成度也具有随机性



## 二、线程的创建

### 法一：继承Thread类

```java
public class ThreadOne {
    //main方法相当于主线程
    public static void main(String[] args) {
        //创建对象
        Thread thread1 = new MyThread();
        //多态 也可以直接是MyThread thread1 = new MyThread();
        //启动子线程
        thread1.start();
    }

}
//新建Thread类
class MyThread extends Thread{
    //重写run方法
    @Override
    public void run() {
        //子线程执行的具体任务
        super.run();
        for(int i = 0 ;i < 100 ;i++){
            System.out.println(i);
        }
    }
}
```

* 步骤

  * 新建类继承于Thread
  * 重写run方法
  * 创建对象
  * 启动子线程

* 优缺点

  * 优点 代码简单
  * 缺点 单继承的局限

* 注意事项

  * 如果线程直接调用run方法，相当于变成了普通类
  * start底层是给cpu注册当前线程，并触发run方法执行
  * 建议先创建子线程，主线程的任务放在后面

* Thread常用API

  * sleep(long time)    耗时操作要放在子线程里面

    ```java
    //新建Thread类
    class MyThread extends Thread{
        //重写run方法
        @Override
        public void run() {
            //子线程执行的具体任务
            super.run();
            for(int i = 0 ;i < 100 ;i++){
                try {
                    sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(i);
            }
        }
    }
    ```

    

  * setName（String name）设置名字
  * getName 主线程默认main，子线程 Thread-索引
  * currentThraed 获取当前线程

  ```java
  public class ThreadOne {
      public static void main(String[] args) {
          //创建对象
          Thread thread1 = new MyThread();
          //多态 也可以直接是MyThread thread1 = new MyThread();
          //启动子线程
          thread1.start();
  
          //代码在哪个线程中就获取哪个线程
          Thread m = Thread.currentThread();
          for(int i = 0 ;i < 100 ;i++){
              System.out.println(m.getName()+"->"+i);
          }
      }
  }
  ```

  * 通过有参构造器设置线程名字

    ```java
    public class ThreadTwo {
        public static void main(String[] args) {
            //创建对象
            Thread thread1 = new MyThread2("子线程1");
            thread1.start();
    
            //代码在哪个线程中就获取哪个线程
            Thread m = Thread.currentThread();
            for(int i = 0 ;i < 100 ;i++){
                System.out.println(m.getName()+"->"+i);
            }
        }
    }
    class MyThread2 extends Thread{
        public MyThread2(String name){
            //调用父类的有参构造为当前子线程设置名字
            super(name);
        }
        @Override
        public void run() {
            super.run();
        }
    }
    ```

###  法二：实现Runnable接口

```java
public class ThreadTwo {
    public static void main(String[] args) {
        //3.创建任务类对象
        Runnable target = new MyRunnable();
        //4.把线程任务对象包装成线程对象
        //Thread thread = new Thread(target);
        Thread thread = new Thread("子线程1");
        //5.启动线程
        thread.start();
        
    }
}
//1.创建线程任务类实现Runnable接口
class MyRunnable implements Runnable{
    //2.重写run方法
    @Override
    public void run() {
        //子线程任务
    }
}
```

* 步骤

  * 创建任务类
  * 重写run方法
  * 创建任务类对象
  * 把任务类对象包装为线程对象（可以设置名字）
  * 启动线程

* Thread类的构造器

  * public Thread（）
  * public  Thread（String name）
  * public  Thread（Runnable target）
  * public  Thread（Runnable target，String name）

* 优缺点

  * 缺点：代码复杂了点
  * 优点：打破了单继承的局限性、同一个任务对象可以包装成多个Thread对象（多个线程可共享一个资源）、实现解耦（线程任务代码和线程独立）、线程池中可放入实现Runnable线程任务对象

* 简化写法：匿名内部类写法（常用）

  ```java
  new Thread(new Runnable() {
              @Override
              public void run() {
                  //子线程任务
              }
          }).start();
  ```

* lambda简化

  ```java
  new Thread(()->{
              //子线程任务
          }).start();
  ```

### 法三：Callable实现

可以实现子线程执行返回结果的获取

```java
public class ThreadThree {
    public static void main(String[] args) {
        //3.创建对象
        Callable callable = new MyCallable();
        //4.把callable对象封装为FutureTask对象
        FutureTask<String> task = new FutureTask<>(callable);
        //5.把FutureTask对象包装为线程对象
        Thread thread = new Thread(task);
        //6.启动线程
        thread.start();
        
        //主线程操作

        //在最后去获取结果，如果没有结果会让出cpu
        try {
            String string = task.get();
            System.out.println(string);
        } catch (ExecutionException e) {
            e.printStackTrace();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
}

//1.创建任务类
class MyCallable implements Callable<String>{

    //2.重写call对象
    @Override
    public String call() throws Exception {
        //具体需求
        return null;
    }
}
```



* 步骤
  * 创建callable任务类
  * 重写call方法
  * 创建callable对象
  * 创建fetureTask对象包装callable对象
  * 创建线程对象包装ferureTask对象
  * 启动线程

* FeatureTask
  * 实际上是runnable对象
  * 这样可以被封装为线程对象
  * 可以在线程执行完毕后得到线程执行的结果
  * 巧用泛型 作为线程任务与线程之间的桥梁
* 优缺点
  * 与法二大同小异
  * 多出的优点 能直接得到子线程执行的结果

## 三、线程安全与同步

### 线程安全

多个线程同时共享资源的时候可能会出现线程安全问题

### 线程同步

* 作用  解决线程安全问题
* 做法  把共享资源上锁，每次只能由一个线程进入访问完毕后其他线程才能进来
* 方式  同步代码块 同步方法 lock显示锁

#### 同步代码块

在需要锁的地方把代码块放进去

```java
synchronized ("锁对象"){
            //访问共享资源的核心代码
        }
```

* 锁对象：“唯一”的对象
* 锁对象建议使用共享资源
  * 在实例方法中用this
  * 在静态方法中用 类名.class(this.class)

#### 同步方法

直接在方法的修饰符处加上synchronized，简单快捷，但性能比同步代码块差

```java
public synchronized void methordName(){
   
}
```

#### lock显示锁

在同步方法和代码块基础上更强大

* lock() 加同步锁
* unlock() 释放同步锁

```java
//创建成员变量（final可加可不加）
private Lock lock = new ReentrantLock();

lock.lock();
//被锁的内容 try catch
finally{
    lock.unlock();
}
```

### 总结

线程安全，性能差；线程不安全性能好。开发中不存在安全问题的话可以使用线程不安全的。

