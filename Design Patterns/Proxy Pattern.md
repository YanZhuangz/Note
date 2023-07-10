[TOC]

# 代理模式(Proxy Pattern)

> 是指为其他对象提供一种代理，以控制对这个对象的访问。
>
> 代理对象在客户端和目标对象之间起到中介作用。
> 
> 属于结构型设计模式

适用场景

- 保护目标对象
- 增强目标对象

## 静态代理

### 简单实现

假设一个快递服务，设计一个接口
```java
public interface CourierService {

    /**
     * 快递服务
     */
    void doSomething();
    
}
```
假设顺丰公司提供顺丰快递服务
```java
public class SFCourierService implements CourierService {

    @Override
    public void doSomething() {
        System.out.println("收送顺丰快递！");
    }
}
```
开设一家顺丰服务网点, 提供快递服务, 代理顺丰快递服务

```java
public class SFExpressCourierServiceNode implements CourierService {

    /**
     * 提供顺丰快递服务
     */
    private SFCourierService courierService;

    public SFExpressCourierServiceNode(SFCourierService courierService) {
        this.courierService = courierService;
    }

    @Override
    public void doSomething() {

        before();
        courierService.doSomething();
        after();

    }

    private void before() {
        System.out.println("顺丰快递提供服务！");
    }

    private void after() {
        System.out.println("感谢使用顺丰快递！");
    }


    public static void main(String[] args) {
        new SFExpressCourierServiceNode(new SFCourierService()).doSomething();

    }

}
```
输出
```txt
顺丰快递提供服务！
收送顺丰快递！
感谢使用顺丰快递！
```

## 动态代理

> 动态代理与静态思考点
>
> 网点直接传入快递服务，直接代理快递服务，即使有新的快递服务进来，我也可以代理，为什么要用到动态代理？
>
> 假设业务场景变更, 添加新的业务代理，非同一个接口，那么代理类变更就会很大。
>

### JDK动态代理

#### 简单实现

快递服务接口保持不变，变更代理实现
```java
public class CourierServiceProxy implements InvocationHandler {

    private CourierService courierService;

    public CourierService getInstance(CourierService courierService) {
        this.courierService = courierService;

        Class<?> clazz = courierService.getClass();

        return (CourierService) Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(), this);
    }


    /**
     * InvocationHandler接口实现，没有另外单独设计，如果代理接口变更，此处则需变更
     * 针对代理业务的变更，对应代理接口实现的设计较好
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        before();
        Object o = method.invoke(this.courierService, args);
        after();
        return o;
    }

    private void before() {
        System.out.println("顺丰快递提供服务！");
    }

    private void after() {
        System.out.println("感谢使用顺丰快递！");
    }

    public static void main(String[] args) {
        CourierService sf = new CourierServiceProxy().getInstance(new SFCourierService());
        sf.doSomething();
    }
}
```

### CGLIB动态代理

#### 简单实现

```java
public class CourierServiceProxy implements MethodInterceptor {

    public <T> T getInstance(Class<T> clazz) {

        Enhancer enhancer = new Enhancer();

        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);

        return (T) enhancer.create();

    }

    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {

        before();
        Object result = methodProxy.invokeSuper(o, objects);
        after();

        return result;
    }

    private void before() {
        System.out.println("顺丰快递提供服务！");
    }

    private void after() {
        System.out.println("感谢使用顺丰快递！");
    }

    public static void main(String[] args) {
        new CourierServiceProxy().getInstance(SFCourierService.class).doSomething();
    }
}
```

### JDK与CGLIB的区别

- JDK动态代理只能对实现了接口的类生成代理，而不能针对类
- CGLib是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法（继承）

### Spring在选择用JDK还是CGLib的依据

- 当Bean实现接口时，Spring就会用JDK的动态代理
- 当Bean没有实现接口时，Spring使用CGLib来实现
- 可以强制使用CGLib（在Spring配置中加入<aop:aspectj-autoproxy proxy-target-class=“true”/>）

### JDK和CGLib的性能对比

 - 使用CGLib实现动态代理，CGLib底层采用ASM字节码生成框架，使用字节码技术生成代理类，在JDK1.6之前比使用Java反射效率要高。唯一需要注意的是，CGLib不能对声明为final的方法进行代理，因为CGLib原理是动态生成被代理类的子类。
- 在JDK1.6、JDK1.7、JDK1.8逐步对JDK动态代理优化之后，在调用次数较少的情况下，JDK代理效率高于CGLib代理效率，只有当进行大量调用的时候，JDK1.6和JDK1.7比CGLib代理效率低一点，但是到JDK1.8的时候，JDK代理效率高于CGLib代理