---
title: Thread 子类 vs Runnable 接口——创建线程的两种方式对比
date: 2026-06-25
tags:
  - Java
  - 多线程
  - Thread
  - Runnable
  - 并发编程
categories:
  - 学习笔记
---

## 一、概述

在 Java 中，创建线程主要有两种方式：

1. **继承 Thread 类**：创建 Thread 的子类，重写 `run()` 方法
2. **实现 Runnable 接口**：实现 Runnable 接口的 `run()` 方法，然后传递给 Thread

这两种方式各有优缺点，理解它们的区别对于编写高质量的多线程代码至关重要。

## 二、两种方式的实现

### 方式1：继承 Thread 类

```java
public class MyThread extends Thread {
    private String taskName;
    
    public MyThread(String taskName) {
        this.taskName = taskName;
    }
    
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(taskName + " 执行: " + i);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args) {
        // 创建线程实例并启动
        MyThread thread1 = new MyThread("线程A");
        MyThread thread2 = new MyThread("线程B");
        
        thread1.start();
        thread2.start();
    }
}
```

### 方式2：实现 Runnable 接口

```java
public class MyRunnable implements Runnable {
    private String taskName;
    
    public MyRunnable(String taskName) {
        this.taskName = taskName;
    }
    
    @Override
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(taskName + " 执行: " + i);
            try {
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    
    public static void main(String[] args) {
        // 创建 Runnable 实例
        MyRunnable runnable1 = new MyRunnable("线程A");
        MyRunnable runnable2 = new MyRunnable("线程B");
        
        // 包装成 Thread 对象并启动
        Thread thread1 = new Thread(runnable1);
        Thread thread2 = new Thread(runnable2);
        
        thread1.start();
        thread2.start();
    }
}
```

### 方式3：使用匿名内部类（Runnable）

```java
public class AnonymousThread {
    public static void main(String[] args) {
        // 使用匿名 Runnable
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("匿名线程执行");
            }
        }).start();
        
        // Java 8+ Lambda 表达式
        new Thread(() -> System.out.println("Lambda 线程执行")).start();
    }
}
```

## 三、核心区别对比

### 3.1 继承 vs 组合

| 特性 | 继承 Thread | 实现 Runnable |
|------|------------|--------------|
| 设计方式 | 继承（is-a） | 组合（has-a） |
| 耦合度 | 高 | 低 |
| 灵活性 | 低 | 高 |
| 代码复用 | 差 | 好 |

**继承的问题**：

```java
// 问题：Java 不支持多继承，如果类已经继承了其他类，就无法继承 Thread
public class MyService extends BaseService {
    // 无法再继承 Thread！
}
```

**组合的优势**：

```java
// 优势：可以同时继承其他类并实现 Runnable
public class MyService extends BaseService implements Runnable {
    @Override
    public void run() {
        // 线程执行逻辑
    }
}
```

### 3.2 资源共享

```java
// 使用 Thread 子类——每个线程有独立的资源
class CounterThread extends Thread {
    private int count = 0;  // 每个线程独立
    
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            count++;
        }
        System.out.println("Thread 计数: " + count);
    }
}

// 使用 Runnable——多个线程共享资源
class CounterRunnable implements Runnable {
    private int count = 0;  // 所有线程共享
    
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            count++;
        }
        System.out.println("Runnable 计数: " + count);
    }
}

public class ResourceSharingDemo {
    public static void main(String[] args) throws InterruptedException {
        // Thread 子类：每个线程独立计数
        CounterThread t1 = new CounterThread();
        CounterThread t2 = new CounterThread();
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        // 输出：Thread 计数: 1000, Thread 计数: 1000
        
        // Runnable：共享计数
        CounterRunnable r = new CounterRunnable();
        Thread t3 = new Thread(r);
        Thread t4 = new Thread(r);
        t3.start();
        t4.start();
        t3.join();
        t4.join();
        // 输出：Runnable 计数: 2000（可能不是精确值，需要同步）
    }
}
```

