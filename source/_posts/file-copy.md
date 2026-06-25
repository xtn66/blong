---
title: Java 文件复制详解——字符缓冲流与字节缓冲流
date: 2026-06-25
tags:
  - Java
  - 文件复制
  - 字符流
  - 字节流
  - IO流
categories:
  - 学习笔记
---

## 一、概述

文件复制是编程中常见的操作，Java 提供了多种方式实现文件复制。根据文件类型的不同，可以分为两大类：

| 文件类型 | 适用场景 | 推荐流 |
|---------|---------|-------|
| 文本文件 | .txt, .java, .xml 等纯文本 | **字符缓冲流** |
| 二进制文件 | .jpg, .mp4, .zip, .exe 等 | **字节缓冲流** |

**核心区别**：
- **字符流**：按字符读取，适合文本文件，会进行编码转换
- **字节流**：按字节读取，适合任意文件，不进行编码转换

## 二、文本文件复制——字符缓冲流

### 2.1 使用 BufferedReader 和 BufferedWriter

字符缓冲流是处理文本文件最常用的方式，效率高且代码简洁。

```java
import java.io.*;

public class TextFileCopy {
    public static void main(String[] args) {
        String sourcePath = "source.txt";
        String targetPath = "target.txt";
        
        // 方式1：使用 try-catch-finally（传统方式）
        copyTextFileTraditional(sourcePath, targetPath);
        
        // 方式2：使用 try-with-resources（推荐）
        copyTextFileModern(sourcePath, targetPath);
    }

    /**
     * 传统方式：try-catch-finally
     */
    public static void copyTextFileTraditional(String source, String target) {
        BufferedReader reader = null;
        BufferedWriter writer = null;
        
        try {
            reader = new BufferedReader(new FileReader(source));
            writer = new BufferedWriter(new FileWriter(target));
            
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();  // 写入换行符
            }
            
            System.out.println("文本文件复制成功！");
            
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (writer != null) writer.close();
                if (reader != null) reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 现代方式：try-with-resources（Java 7+）
     */
    public static void copyTextFileModern(String source, String target) {
        try (BufferedReader reader = new BufferedReader(new FileReader(source));
             BufferedWriter writer = new BufferedWriter(new FileWriter(target))) {
            
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }
            
            System.out.println("文本文件复制成功！");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 2.2 带缓冲区大小的复制

```java
public class TextFileCopyWithBuffer {
    public static void copy(String source, String target, int bufferSize) {
        try (BufferedReader reader = new BufferedReader(new FileReader(source), bufferSize);
             BufferedWriter writer = new BufferedWriter(new FileWriter(target), bufferSize)) {
            
            char[] buffer = new char[bufferSize];
            int len;
            while ((len = reader.read(buffer)) != -1) {
                writer.write(buffer, 0, len);
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 2.3 处理不同编码格式

```java
public class TextFileCopyWithEncoding {
    public static void copyWithEncoding(String source, String target, String encoding) {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(
                new FileInputStream(source), encoding));
             BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(
                new FileOutputStream(target), encoding))) {
            
            String line;
            while ((line = reader.readLine()) != null) {
                writer.write(line);
                writer.newLine();
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 三、任意文件复制——字节缓冲流

### 3.1 使用 BufferedInputStream 和 BufferedOutputStream

字节缓冲流可以复制任意类型的文件，是"万能复制"方法。

```java
import java.io.*;

public class BinaryFileCopy {
    public static void main(String[] args) {
        String sourcePath = "source.jpg";
        String targetPath = "target.jpg";
        
        // 方式1：使用 try-with-resources
        copyFileModern(sourcePath, targetPath);
        
        // 方式2：自定义缓冲区大小
        copyFileWithBuffer(sourcePath, targetPath, 8192);
        
        // 方式3：单字节复制（不推荐，效率低）
        copyFileSingleByte(sourcePath, targetPath);
    }

    /**
     * 推荐方式：使用 BufferedInputStream 和 BufferedOutputStream
     */
    public static void copyFileModern(String source, String target) {
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(source));
             BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(target))) {
            
            byte[] buffer = new byte[8192];  // 8KB 缓冲区
            int len;
            while ((len = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, len);
            }
            
            System.out.println("文件复制成功！");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 自定义缓冲区大小
     */
    public static void copyFileWithBuffer(String source, String target, int bufferSize) {
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(source), bufferSize);
             BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(target), bufferSize)) {
            
            byte[] buffer = new byte[bufferSize];
            int len;
            while ((len = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, len);
            }
            
            System.out.println("文件复制成功！缓冲区大小: " + bufferSize + " bytes");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 单字节复制（效率极低，不推荐）
     */
    public static void copyFileSingleByte(String source, String target) {
        try (InputStream is = new FileInputStream(source);
             OutputStream os = new FileOutputStream(target)) {
            
            int byteRead;
            while ((byteRead = is.read()) != -1) {
                os.write(byteRead);
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 3.2 复制大文件（带进度显示）

```java
public class LargeFileCopy {
    public static void copyWithProgress(String source, String target) {
        File sourceFile = new File(source);
        long totalSize = sourceFile.length();
        long copiedSize = 0;
        
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(source));
             BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(target))) {
            
            byte[] buffer = new byte[65536];  // 64KB 缓冲区
            int len;
            
            while ((len = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, len);
                copiedSize += len;
                
                // 显示进度
                int progress = (int) ((copiedSize * 100) / totalSize);
                System.out.print("\r复制进度: " + progress + "%");
            }
            
            System.out.println("\n文件复制完成！");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 3.3 复制整个目录

```java
public class DirectoryCopy {
    public static void copyDirectory(String sourceDir, String targetDir) {
        File source = new File(sourceDir);
        File target = new File(targetDir);
        
        // 创建目标目录
        if (!target.exists()) {
            target.mkdirs();
        }
        
        File[] files = source.listFiles();
        if (files != null) {
            for (File file : files) {
                if (file.isDirectory()) {
                    // 递归复制子目录
                    copyDirectory(file.getAbsolutePath(), 
                        target.getAbsolutePath() + File.separator + file.getName());
                } else {
                    // 复制文件
                    copyFile(file.getAbsolutePath(), 
                        target.getAbsolutePath() + File.separator + file.getName());
                    System.out.println("已复制: " + file.getName());
                }
            }
        }
    }
    
    private static void copyFile(String source, String target) {
        try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(source));
             BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(target))) {
            
            byte[] buffer = new byte[8192];
            int len;
            while ((len = bis.read(buffer)) != -1) {
                bos.write(buffer, 0, len);
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 四、Java NIO 文件复制（高效）

Java NIO 提供了更高效的文件复制方式。

### 4.1 使用 Files.copy()

```java
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;

public class NioFileCopy {
    public static void main(String[] args) {
        String source = "source.txt";
        String target = "target.txt";
        
        try {
            // 方式1：基本复制
            Files.copy(Paths.get(source), Paths.get(target));
            
            // 方式2：覆盖已存在的文件
            Files.copy(Paths.get(source), Paths.get(target), 
                StandardCopyOption.REPLACE_EXISTING);
            
            // 方式3：复制目录
            Files.copy(Paths.get(source), Paths.get(target),
                StandardCopyOption.REPLACE_EXISTING,
                StandardCopyOption.COPY_ATTRIBUTES);
            
            System.out.println("NIO 文件复制成功！");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 4.2 使用 Channel 复制

```java
import java.io.*;
import java.nio.channels.FileChannel;

public class ChannelFileCopy {
    public static void copy(String source, String target) {
        try (FileChannel sourceChannel = new FileInputStream(source).getChannel();
             FileChannel targetChannel = new FileOutputStream(target).getChannel()) {
            
            // 方式1：直接传输
            sourceChannel.transferTo(0, sourceChannel.size(), targetChannel);
            
            // 方式2：使用 ByteBuffer
            // ByteBuffer buffer = ByteBuffer.allocate(8192);
            // while (sourceChannel.read(buffer) != -1) {
            //     buffer.flip();
            //     targetChannel.write(buffer);
            //     buffer.clear();
            // }
            
            System.out.println("Channel 文件复制成功！");
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## 五、各种复制方式对比

### 5.1 性能对比

```java
public class CopyPerformanceTest {
    public static void main(String[] args) {
        String source = "large_file.zip";
        String target = "large_file_copy.zip";
        
        // 测试单字节复制
        long start = System.currentTimeMillis();
        BinaryFileCopy.copyFileSingleByte(source, target);
        long end = System.currentTimeMillis();
        System.out.println("单字节复制耗时: " + (end - start) + " ms");
        
        // 测试字节流复制（无缓冲）
        start = System.currentTimeMillis();
        copyWithStream(source, target);
        end = System.currentTimeMillis();
        System.out.println("字节流复制耗时: " + (end - start) + " ms");
        
        // 测试字节缓冲流复制
        start = System.currentTimeMillis();
        BinaryFileCopy.copyFileModern(source, target);
        end = System.currentTimeMillis();
        System.out.println("字节缓冲流复制耗时: " + (end - start) + " ms");
        
        // 测试 NIO Channel 复制
        start = System.currentTimeMillis();
        ChannelFileCopy.copy(source, target);
        end = System.currentTimeMillis();
        System.out.println("NIO Channel 复制耗时: " + (end - start) + " ms");
    }
    
    private static void copyWithStream(String source, String target) {
        try (InputStream is = new FileInputStream(source);
             OutputStream os = new FileOutputStream(target)) {
            
            byte[] buffer = new byte[8192];
            int len;
            while ((len = is.read(buffer)) != -1) {
                os.write(buffer, 0, len);
            }
            
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 5.2 方式对比表

| 方式 | 适用场景 | 效率 | 代码复杂度 |
|------|---------|------|-----------|
| 单字节复制 | 小文件 | ❌ 最低 | ✅ 简单 |
| 字节流复制（无缓冲） | 小文件 | ⚠️ 较低 | ✅ 简单 |
| **字节缓冲流** | **任意文件** | **✅ 较高** | **✅ 简单** |
| **字符缓冲流** | **文本文件** | **✅ 较高** | **✅ 简单** |
| NIO Files.copy() | 任意文件 | ✅ 最高 | ✅ 最简单 |
| NIO Channel | 大文件 | ✅ 最高 | ⚠️ 中等 |

## 六、常见问题与解决方案

### 6.1 文件编码问题

```java
// 问题：复制中文文件出现乱码
// 解决方案：指定编码格式
BufferedReader reader = new BufferedReader(
    new InputStreamReader(new FileInputStream(source), "UTF-8"));
BufferedWriter writer = new BufferedWriter(
    new OutputStreamWriter(new FileOutputStream(target), "UTF-8"));
```

### 6.2 文件过大问题

```java
// 问题：大文件复制内存溢出
// 解决方案：使用缓冲流，分块读取
byte[] buffer = new byte[8192];  // 8KB 缓冲区
int len;
while ((len = bis.read(buffer)) != -1) {
    bos.write(buffer, 0, len);
}
```

### 6.3 文件权限问题

```java
// 问题：复制文件时权限不足
// 解决方案：检查文件权限
File file = new File(source);
if (!file.canRead()) {
    System.err.println("无法读取源文件");
    return;
}
```

### 6.4 文件不存在问题

```java
// 问题：源文件不存在
// 解决方案：检查文件是否存在
File sourceFile = new File(source);
if (!sourceFile.exists()) {
    System.err.println("源文件不存在");
    return;
}
```

## 七、最佳实践

### 7.1 文本文件复制

```java
public static void copyTextFile(String source, String target) {
    try (BufferedReader reader = new BufferedReader(new FileReader(source));
         BufferedWriter writer = new BufferedWriter(new FileWriter(target))) {
        
        String line;
        while ((line = reader.readLine()) != null) {
            writer.write(line);
            writer.newLine();
        }
        
    } catch (IOException e) {
        // 处理异常
        e.printStackTrace();
    }
}
```

### 7.2 任意文件复制

```java
public static void copyFile(String source, String target) {
    try (BufferedInputStream bis = new BufferedInputStream(new FileInputStream(source));
         BufferedOutputStream bos = new BufferedOutputStream(new FileOutputStream(target))) {
        
        byte[] buffer = new byte[8192];
        int len;
        while ((len = bis.read(buffer)) != -1) {
            bos.write(buffer, 0, len);
        }
        
    } catch (IOException e) {
        // 处理异常
        e.printStackTrace();
    }
}
```

### 7.3 使用 NIO（推荐）

```java
import java.nio.file.Files;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;

public static void copyFileNio(String source, String target) {
    try {
        Files.copy(Paths.get(source), Paths.get(target), 
            StandardCopyOption.REPLACE_EXISTING);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

## 八、总结

文件复制是 Java IO 操作的基础，选择合适的复制方式至关重要：

1. **文本文件复制**：使用 **字符缓冲流**（BufferedReader/BufferedWriter），会进行编码转换
2. **任意文件复制**：使用 **字节缓冲流**（BufferedInputStream/BufferedOutputStream），不进行编码转换
3. **大文件复制**：使用 **NIO Channel** 或 **Files.copy()**，效率更高
4. **推荐做法**：使用 **try-with-resources** 自动关闭资源

**核心要点**：
- 不要使用单字节复制，效率极低
- 始终使用缓冲流，减少 IO 次数
- 处理文本文件时注意编码格式
- 使用 try-with-resources 确保资源正确关闭

掌握文件复制的各种方式，你将能够处理各种文件操作场景！

**推荐阅读**：
- 《Java 编程思想》IO 流章节
- 《Effective Java》资源管理章节
- 《深入理解 Java IO 与 NIO》

**相关标签**：Java、文件复制、字符流、字节流、IO流、NIO
