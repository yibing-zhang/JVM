# 1.Class文件内容格式
![图片](https://uploader.shimo.im/f/P6KGU3aBGU76ljyr.png!thumbnail)

# 2.一个class文件是被加载到内存的过程是怎样的？

1. loading

把一个class文件装到内存里，class文件是一个二进制，一个个的字节

2. linking

* Verification：校验装载进来的class文件是不是符合jvm规定，如果不符合规范，直接就被拒掉了
* Preparation：把class文件静态变量赋默认值，比如int i = 8；那么此时i = 0而不是8
* Resolution：把class文件中的类、方法、属性等符号引用转换为直接引用，常量池里面用到的符号引用，解析为指针、偏移量等内存直接引用地址
3. Initlalizing

给变量赋初始值int i = 8,上个步骤是0，到这一步才真正的赋值为8

![图片](https://uploader.shimo.im/f/2tAE90u22T7d6Cwm.png!thumbnail)

#### 一个class文件被加载到内存之后是创建了两块内容

1.分配一块内存用来存储class文件的内容

2.创建一个class对象，指向class文件内容的内存地址

# 3.类加载器
![图片](https://uploader.shimo.im/f/3PTLY7SpObgq6FmS.png!thumbnail)

![图片](https://uploader.shimo.im/f/4Zj24Mjl6KQFvSq3.png!thumbnail)

# loading:双亲委派

**定义**：如果一个类加载器收到了加载某个类的请求,则该类加载器并不会直接去加载该类,而是把这个请求委派给父类加载器,如果父类加载器已经把这个类加载进内存，则直接返回，如果没有则继续委派给父加载器，以此类推，只有当父类加载器在其搜索范围内无法找到所需的类,并将该结果反馈给子类加载器,子类加载器会尝试去自己加载。

![图片](https://uploader.shimo.im/f/N4VWfBJbfE1rCGsH.png!thumbnail)

# 为什么需要双亲委派？

主要是为了安全。如果有人想替换系统级别的类：String.java。篡改它的实现，但是在这种机制下这些系统的类已经被Bootstrap classLoader加载过了，所以并不会再去加载，从一定程度上防止了危险代码的植入

# 父加载器

父加载器不是“类加载器的加载器”，也不是“类加载器的父类加载器”

# 在Launcher类（ClassLoader的一个包装启动类）中，可以看到类加载器范围

**bootstartap**

![图片](https://uploader.shimo.im/f/aEkPuTfGYehQcutt.png!thumbnail)

**ext**

![图片](https://uploader.shimo.im/f/0UJxVSA8OEH3UITj.png!thumbnail)

**app**

![图片](https://uploader.shimo.im/f/gWslC0JPbdIrXuzQ.png!thumbnail)

# 
# 4.自定义类加载器

* 定义一个类继承ClassLoader
* 重写模板方法:findClass，并在内部调用defineClass

load class---->findClass（模板方法/钩子函数）

```java
public class OverrideFindClass extends ClassLoader {
    /**
     * 框架，类库，tomcat/spring都有自定义的类加载器
     * 自定义类加载器
     * 下面是思路：
     * 1.把class文件以流的方式读入内存
     * 2.调用classLoader的内置方法：defineClass
     *
     */
  @Override
  protected Class<?> findClass(String name) throws ClassNotFoundException {
      File file = new File("", name.replaceAll(".", "/").concat(".class"));
      try {
          FileInputStream fis = new FileInputStream(file);
          ByteArrayOutputStream baos = new ByteArrayOutputStream();
          int b = 0;
          while ((b = fis.read()) != 0) {
              baos.write(b);
          }
          byte[] bytes = baos.toByteArray();
          baos.close();
          //把二进制字节流转化为类
          return defineClass(name, bytes, 0, bytes.length);
      } catch (Exception e) {
          e.printStackTrace();
      }
      return super.findClass(name);
  }
  public static void main(String[] args) throws ClassNotFoundException, IllegalAccessException, InstantiationException {
      ClassLoader classLoader = new OverrideFindClass();
      //name是指项目内，文件的包路径
      Class clazz = classLoader.loadClass("classloader.Hello");
      Hello hello = (Hello) clazz.newInstance();
      hello.m();
      System.out.println(classLoader.getClass().getClassLoader());
      System.out.println(classLoader.getParent());
  }
}
```
### Java可以解释执行也可以即时编译

#### -Xmixed：默认混合模式，解释执行+热点代码编译


* **什么是热点代码编译**？

多次被调用的方法，多次被调用的循环，内部维护了一个计数器，如果发现某个方法被执行了很多次，就会被编译为本地代码，再用的话就可以直接执行


* **既然即时编译运行速度这么快，为什么不直接都编译为本地代码呢**？

第一：Java现在的解释器的效率已经非常高了，在一些代码的执行上也不输给编译器

第二：如果类库的类很多，二话不说直接编译，那速度必然是很慢很慢，所以就采用了混合模式

#### -Xint：解释模式，启动很快，执行稍慢

#### -Xcomp：纯编译模式，启动很慢，执行很快

### 自定义classLoader的父亲


* 继承ClassLoader
* 指定父加载器：super(parent)
```java
public class SetClassLoaderParentBySelf  {
    static OverrideFindClass parent = new OverrideFindClass();
    static class MyLoader extends ClassLoader {
        public MyLoader(){
            super(parent);
            //不指定的话默认是AppClassLoader
            //System.out.println(this.getParent());
        }
    }
    public static void main(String[] args) {
        MyLoader loader = new MyLoader();
    }
}
```
# 5.打破双亲委派
#### 如何打破？


* 重写loadClass
#### 何时打破？


* jdk1.2之前，自定义classLoader都必须重写loadClass()
* 热启动，热部署：tomcat有自定义的classLoader（可以加在同一类库的不同版本，指不同应用，即每个应用都有自己的classLoader，所以出现同名不同版本的类库依然是可以都加载进来的）
#### 如何自定义classLoader实现热部署的效果？

重写loadClass，干掉上一次加载的classLoader

```java
public class DapoShuangQinWeiPai {
  private static class MyLoader extends ClassLoader {
      @Override
      public Class<?> loadClass(String name) throws ClassNotFoundException {
          File file = new File("H:/Learning/jvm/src/main/java/" + name.replace(".", "/").concat(".class"));
          if (!file.exists()) {
              return super.loadClass(name);
          }
          try {
              InputStream is = new FileInputStream(file);
              byte[] b = new byte[is.available()];
              is.read();
              return defineClass(name, b, 0, b.length);
          } catch (IOException e) {
              e.printStackTrace();
          }
          return super.loadClass(name);
      }
  }
  public static void main(String[] args) {
      MyLoader loader = new MyLoader();
      try {
          Class clazz = loader.loadClass("classloader.Hello");
          loader = new MyLoader();
          Class cla = loader.loadClass("classloader.Hello");
          System.out.println(cla == clazz);
      } catch (ClassNotFoundException e) {
          e.printStackTrace();
      }
  }
}
```
# 6.new对象的步骤

1. 给对象申请内存
2. 给对象的成员变量赋默认值
3. 调用构造方法，给成员变量赋初始值
# 7.扩展知识

* 可以加密class不被反编译(利用异或运算)
* lazyloading
# 



