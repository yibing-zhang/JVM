# 1.对象的创建过程

1. class loading
2. class linking(verification,preparation,resolution)
3. class initializing
4. 申请对象内存
5. 给对象的成员变量赋默认值
6. 调用构造方法<init>
    * 成员变量顺序赋初始值
    * 执行构造方法语句(如果有父类，则先调用父类的构造方法)
# 2.对象在内存中的存储布局
由于对象在内存中分配非常的依赖环境配置，所以先看下虚拟机的配置

### 1.观察虚拟机的配置

```plain
java -XX:+PrintCommandLineFlags -version
```
# ![图片](https://uploader.shimo.im/f/mAug7QIDZtkB2hyq.png!thumbnail)

* 普通对象(

new Object() = 12个字节（对象头+classpoint = 12,由于有padding对齐：8的倍数，因此至少为16个字节）

#### 对象头：markword  8个字节

#### ClassPoint指针：(默认4个字节)

指向xxxx.class源文件

```plain
-XX:+UseCompressedClassPointers 为4字节 不开启为8字节
```
#### 实例数据（成员变量，没有成员变量则为0）：

```plain
引用类型：-XX:+UseCompressedOops 为4字节 不开启为8字节 
```
#### Padding对齐

不占用内存，知识一个对象的内存分配规则：以8的倍数为单位分配，比如new Object是12，但还是要分配16个字节


* 数组对象(默认20字节，关闭classpoint压缩时为24字节)

比普通对象多了一个数组长度(4个字节)

# 3.对象头具体包括什么

* 锁状态信息两位代表对象有没有被锁定
* GC标记被回收了多少次了，分代年龄
# 4.对象怎么定位，即T t = new T();这个t时如何找到这个对象的？
```xml
参考文章：https://blog.csdn.net/clover_lily/article/details/80095580
```

1. 句柄池
2. 直接指针

![图片](https://uploader.shimo.im/f/2UKy0SGXZTWZ6InP.png!thumbnail)

这两者没有优劣之分，有的虚拟机使用句柄池，有的虚拟机使用直接指针

HotSpot使用的是直接指针，直接指针效率比较高直接找到对象，

句柄池要找一个指针再找下一个，但是GC的时候效率比较高

# 5.对象怎么分配
GC内容

# 6.Object c = new Object()在内存中占用多少字节？
**普通对象：16字节**

mark word(8)

class point(4)

padding(4) = 16

**new int[] :16个字节**

mark word(8)

class point(4)

数组长度(4)

padding(0) = 16

# 7.为什么GC年龄默认为15？
最大是15