### 3.3 线程池兼容性

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class ThreadPoolDemo {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        // Runnable 可以直接提交给线程池
        executor.submit(new MyRunnable("任务A"));
        executor.submit(new MyRunnable("任务B"));
        
        // Thread 子类也可以提交（因为 Thread 实现了 Runnable）
        executor.submit(new MyThread("任务C"));
        
        executor.shutdown();
    }
}
```

### 3.4 完整对比表

| 特性 | 继承 Thread | 实现 Runnable |
|------|------------|--------------|
| **继承限制** | ❌ 不支持多继承 | ✅ 无继承限制 |
| **资源共享** | ❌ 困难 | ✅ 容易 |
| **代码复用** | ❌ 差 | ✅ 好 |
| **耦合度** | ❌ 高 | ✅ 低 |
| **线程池** | ✅ 支持（间接） | ✅ 原生支持 |
| **Lambda 兼容** | ❌ 不支持 | ✅ 支持 |
| **可扩展性** | ❌ 差 | ✅ 好 |
| **推荐度** | ⚠️ 不推荐 | ✅ 强烈推荐 |

## 四、为什么推荐使用 Runnable

### 4.1 符合面向对象设计原则

1. **单一职责原则**：Runnable 只负责定义任务，不负责线程管理
2. **开闭原则**：任务逻辑可以独立扩展，不影响线程管理
3. **依赖倒置原则**：高层模块依赖抽象（Runnable），而非具体实现

### 4.2 实际项目示例

```java
// 实际项目中的典型用法
public class OrderService {
    private ExecutorService executor = Executors.newFixedThreadPool(10);
    
    public void processOrder(Order order) {
        // 将任务封装为 Runnable
        Runnable task = () -> {
            validateOrder(order);
            deductInventory(order);
            sendNotification(order);
            updateOrderStatus(order);
        };
        
        // 提交到线程池执行
        executor.submit(task);
    }
    
    private void validateOrder(Order order) { /* ... */ }
    private void deductInventory(Order order) { /* ... */ }
    private void sendNotification(Order order) { /* ... */ }
    private void updateOrderStatus(Order order) { /* ... */ }
}
```

### 4.3 Spring 中的应用

```java
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

@Service
public class EmailService {
    
    @Async
    public void sendEmailAsync(String to, String content) {
        // Spring 会自动将此方法包装为 Runnable 提交到线程池
        System.out.println("发送邮件给: " + to);
    }
}
```

## 五、特殊场景：何时使用 Thread 子类

虽然 Runnable 更推荐，但在某些特殊场景下使用 Thread 子类更合适：

### 场景1：需要重写 Thread 的其他方法

```java
public class CustomThread extends Thread {
    @Override
    public void start() {
        System.out.println("线程开始前的准备工作");
        super.start();
    }
    
    @Override
    public void run() {
        System.out.println("线程执行");
    }
    
    @Override
    public void interrupt() {
        System.out.println("线程被中断");
        super.interrupt();
    }
}
```

### 场景2：自定义线程生命周期管理

```java
public class ManagedThread extends Thread {
    private boolean running = true;
    
    @Override
    public void run() {
        while (running) {
            // 执行任务
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                running = false;
            }
        }
        System.out.println("线程优雅退出");
    }
    
    public void stopGracefully() {
        running = false;
        this.interrupt();
    }
}
```

### 场景3：简单的一次性线程

```java
// 简单场景下可以直接使用匿名 Thread
new Thread() {
    @Override
    public void run() {
        System.out.println("简单任务");
    }
}.start();

// 但更推荐使用 Runnable
new Thread(() -> System.out.println("简单任务")).start();
```

## 六、线程状态与生命周期

```java
public class ThreadStateDemo {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        
        // NEW：线程已创建但未启动
        System.out.println("状态1: " + thread.getState());  // NEW
        
        thread.start();
        
        // RUNNABLE：线程正在运行或准备运行
        System.out.println("状态2: " + thread.getState());  // RUNNABLE
        
