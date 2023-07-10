[TOC]

# 模板模式（Template Pattern）

> 模板模式（Template Pattern）中，一个抽象类公开定义了执行它的方法的方式/模板。
>
> 它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。
>
> 这种类型的设计模式属于行为型模式。

> **优点：**
>
> - 封装不变部分，拓展可变部分
> - 提取公用部分代码，便于维护
> - 行为由父类控制，子类实现
>
> **缺点：**
>
> - 抽象类负责声明最抽象、最一般的事物属性和方法，实现类完成具体的事物和方法。模板方式颠倒
>
>   抽象类定了部分抽象方法，由子类实现，子类的执行结果影响父类的结果

## 简单实现

```java
public abstract class JobActuator {

    /**
     * 前置处理
     */
    protected abstract void before();

    /**
     * 后置处理
     */
    protected abstract void after();

    /**
     * 执行主体
     */
    protected abstract void doJob();

    /**
     * 执行入口
     */
    public void run() {
        before();

        doJob();

        after();
    }

    public static void main(String[] args) {
        JobActuator readJob = new ReadFileJob();
        JobActuator saveJob = new SaveDataJob();

        readJob.run();
        saveJob.run();
    }

}

class ReadFileJob extends JobActuator {

    /**
     * 前置处理
     */
    @Override
    protected void before() {
        System.out.println("开始读取文件, 对文件加读锁!");
    }

    /**
     * 后置处理
     */
    @Override
    protected void after() {
        System.out.println("读取文件结束, 解除加锁!");
    }

    /**
     * 执行主体
     */
    @Override
    protected void doJob() {
        System.out.println("读取文件...");
    }
}

class SaveDataJob extends JobActuator {
    /**
     * 前置处理
     */
    @Override
    protected void before() {
        System.out.println("开始保存数据, 数据准备!");
    }

    /**
     * 后置处理
     */
    @Override
    protected void after() {
        System.out.println("数据保存结束, 数据清理!");
    }

    /**
     * 执行主体
     */
    @Override
    protected void doJob() {
        System.out.println("save data ...");
    }
}
```