[TOC]

# 单例模式（Singleton Pattern）

> - 单例类只能有一个实例。
> - 单例类必须自己创建自己的唯一实例。
> - 单例类必须给所有其他对象提供这一实例。

## 饿汉式单例模式

> **优点**：没有加任何锁、执行效率比较高，用户体验比懒汉式单例模式更好。
>
> **缺点**：类加载……的时候就初始化，不管用与不用都占着空间，浪费了内存。

```java

public class Singleton {

    private static Singleton INSTANCE;
    
    static {
        INSTANCE = new Singleton();
    }

    private Singleton() {
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

## 懒汉式单例模式

> **特点**：被外部类调用的时候内部类才会加载。

### 非线程安全

> 实例可能被实例化多次，有时运行结果为同一对象，实际是因为先被实例化的对象被后实例化的对象覆盖。

```java
public class Singleton {

    private static Singleton INSTANCE;

    private Singleton() {}

    public static Singleton getInstance() {

        if (INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }

}
```

### 线程安全

> 用synchronized加锁时，在线程数量比较多的情况下，如果CPU分配压力上升，则会导致大批线程阻塞，从而导致程序性能大幅下降。

```java
public class Singleton {

    private static Singleton INSTANCE;

    private Singleton() {}

    public static synchronized Singleton getInstance() {

        if (INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }

}
```

###  双重校验锁优化

> 出现阻塞时，阻塞并不是基于整个`Singleton`类的阻塞，而是在`#getInstance`方法内部的阻塞，只要逻辑不太复杂，对于调用者而言感知不到。

```java
public class Singleton {

    private static Singleton INSTANCE;

    private Singleton() {
    }

    public static Singleton getInstance() {

        if (INSTANCE == null) {
            synchronized (Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

### 静态内部类

> 用到 synchronized 关键字总归要上锁，对程序性能还是存在一定影响的。
>
> 可以从类初始化的角度来考虑，采用静态内部类的方式
>
> 这种方式兼顾了饿汉式单例模式的内存浪费问题和`synchronized`的性能问题。内部类一定是要在方法调用之前初始化，巧妙地避免了线程安全问题。

```java
public class Singleton {

    private Singleton() {}

    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }
    
    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
}
```

## 破坏单例

### 反射破坏单例

```java
public class Singleton {

    private Singleton() {}

    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
}
```

```java
public class SingletonTest {

