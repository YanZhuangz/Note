
[TOC]

# 工厂模式（Factory Pattern）

> 工厂模式（Factory Pattern）是 Java 中最常用的设计模式之一。
>
> 这种类型的设计模式属于创建型模式，它提供了一种创建对象的最佳方式。
>
> 在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。

> **工厂模式优点**
>
> - **可以使代码结构清晰，有效地封装变化**。在编程中，产品类的实例化有时候是比较复杂和多变的，通过工厂模式，将产品的实例化封装起来，使得调用者根本无需关心产品的实例化过程，只需依赖工厂即可得到自己想要的产品。
> - **对调用者屏蔽具体的产品类**。如果使用工厂模式，调用者只关心产品的接口就可以了，至于具体的实现，调用者根本无需关心。即使变更了具体的实现，对调用者来说没有任何影响。
> - **降低耦合度**。产品类的实例化通常来说是很复杂的，它需要依赖很多的类，而这些类对于调用者来说根本无需知道，如果使用了工厂方法，我们需要做的仅仅是实例化好产品类，然后交给调用者使用。对调用者来说，产品所依赖的类都是透明的。

## 简单工厂

> **提供一个创建对象实例的功能，而无需关心其具体实现。被创建实例的类型可以是接口、抽象类，也可以是具体的类。**
>
> 简单工厂模式包含 3 个角色（要素）：
>
> | 名词            | 名称       | 说明                                                         |
> | --------------- | ---------- | ------------------------------------------------------------ |
> | Factory         | 工厂类     | 简单工厂模式的核心部分，负责实现创建所有产品的内部逻辑；工厂类可以被外界直接调用，创建所需对象 |
> | Product         | 抽象类产品 | 它是工厂类所创建的所有对象的父类，封装了各种产品对象的公有方法，<br />它的引入将提高系统的灵活性，使得在工厂类中只需定义一个通用的工厂方法，因为所有创建的具体产品对象都是其子类对象 |
> | ConcreteProduct | 具体产品   | 它是简单工厂模式的创建目标，所有被创建的对象都充当这个角色的某个具体类的实例。它要实现抽象产品中声明的抽象方法 |

### 简单实现

1. 创建方法接口

   ```java
   interface Animal {
       /**
        * Animal Eat
        */
       void eat();
   }
   ```

   

2. 创建接口子类实现类

   ```java
   class Dog implements Animal {
   
       @Override
       public void eat() {
           System.out.println("Dog Eat Something!");
       }
   }
   
   class Cat implements Animal {
   
       @Override
       public void eat() {
           System.out.println("Cat Eat Something!");
       }
   }
   ```

3. 创建工厂类

   ```java
   public class SimpleFactory {
   
       public static Animal getAnimal(String species) {
   
           Animal animal = null;
   
           switch (species) {
               case "DOG":
                   animal = new Dog();
                   break;
               case "CAT":
                   animal = new Cat();
                   break;
               default:
                   break;
           }
           return animal;
       }
   }
   ```

4. 使用

   ```java
   public static void main(String[] args) {
       SimpleFactory.getAnimal("DOG").eat();
       SimpleFactory.getAnimal("CAT").eat();
   }
   ```

### 适用场景

> 当每添加一种动物种类，则必须修改工厂实现，违反开闭原则，不可取；
>
> 所以，简单工厂只适合产品对象比较少的情况，且产品固定的需求，对于产品变化较多的显然不适合

## 工厂方法

> **定义一个创建对象的接口，但让实现这个接口的类来决定实例化哪个类。工厂方法让类的实例化推迟到子类中进行**。
>
> 工厂方法模式包含 4 个角色（要素）：
>
> | 名词                              | 名字                  | 说明                                                         |
> | --------------------------------- | --------------------- | ------------------------------------------------------------ |
> | Product                           | 抽象产品              | 定义工厂方法所创建的对象的接口，也就是实际需要使用的对象的接口 |
> | ConcreteProduct                   | 具体产品              | 具体的Product接口的实现对象                                  |
> | Factory / Creator                 | 工厂接口 / 创建器     | 申明工厂方法，通常返回一个 Product 类型的实例对象            |
> | ConcreteFactory / ConcreteCreator | 工厂实现 / 创建器对象 | 覆盖 Factory 定义的工厂方法，返回具体的 Product 实例         |

