---
title: 匿名内部类的使用场景——计算方法执行时间
date: 2026-06-25
tags:
  - 匿名内部类
  - Java
  - 设计模式
  - AOP
  - 性能监控
categories:
  - 学习笔记
---

## 一、概述

匿名内部类（Anonymous Inner Class）是 Java 中一种特殊的类，它没有类名，是在使用时直接定义和实例化的类。匿名内部类通常用于创建一次性使用的对象，特别适合于实现接口或继承类的场景。

**语法**：

```java
new 父类/接口() {
    // 匿名内部类的成员和方法
};
```

## 二、核心应用场景：计算方法执行时间

### 场景说明

在开发中，我们经常需要监控某个方法的执行时间，以便进行性能分析和优化。使用匿名内部类可以优雅地实现这个功能，将计时逻辑与业务逻辑分离。

### 例1：基础实现

**定义一个可执行接口**：

```java
public interface Executable {
    void execute();
}
```

**创建计时工具类**：

```java
public class TimerUtil {
    public static long measureTime(Executable executable) {
        long startTime = System.currentTimeMillis();
        executable.execute();
        long endTime = System.currentTimeMillis();
        return endTime - startTime;
    }

    public static long measureTimeNanos(Executable executable) {
        long startTime = System.nanoTime();
        executable.execute();
        long endTime = System.nanoTime();
        return endTime - startTime;
    }

    public static void measureAndPrint(Executable executable, String methodName) {
        long startTime = System.currentTimeMillis();
        executable.execute();
        long endTime = System.currentTimeMillis();
        System.out.println("方法 [" + methodName + "] 执行时间: " + (endTime - startTime) + " ms");
    }
}
```

**使用匿名内部类调用**：

```java
public class PerformanceTest {
    public static void main(String[] args) {
        // 场景1：计算排序算法执行时间
        long sortTime = TimerUtil.measureTime(new Executable() {
            @Override
            public void execute() {
                int[] array = generateRandomArray(100000);
                Arrays.sort(array);
            }
        });
        System.out.println("排序执行时间: " + sortTime + " ms");

        // 场景2：计算数据库查询时间
        TimerUtil.measureAndPrint(new Executable() {
            @Override
            public void execute() {
                // 模拟数据库查询
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "数据库查询");

        // 场景3：计算复杂计算时间
        long computeTime = TimerUtil.measureTimeNanos(new Executable() {
            @Override
            public void execute() {
                // 模拟复杂计算
                double result = 0;
                for (int i = 0; i < 1000000; i++) {
                    result += Math.sin(i) * Math.cos(i);
                }
            }
        });
        System.out.println("计算执行时间: " + (computeTime / 1_000_000.0) + " ms");
    }

    private static int[] generateRandomArray(int size) {
        int[] array = new int[size];
        Random random = new Random();
        for (int i = 0; i < size; i++) {
            array[i] = random.nextInt();
        }
        return array;
    }
}
```

**输出结果**：

```
排序执行时间: 15 ms
方法 [数据库查询] 执行时间: 102 ms
计算执行时间: 23.56 ms
```

### 例2：带返回值的执行时间计算

**定义带返回值的接口**：

```java
public interface Callable<T> {
    T call() throws Exception;
}
```

**扩展计时工具类**：

```java
public class TimerUtil {
    public static <T> TimedResult<T> measureTimeWithResult(Callable<T> callable) {
        long startTime = System.currentTimeMillis();
        T result = null;
        Exception exception = null;
        try {
            result = callable.call();
        } catch (Exception e) {
            exception = e;
        }
        long endTime = System.currentTimeMillis();
        
        return new TimedResult<>(result, endTime - startTime, exception);
    }

    public static class TimedResult<T> {
        private final T result;
        private final long durationMs;
        private final Exception exception;

        public TimedResult(T result, long durationMs, Exception exception) {
            this.result = result;
            this.durationMs = durationMs;
            this.exception = exception;
        }

        public T getResult() { return result; }
        public long getDurationMs() { return durationMs; }
        public boolean hasException() { return exception != null; }
        public Exception getException() { return exception; }
    }
}
```

**使用示例**：

```java
public class DataProcessor {
    public static void main(String[] args) {
        // 计算并获取返回值
        TimerUtil.TimedResult<Integer> result = TimerUtil.measureTimeWithResult(new Callable<Integer>() {
            @Override
            public Integer call() throws Exception {
                // 模拟数据处理
                Thread.sleep(50);
                return 42;
            }
        });

        System.out.println("执行时间: " + result.getDurationMs() + " ms");
        System.out.println("返回结果: " + result.getResult());

        // 处理可能的异常
        if (result.hasException()) {
            System.err.println("执行出错: " + result.getException().getMessage());
        }
    }
}
```

## 三、匿名内部类的其他使用场景

### 场景1：事件监听器

在 GUI 编程中，匿名内部类常用于事件处理：

```java
public class ButtonExample {
    public static void main(String[] args) {
        JButton button = new JButton("点击我");
        
        button.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                System.out.println("按钮被点击了！");
            }
        });

        button.addMouseListener(new MouseAdapter() {
            @Override
            public void mouseEntered(MouseEvent e) {
                System.out.println("鼠标进入按钮区域");
            }

            @Override
            public void mouseExited(MouseEvent e) {
                System.out.println("鼠标离开按钮区域");
            }
        });
    }
}
```

### 场景2：线程创建

```java
public class ThreadExample {
    public static void main(String[] args) {
        // 创建线程执行任务
        new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 10; i++) {
                    System.out.println("线程执行: " + i);
                    try {
                        Thread.sleep(100);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }).start();

        // 匿名内部类继承 Thread
        new Thread() {
            @Override
            public void run() {
                System.out.println("继承 Thread 的匿名内部类");
            }
        }.start();
    }
}
```