        try {
            Thread.sleep(500);
            // TIMED_WAITING：线程在 sleep 中
            System.out.println("状态3: " + thread.getState());  // TIMED_WAITING
            
            thread.join();
            // TERMINATED：线程已终止
            System.out.println("状态4: " + thread.getState());  // TERMINATED
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

## 七、常见问题与解决方案

### 7.1 错误：调用 run() 而非 start()

```java
// 错误：直接调用 run() 不会启动新线程
MyRunnable r = new MyRunnable();
r.run();  // 在主线程中执行！

// 正确：调用 start() 启动新线程
new Thread(r).start();  // 在新线程中执行！
```

### 7.2 错误：多次调用 start()

```java
Thread thread = new Thread(() -> System.out.println("任务"));
thread.start();
thread.start();  // 抛出 IllegalThreadStateException！
```

### 7.3 线程安全问题

```java
// 问题：多个线程共享资源时的并发问题
class UnsafeCounter implements Runnable {
    private int count = 0;
    
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            count++;  // 非原子操作，存在竞态条件
        }
    }
    
    public int getCount() { return count; }
}

// 解决方案：使用 synchronized 或 AtomicInteger
class SafeCounter implements Runnable {
    private AtomicInteger count = new AtomicInteger(0);
    
    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            count.incrementAndGet();  // 原子操作
        }
    }
    
    public int getCount() { return count.get(); }
}
```

## 八、最佳实践

### 8.1 推荐使用 Runnable + Lambda

```java
// 推荐方式1：Lambda 表达式
new Thread(() -> {
    // 任务逻辑
}).start();

// 推荐方式2：方法引用
new Thread(this::processTask).start();

// 推荐方式3：提交到线程池
ExecutorService executor = Executors.newCachedThreadPool();
executor.submit(() -> processTask());
```

### 8.2 避免创建过多线程

```java
// 不好的做法：为每个任务创建新线程
for (int i = 0; i < 1000; i++) {
    new Thread(() -> processTask()).start();  // 可能创建 1000 个线程！
}

// 好的做法：使用线程池
ExecutorService executor = Executors.newFixedThreadPool(10);
for (int i = 0; i < 1000; i++) {
    executor.submit(() -> processTask());  // 最多 10 个线程
}
```

### 8.3 正确处理线程中断

```java
public class InterruptibleTask implements Runnable {
    @Override
    public void run() {
        try {
            while (!Thread.currentThread().isInterrupted()) {
                // 执行任务
                Thread.sleep(100);
            }
        } catch (InterruptedException e) {
            // 响应中断，清理资源
            Thread.currentThread().interrupt();  // 恢复中断状态
            System.out.println("任务被中断");
        }
    }
}
```

### 8.4 使用 Callable 获取返回值

```java
import java.util.concurrent.Callable;
import java.util.concurrent.Future;

public class CallableDemo {
    public static void main(String[] args) throws Exception {
        Callable<Integer> task = () -> {
            Thread.sleep(1000);
            return 42;
        };
        
        Future<Integer> future = Executors.newSingleThreadExecutor().submit(task);
        Integer result = future.get();  // 阻塞等待结果
        System.out.println("任务返回: " + result);
    }
}
```

## 九、总结

| 维度 | 继承 Thread | 实现 Runnable |
|------|------------|--------------|
| **设计模式** | 继承模式 | 组合模式 |
| **灵活性** | 受限 | 灵活 |
| **资源共享** | 困难 | 容易 |
| **代码复用** | 差 | 好 |
| **推荐场景** | 自定义线程行为 | 普通任务 |

**核心建议**：

1. **优先使用 Runnable**：更灵活，符合面向对象设计原则
2. **使用 Lambda 简化代码**：Java 8+ 推荐方式
3. **使用线程池管理线程**：避免手动创建和管理线程
4. **理解线程生命周期**：避免常见错误

掌握这两种方式的区别和适用场景，你将能够编写出高效、可维护的多线程代码！

**推荐阅读**：
- 《Java 并发编程实战》
- 《深入理解 Java 虚拟机》线程章节
- 《Effective Java》并发相关章节

**相关标签**：Java、多线程、Thread、Runnable、并发编程