### 简单实现

> 基于简单工厂基础实现

1. 创建接口工厂

   ```java
   interface AnimalsFactory {
       /**
        * get some one animal
        * @return Animal
        */
       Animal getAnimal();
   }
   ```

2. 创建各产品线实现

   ```java
   class DogFactory implements AnimalsFactory {
   
       /**
        * get some one animal
        *
        * @return Animal
        */
       @Override
       public Animal getAnimal() {
           return new Dog();
       }
   }
   
   class CatFactory implements AnimalsFactory {
       /**
        * get some one animal
        *
        * @return Animal
        */
       @Override
       public Animal getAnimal() {
           return new Cat();
       }
   }
   ```

3. 使用

   ```java
   public static void main(String[] args) {
       AnimalsFactory factory = new DogFactory();
       Animal animal = factory.getAnimal();
       animal.eat();
   }
   ```


### 适用场景

> 虽然解耦了，也遵循了开闭原则，但是问题根本还是没有解决，如果我需要的产品很多的话，需要创建非常多的工厂，所以这种方式的缺点也很明显；

## 抽象工厂

> 定义：为创建一组相关或者是相互依赖的对象提供的一个接口，而不需要指定它们的具体类。
>
> 抽象工厂模式包含的角色（要素）：
>
> | 名称            | 名称     | 说明                                                         |
> | --------------- | -------- | ------------------------------------------------------------ |
> | AbstractFactory | 抽象工厂 | 用于声明生成抽象产品的方法                                   |
> | ConcreteFactory | 具体工厂 | 实现抽象工厂定义的方法，具体实现一系列产品对象的创建         |
> | AbstractProduct | 抽象产品 | 定义一类产品对象的接口                                       |
> | ConcreteProduct | 具体产品 | 通常在具体工厂里，会选择具体的产品实现，来创建符合抽象工厂定义的方法返回的产品类型的对象。 |
> | Client          | 客户端   | 使用抽象工厂来获取一系列所需要的产品对象                     |

### 简单实现

1. 创建抽象工厂

   ```java
   interface Street {
   
       /**
        * one Animal
        * @return Animal
        */
       Animal newAnimal();
   
       /**
        * one Traffic
        * @return Traffic
        */
       Traffic newTraffic();
   }
   ```

2. 创建具体工厂

   ```java
   class OldStreet implements Street {
       
       @Override
       public Animal newAnimal() {
           return new Cat();
       }
       
       @Override
       public Traffic newTraffic() {
           return new Bicycle();
       }
   }
   
   class FirstStreet implements Street {
   
       @Override
       public Animal newAnimal() {
           return new Dog();
       }
   
       @Override
       public Traffic newTraffic() {
           return new Car();
       }
   }
   ```

3. 抽象产品

   ```java
   interface Traffic {
       /**
        * 
        */
       void showModel();
   }
   ```

4. 具体产品

   ```java
   class Bicycle implements Traffic {
   
       @Override
       public void showModel() {
           System.out.println("自行车");
       }
   }
   
   class Car implements Traffic {
   
       @Override
       public void showModel() {
           System.out.println("汽车");
       }
   }
   ```

5. 使用

   ```java
   public static void main(String[] args) {
       Animal animal = new OldStreet().newAnimal();
       Traffic traffic = new FirstStreet().newTraffic();
   
       animal.eat();
       traffic.showModel();
   }
   ```

   

### 适用场景

> 抽象工厂用来解决相对复杂的问题，适用于一系列、大批量的对象生产；