    public static void main(String[] args) {

        try {
            Class<?> clazz = Singleton.class;
            // 通过反射获取私有构造方法
            Constructor c = clazz.getDeclaredConstructor();
            // 设置强制访问
            c.setAccessible(true);

            Object o1 = c.newInstance();

            Object o2 = c.newInstance();

            System.out.println(o1 == o2);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```



### 改造内部类实现

```java
public class Singleton {

    // 使用Singleton的时候，默认先初始化内部类
    // 如果没使用，则内部类是不加载的
    private Singleton() {
        if (SingletonHolder.INSTANCE != null) {
            throw new RuntimeException("");
        }
    }
    // static是为了使单例的空间共享，final保证该方法不被重写、重载
    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
}
```

### 反序列化破坏单例

```java
public class Singleton implements Serializable {

    private Singleton() {
        if (SingletonHolder.INSTANCE != null) {
            throw new RuntimeException("");
        }
    }

    public static final Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
}
```

```java
public class SingletonTest {

    public static void main(String[] args) {
        serialize();
    }
    
    private static void serialize() {
        Singleton s1 = Singleton.getInstance();

        Singleton s2 = null;

        try (FileOutputStream fos = new FileOutputStream("Singleton.obj"); ObjectOutputStream oos = new ObjectOutputStream(fos)) {
            oos.writeObject(s1);
        } catch (IOException e) {
            e.printStackTrace();
        }

        try (FileInputStream fis = new FileInputStream("Singleton.obj"); ObjectInputStream ois = new ObjectInputStream(fis)) {
            s2 = (Singleton) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }

        System.out.println(s1);
        System.out.println(s2);
        System.out.println(s2 == s1);

    }
}
```

```
com.xxx.patterns.singleton.Singleton@7ea987ac
com.xxx.patterns.singleton.Singleton@5fd0d5ae
false
```

### 改造单例实现

> 添加方法`#readResolve`

```java
public class Singleton implements Serializable {

    private Singleton() {
        if (SingletonHolder.INSTANCE != null) {
            throw new RuntimeException("");
        }
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    // 添加方法
    private Object readResolve() {
        return getInstance();
    }
}
```

```
com.xxx.patterns.singleton.Singleton@7ea987ac
com.xxx.patterns.singleton.Singleton@7ea987ac
true
```

### 源码解读

> 查看`ObjectInputStream#readObject`

```java
public final Object readObject() throws IOException, ClassNotFoundException {
    /** if true, invoke readObjectOverride() instead of readObject() */
    if (enableOverride) {
        return readObjectOverride();
    }

    // if nested read, passHandle contains handle of enclosing object
    int outerHandle = passHandle;
    try {
        Object obj = readObject0(false);
        handles.markDependency(outerHandle, passHandle);
        ClassNotFoundException ex = handles.lookupException(passHandle);
        if (ex != null) {
            throw ex;
        }
        if (depth == 0) {
            vlist.doCallbacks();
        }
        return obj;
    } finally {
        passHandle = outerHandle;
        if (closed && depth == 0) {
            clear();
        }
    }
}
```

> 非`ObjectInputStream`子类重写方法`readObjectOverride()`
>
> 进入`readObject0()`

```java
/**
  * Underlying readObject implementation.
  */
private Object readObject0(boolean unshared) throws IOException {
    ...
        
    case TC_ENUM:
    	return checkResolve(readEnum(unshared));

    case TC_OBJECT:
    	return checkResolve(readOrdinaryObject(unshared));

	...
}
```

> `desc.isInstantiable()`判断是否无参构造函数，是则实例化
>
> `desc.hasReadResolveMethod()` 判断是否为空

```java
private Object readOrdinaryObject(boolean unshared) throws IOException {
    if (bin.readByte() != TC_OBJECT) {
        throw new InternalError();
    }

    ObjectStreamClass desc = readClassDesc(false);
    desc.checkDeserialize();

    Class<?> cl = desc.forClass();
    if (cl == String.class || cl == Class.class
        || cl == ObjectStreamClass.class) {
        throw new InvalidClassException("invalid class descriptor");
    }

    Object obj;
    try {
        // 判断是否无参构造函数，是则实例化
        obj = desc.isInstantiable() ? desc.newInstance() : null;
    } catch (Exception ex) {
        throw (IOException) new InvalidClassException(
            desc.forClass().getName(),
            "unable to create instance").initCause(ex);
    }

    passHandle = handles.assign(unshared ? unsharedMarker : obj);
    ClassNotFoundException resolveEx = desc.getResolveException();
    if (resolveEx != null) {
        handles.markException(passHandle, resolveEx);
    }

    if (desc.isExternalizable()) {
        readExternalData((Externalizable) obj, desc);
    } else {
        readSerialData(obj, desc);
    }

    handles.finish(passHandle);

    if (obj != null &&
        handles.lookupException(passHandle) == null &&
        // 判断是否readResolveMethod是否为空
        desc.hasReadResolveMethod())
    {
        Object rep = desc.invokeReadResolve(obj);
        if (unshared && rep.getClass().isArray()) {
            rep = cloneArray(rep);
        }
        if (rep != obj) {
            // Filter the replacement object
            if (rep != null) {
                if (rep.getClass().isArray()) {
                    filterCheck(rep.getClass(), Array.getLength(rep));
                } else {
                    filterCheck(rep.getClass(), -1);
                }
            }
            handles.setObject(passHandle, obj = rep);
        }
    }

    return obj;
}
```

> 进入`ObjectStreamClass#hasReadResolveMethod`

```java
boolean hasReadResolveMethod() {
    requireInitialized();
    return (readResolveMethod != null);
}
```

> 全局搜索`readResolveMethod`
>
> 私有构造函数反射赋值`private ObjectStreamClass(final Class<?> cl){}`

```java
readResolveMethod = getInheritableMethod(cl, "readResolve", null, Object.class);
```

> 反射查找方法`#readResolve`

```java
private static Method getInheritableMethod(Class<?> cl, String name, Class<?>[] argTypes, Class<?> returnType) {
    Method meth = null;
    Class<?> defCl = cl;
    while (defCl != null) {
        try {
            meth = defCl.getDeclaredMethod(name, argTypes);
            break;
        } catch (NoSuchMethodException ex) {
            // 若无则从父类查找
            defCl = defCl.getSuperclass();
        }
    }

    if ((meth == null) || (meth.getReturnType() != returnType)) {
        return null;
    }
    meth.setAccessible(true);
    int mods = meth.getModifiers();
    if ((mods & (Modifier.STATIC | Modifier.ABSTRACT)) != 0) {
        return null;
    } else if ((mods & (Modifier.PUBLIC | Modifier.PROTECTED)) != 0) {
        return meth;
    } else if ((mods & Modifier.PRIVATE) != 0) {
        return (cl == defCl) ? meth : null;
    } else {
        return packageEquals(cl, defCl) ? meth : null;
    }
}
```

> 返回`ObjectInputStream#readOrdinaryObject`查看
>
> 在`invokeReadResolve(obj)`反射调用`readResolveMethod`避免了单例被破坏的问题
>
> 但是实际上实例化了两次，只不过新创建的对象没有被返回而已。
>
> 如果创建对象的动作发生频率加快，就意味着内存分配开销也会随之增大



## 注册式单例

### 枚举式单例

> 基于java虚拟机编译实现
>
> 查看反编译文件
>
> 枚举式单例模式在静态代码块中就给INSTANCE进行了赋值，是饿汉式单例模式的实现。

```java
public enum Singleton {

    INSTANCE;

    public void whatEverMethod() {
        // do something
    }
}
```

### 枚举式破坏单例

#### 反序列化无法破坏解读

> 基于`ObjectInputStream#readObject`

```java
/**
  * Underlying readObject implementation.
  */
private Object readObject0(boolean unshared) throws IOException {
    ...
        
    case TC_ENUM:
    	return checkResolve(readEnum(unshared));

    case TC_OBJECT:
    	return checkResolve(readOrdinaryObject(unshared));

	...
}
```

> `readEnum`方法中无法创建多个枚举类实例

```java
private Enum<?> readEnum(boolean unshared) throws IOException {
    if (bin.readByte() != TC_ENUM) {
        throw new InternalError();
    }

    ObjectStreamClass desc = readClassDesc(false);
    if (!desc.isEnum()) {
        throw new InvalidClassException("non-enum class: " + desc);
    }

    int enumHandle = handles.assign(unshared ? unsharedMarker : null);
    ClassNotFoundException resolveEx = desc.getResolveException();
    if (resolveEx != null) {
        handles.markException(enumHandle, resolveEx);
    }

    String name = readString(false);
    Enum<?> result = null;
    Class<?> cl = desc.forClass();
    if (cl != null) {
        try {
            @SuppressWarnings("unchecked")
            Enum<?> en = Enum.valueOf((Class)cl, name);
            result = en;
        } catch (IllegalArgumentException ex) {
            throw (IOException) new InvalidObjectException(
                "enum constant " + name + " does not exist in " +
                cl).initCause(ex);
        }
        if (!unshared) {
            handles.setObject(enumHandle, result);
        }
    }

    handles.finish(enumHandle);
    passHandle = enumHandle;
    return result;
}
```

#### 反射无法破坏解读

> 进入`java.lang.Enum`

```java
protected Enum(String name, int ordinal) {
    this.name = name;
    this.ordinal = ordinal;
}
```

> 只有一个有参构造函数
>
> 若：反序列化时使用有参构造进行反序列化
>
> 进入`Constructor#newInstance`方法，即可发现枚举类实例化时抛出异常

```java
public T newInstance(Object ... initargs) throws InstantiationException, IllegalAccessException, IllegalArgumentException, InvocationTargetException {
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, null, modifiers);
        }
    }
    if ((clazz.getModifiers() & Modifier.ENUM) != 0)
        throw new IllegalArgumentException("Cannot reflectively create enum objects");
    ConstructorAccessor ca = constructorAccessor;   // read volatile
    if (ca == null) {
        ca = acquireConstructorAccessor();
    }
    @SuppressWarnings("unchecked")
    T inst = (T) ca.newInstance(initargs);
    return inst;
}
```

### 容器式单例

> 适合单例比较多的场景
>
> 例：
>
> ```java
> public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory implements AutowireCapableBeanFactory {
>     ...
>     private final Map<String, BeanWrapper> factoryBeanInstanceCache = new ConcurrentHashMap<String, BeanWrapper>(16);
>     ...
> }
> ```
>
> 注意：`ioc`应线程安全

```java
public class Singleton {

    private Singleton() {}

    private static final Map<String, Object> ioc = new ConcurrentHashMap<>(16);

    public static Object getBean(String className) {

        synchronized (ioc) {
            if (ioc.containsKey(className)) {
                Object obj = null;
                try {
                    obj = Class.forName(className).newInstance();
                    ioc.put(className, obj);
                } catch (IllegalAccessException | InstantiationException | ClassNotFoundException e) {
                    e.printStackTrace();
                }
                return obj;
            } else {
                return ioc.get(className);
            }
        }
    }
}
```

## 线程单例实现ThreadLocal

> 单例模式为了达到线程安全的目的，会给方法上锁，以时间换空间。
>
> `ThreadLocal `将所有的对象全部放在`ThreadLocalMap`中，为每个线程都提供一个对象，实际上是以空间换时间来实现线程隔离的。

```java
public class Singleton {

    private static final ThreadLocal<Singleton> SINGLETON_THREAD_LOCAL = ThreadLocal.withInitial(Singleton::new);

    private Singleton() {}

    public static Singleton getInstance() {
        return SINGLETON_THREAD_LOCAL.get();
    }
}
```

