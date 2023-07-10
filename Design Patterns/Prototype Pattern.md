[TOC]

# 原型模式（Prototype Pattern）

> 原型模式（Prototype Pattern）是指原型实例指定创建对象的种类，并且通过复制这些原型创建新的对象。
>
> 场景：
>
> - 类初始化消耗资源较多
> - 使用new生成一个对象需要非常烦琐的过程（数据准备、访问权限等）
> - 构造函数比较复杂
> - 在循环体中产生大量对象
>
> 例：
>
> - Spring中应用`@scope = "prototype"`
> - `JSON.parseObject()`

> **原型模式的优点：**
>
> - [Java](http://c.biancheng.net/java/) 自带的原型模式基于内存二进制流的复制，在性能上比直接 new 一个对象更加优良。
> - 可以使用深克隆方式保存对象的状态，使用原型模式将对象复制一份，并将其状态保存起来，简化了创建对象的过程，以便在需要的时候使用（例如恢复到历史某一状态），可辅助实现撤销操作。
>
> **原型模式的缺点：**
>
> - 需要为每一个类都配置一个 clone 方法
> - clone 方法位于类的内部，当对已有类进行改造的时候，需要修改代码，违背了开闭原则。
> - 当实现深克隆时，需要编写较为复杂的代码，而且当对象之间存在多重嵌套引用时，为了实现深克隆，每一层对象对应的类都必须支持深克隆，实现起来会比较麻烦。因此，深克隆、浅克隆需要运用得当。





## 浅克隆

```java
private Object shallowClone() {

    ConcretePrototype result = new ConcretePrototype();

    result.setAge(this.getAge());
    result.setName(this.getName());
    result.setHobbies(this.getHobbies());
    result.setBirthdy(this.getBirthdy());

    return result;
}
```

## 深克隆

```java
private Object deepClone() {

    try (ByteArrayOutputStream baos = new ByteArrayOutputStream(); ObjectOutputStream oos = new ObjectOutputStream(baos)) {
        oos.writeObject(this);

        ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(bais);

        ConcretePrototype copy = (ConcretePrototype) ois.readObject();
        copy.setAge(15);
        copy.setBirthdy(new Date());

        return copy;

    } catch (IOException | ClassNotFoundException e) {
        e.printStackTrace();
        return null;
    }

}
```

## 完整实现

```java
public class ConcretePrototype implements Cloneable, Serializable {

    private int age;

    private String name;

    private List<String> hobbies;

    private Date birthdy;

    private ConcretePrototype(Builder builder) {
        setAge(builder.age);
        setName(builder.name);
        setHobbies(builder.hobbies);
        setBirthdy(builder.birthdy);
    }

    public ConcretePrototype() {}

    ...
        gettter/setter
    ...

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return deepClone();
//      return shallowClone();
    }

    private Object deepClone() {

        try (ByteArrayOutputStream baos = new ByteArrayOutputStream(); ObjectOutputStream oos = new ObjectOutputStream(baos)) {
            oos.writeObject(this);

            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bais);

            ConcretePrototype copy = (ConcretePrototype) ois.readObject();
            copy.setAge(15);
            copy.setBirthdy(new Date());

            return copy;

        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
            return null;
        }

    }

    private Object shallowClone() {

        ConcretePrototype result = new ConcretePrototype();

        result.setAge(this.getAge());
        result.setName(this.getName());
        result.setHobbies(this.getHobbies());
        result.setBirthdy(this.getBirthdy());

        return result;
    }

    public static void main(String[] args) {

        ConcretePrototype a = ConcretePrototype.builder()
                .age(12)
                .birthdy(new Date())
                .name("A")
                .hobbies(new ArrayList<>())
                .build();

        try {
            ConcretePrototype b = (ConcretePrototype) a.clone();

            System.out.println(b == a);
            System.out.println(b.getHobbies() == a.getHobbies());

            a.setAge(0);
            a.getHobbies().add("123");

            System.out.println(b.getAge());
            System.out.println(b.getHobbies().size());


        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }

    }

    public static Builder builder() {
        return new Builder();
    }

    public static final class Builder {
        private int age;
        private String name;
        private List<String> hobbies;
        private Date birthdy;

        public Builder() {
        }

        public Builder age(int val) {
            age = val;
            return this;
        }

        public Builder name(String val) {
            name = val;
            return this;
        }

        public Builder hobbies(List<String> val) {
            hobbies = val;
            return this;
        }

        public Builder birthdy(Date val) {
            birthdy = val;
            return this;
        }

        public ConcretePrototype build() {
            return new ConcretePrototype(this);
        }
    }
}
```



## 测试对比

```java
public static void main(String[] args) {

    ConcretePrototype a = ConcretePrototype.builder()
        .age(12)
        .birthdy(new Date())
        .name("A")
        .hobbies(new ArrayList<>())
        .build();

    try {
        ConcretePrototype b = (ConcretePrototype) a.clone();

        System.out.println(b == a);
        System.out.println(b.getHobbies() == a.getHobbies());


    } catch (CloneNotSupportedException e) {
        e.printStackTrace();
    }

}
```

### 深克隆结果

```
false
false
```

### 浅克隆结果

```
false
true
```

### 说明

> - 浅克隆
>   - 对象不同
>   - 属性引用地址相同；若指向对象变更，则都引发属性值变更
> - 深克隆
>   - 两个独立对象
>   - 属性值各自指向不同的对象
>
> 当对象之间存在多重嵌套引用时，为了实现深克隆，每一层对象对应的类都必须支持深克隆