### 场景3：集合排序比较器

```java
public class SortExample {
    public static void main(String[] args) {
        List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

        // 使用匿名内部类自定义排序
        Collections.sort(names, new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                // 按字符串长度排序
                return s1.length() - s2.length();
            }
        });

        System.out.println("按长度排序: " + names);

        // 按字母逆序排序
        Collections.sort(names, new Comparator<String>() {
            @Override
            public int compare(String s1, String s2) {
                return s2.compareTo(s1);
            }
        });

        System.out.println("按字母逆序: " + names);
    }
}
```

### 场景4：资源管理模板方法

```java
public class ResourceManager {
    public interface ResourceOperation<T, R> {
        R execute(T resource) throws Exception;
    }

    public static <T, R> R withResource(T resource, ResourceOperation<T, R> operation) {
        try {
            return operation.execute(resource);
        } catch (Exception e) {
            throw new RuntimeException("资源操作失败", e);
        }
    }

    public static void main(String[] args) {
        // 使用匿名内部类处理文件资源
        String content = withResource(new BufferedReader(new FileReader("data.txt")), 
            new ResourceOperation<BufferedReader, String>() {
                @Override
                public String execute(BufferedReader reader) throws Exception {
                    StringBuilder sb = new StringBuilder();
                    String line;
                    while ((line = reader.readLine()) != null) {
                        sb.append(line).append("\n");
                    }
                    return sb.toString();
                }
            });

        System.out.println("文件内容: " + content);
    }
}
```

## 四、Lambda 表达式 vs 匿名内部类

在 Java 8 之后，Lambda 表达式可以替代大部分匿名内部类的使用场景：

### 对比示例

**匿名内部类**：

```java
TimerUtil.measureTime(new Executable() {
    @Override
    public void execute() {
        System.out.println("执行任务");
    }
});
```

**Lambda 表达式**：

```java
TimerUtil.measureTime(() -> System.out.println("执行任务"));
```

### 区别总结

| 特性 | 匿名内部类 | Lambda 表达式 |
|------|-----------|--------------|
| 适用范围 | 接口或类 | 仅函数式接口 |
| this 指向 | 指向匿名类本身 | 指向外部类 |
| 变量访问 | 可访问实例字段 | 只能访问有效 final 变量 |
| 性能 | 每个匿名类生成一个 class 文件 | 更轻量 |
| 可读性 | 代码冗长 | 简洁明了 |

### 何时使用匿名内部类

尽管 Lambda 表达式更简洁，但在以下情况下仍需使用匿名内部类：

1. **需要继承类而非实现接口**
2. **需要实现多个接口**
3. **需要定义构造函数**
4. **需要添加额外的方法或字段**
5. **需要访问非 final 的局部变量**

## 五、匿名内部类的优缺点

### 优点

1. **代码简洁**：避免创建单独的类文件
2. **逻辑内聚**：相关代码集中在一起
3. **快速实现**：适合一次性使用的场景
4. **封装性好**：类只在使用处可见

### 缺点

1. **可读性差**：复杂逻辑会使代码难以理解
2. **难以复用**：无法在其他地方重用
3. **调试困难**：匿名类没有明确的类名
4. **内存开销**：每个匿名类都会生成独立的 class 文件

## 六、最佳实践

### 1. 保持匿名内部类简短

匿名内部类的代码应该尽量简短，建议不超过 10-15 行：

```java
// 好的做法
TimerUtil.measureTime(() -> {
    // 简短的业务逻辑
    processData();
});

// 不好的做法：逻辑过于复杂
TimerUtil.measureTime(() -> {
    // 大量复杂逻辑...
    // ...
    // ...
});
```

### 2. 使用有意义的变量名

```java
// 好的做法
Executable databaseQuery = () -> { /* 查询逻辑 */ };
long time = TimerUtil.measureTime(databaseQuery);

// 不好的做法
Executable e = () -> { /* 查询逻辑 */ };
long t = TimerUtil.measureTime(e);
```

### 3. 优先使用 Lambda 表达式

在 Java 8+ 环境下，优先使用 Lambda 表达式替代匿名内部类：

```java
// Java 8+
TimerUtil.measureTime(() -> processData());

// 旧版 Java
TimerUtil.measureTime(new Executable() {
    public void execute() {
        processData();
    }
});
```

### 4. 提取复杂逻辑到方法

```java
// 好的做法
TimerUtil.measureTime(this::complexOperation);

private void complexOperation() {
    // 复杂逻辑
}

// 不好的做法
TimerUtil.measureTime(() -> {
    // 直接写复杂逻辑
});
```

## 七、总结

匿名内部类是 Java 中一种强大的特性，尤其在以下场景中非常实用：

1. **方法执行时间计算**：将计时逻辑与业务逻辑分离
2. **事件处理**：GUI 编程中的事件监听器
3. **线程创建**：快速创建一次性任务线程
4. **集合排序**：自定义比较器
5. **资源管理**：模板方法模式的实现

随着 Java 8 Lambda 表达式的普及，匿名内部类的使用场景有所减少，但在某些特定情况下仍然是不可或缺的工具。掌握匿名内部类的使用场景和最佳实践，可以写出更加优雅和高效的代码。

**推荐阅读**：
- 《Effective Java》第24条：优先使用静态成员类而非非静态成员类
- 《Java 编程思想》内部类章节
- 《深入理解 Java 虚拟机》类加载机制

**相关标签**：匿名内部类、Java、Lambda、性能监控、AOP
