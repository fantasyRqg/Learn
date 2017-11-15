# # Singleton 实现方式
程序内访问同一个实例，需要保持在多线程情况
## Static 初始化方式
利用java 类 static 的几种特性实现单例的初始化
### Static variable
静态变量方式。依赖java 类的静态成员只在类加载时初始化保证保证单例特性，保证线程安全

```
public class A {
    private A(){}
    private static A sInstance = new A();


    public static A getInstance() {
        return sInstance;
    }
}
```



### Static block 
静态block 快初始化单例，类在加载的时候会执行静态块的内容，由此来保证单例和线程安全。相对于 ```static variable``` 静态块的初始化方式可以做更多的保障处理，比如错误捕捉。

```
public class A {
    private A() { }

    private static A sInstance = null;

    static {
        try {
            sInstance = new A();
        } catch (Exception e) {
            System.out.println("Unable to init instance");
        }
    }


    public static A getInstance() {
        return sInstance;
    }
}
```
### Inner class static variable
内部类的静态成员变量，相对于前两种，此种有懒加载的优势。

```
public class A {
    private A() {
    }


    private static class SingletonHleper {
        static A sInstance = new A();
    }


    public static A getInstance() {
        return SingletonHleper.sInstance;
    }
}
```
静态内部类也可以使用`static block` 的方式初始化实例。内部类只有在调用到 `getInstatnce()`方法的时候才会被夹在，从而实现懒加载。
   

## 利用锁机制实现Singleton


```
public class A {
    private static A sInstance = null;


    public static A getInstance() {
        if (sInstance == null) {
            synchronized (A.class) {
                if (sInstance == null)
                    sInstance = new A();
            }
        }

        return sInstance;
    }
}
```

双重判断是用于避免后续实例获取加锁，提高效率。


----

**上述的几种方式都都会面临 `Serialization & Reflection` 的问题。**
Serialization 的问题可以通过实现 `readResolve()` 函数解决。但是不能解决反射的问题，示例代码如下：

```
    @SuppressWarnings("unused")
    private Foo readResolve() {
        return FooLoader.INSTANCE;
    }
``` 

## enum 单例
```
public enum A {
    INSTANCE,;


    public void funA() {
        System.out.println("I'm A");
    }
}
```

得益于java `enum` 机制， 我们可以用于实现单例，此方法可以解决 `Serialization & Reflection` 的问题。但是 enum 单例无法在 Serialization 过程中保存状态（无法在程序关闭后通过反序列化获取内部数据）。java enum 的序列化和反序列话与普通java 类不同，序列化只会存储 name 不会存储其他数据，反序列化通过name过去实例。 


