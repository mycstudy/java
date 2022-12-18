# 创建型模式通俗理解与实际应用

这些设计模式提供了一种在创建对象的同时隐藏创建逻辑的方式，而不是使用 new 运算符直接实例化对象。

### 单例模式

适用于保证一个类只有一个实例对象的情形。

常用写法：

- 饿汉式：类加载的时候就创建单例对象。
- 懒汉式：默认不会实例化，什么时候用什么时候new。

#### 静态常量

静态常量方式属于饿汉式，以静态变量的方式声明对象。

```java
public class Singleton {
    public static final Singleton singleton = new Singleton();
    // 私有化构造函数，防止被调用，从而创建对象
    private Singleton() {
    }
}
```

singleton是静态变量，调用时用类名调用。

```java
public class Main {
    public static void main(String[] args) {
        Singleton singleton = Singleton.singleton;

        // 变量可以直接访问，这样会变得不安全，比如被不小心赋值为null
        // 这种问题的通常解决方法是用getter和setter
        singleton = null;
    }
}
```

#### 双重检查机制

双重检查机制方式属于懒汉式。

```java
public class Singleton {
    private volatile Singleton singleton;

    // 私有化构造函数，防止被调用，从而创建对象
    private Singleton() {
    }
    
    public Singleton getInstance() {
        if (singleton == null) {
            synchronized (Singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

**外层判断null的作用**：避免每次获取实例都进入同步块，可以增加效率。

**内层判断null的作用**：避免多次创建对象。和synchronized一起保证只有第一个抢到锁的线程可以创建唯一一个对象。

**volatile关键字的作用**：防止创建对象语句的指令重排序。创建对象语句实际上有多个指令构成，在多线程情况下，这些指令可能会被分配到不同线程中去执行，导致指令重排序（乱序执行），从而导致空指针等异常（如第三步在第二步之前执行）。

- 第一步：分配内存空间给Singleton这个对象
- 第二步：初始化对象
- 第三步：将INSTANCE变量指向Singleton这个对象内存地址



### 建造者模式

通过setter或者有参构造函数，将一个复杂对象（多个参数）的构造和表示分离，使相同的构建过程可以创建不同的表示。当类的属性较多时，为其封装一个建造者类，会使创建对象的时候代码看起来比较优雅。

```java
public class DoContact {
    private final int age;
    private final int safeID;
    private final String name;
    private final String address;

    public int getAge() {
        return age;
    }
    public int getSafeID() {
        return safeID;
    }
    public String getName() {
        return name;
    }
    public String getAddress() {
        return address;
    }

    public static class Builder {
        private int age = 0;
        private int safeID = 0;
        private String name = null;
        private String address = null;

        // 构建的步骤
        public Builder(String name) {
            this.name = name;
        }
        public Builder age(int val) {
            this.age = val;
            return this;
        }
        public Builder safeID(int val) {
            this.safeID = val;
            return this;
        }
        public Builder address(String val) {
            this.address = val;
            return this;
        }
        // 构建，返回一个新对象
        public DoContact build() {
            return new DoContact(this);
        }
    }

    private DoContact(Builder b) {
        age = b.age;
        safeID = b.safeID;
        name = b.name;
        address = b.address;
    }
}

public class Main {
    public static void main(String[] args) {
        DoContact object = new DoContact.Builder("MChopin").age(18)
                .address("shanghai").build();
        System.out.println("name=" + object.getName() + " age=" + object.getAge()
                + " address=" + object.getAddress());
    }
}

```

#### 源码分析

Spring在创建Bean之前，会将每个Bean的声明封装成对应的一个BeanDefinition，而BeanDefinition会封装很多属性（比如他的class类型、Bean的作用域、是否懒加载），所以Spring为了更加优雅地创建BeanDefinition，就提供了BeanDefinitionBuilder这个建造者类。

```java
package org.springframework.beans.factory.support;
public final class BeanDefinitionBuilder {
    private final AbstractBeanDefinition beanDefinition;
	/**
	 * Create a new {@code BeanDefinitionBuilder} used to construct a {@link GenericBeanDefinition}.
	 * @param beanClassName the class name for the bean that the definition is being created for
	 */
	public static BeanDefinitionBuilder genericBeanDefinition(String beanClassName) {
		BeanDefinitionBuilder builder = new BeanDefinitionBuilder(new GenericBeanDefinition());
		builder.beanDefinition.setBeanClassName(beanClassName);
		return builder;
	}
    ...
}

public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor
		implements BeanDefinition, Cloneable {
    ...
}

public class GenericBeanDefinition extends AbstractBeanDefinition{
    ...
}

package org.springframework.beans.factory.config;
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    ...
}
```



### 工厂模式

封装对象的创建过程，调用者不需要关心对象是如何具体创建的，工厂模式适合对象创建过程复杂的场景。

#### 简单工厂模式

不满足开闭原则（“对修改关闭，对扩展开发”）

```java
public class SimpleAnimalFactory {
    public Animal createAnimal(String animalType) {
        if ("cat".equals(animalType)) {
            Cat cat = new Cat();
            //一系列复杂操作
            return cat;
        } else if ("dog".equals(animalType)) {
            Dog dog = new Dog();
            //一系列复杂操作
            return dog;
        } else {
            throw new RuntimeException("animalType=" + animalType + "无法创建对应对象");
        }
    }
}

public class Main{
    SimpleAnimalFactory factory= new SimpleAnimalFactory();
    Animal cat = factory.createAnimal("cat");
}
```

#### 工厂模式

满足开闭原则

```java
public interface AbstractAnimalFactory {
    Animal createAnimal();
}

public class CatFactory implements AbstractAnimalFactory {
    public Animal createAnimal(){
        Cat cat = new Cat();
        //一系列复杂操作
        return cat;
    }
}

public class DogFactory implements AbstractAnimalFactory {
    public Animal createAnimal(){
        Dog dog = new Dog();
        //一系列复杂操作
        return dog;
    }
}

public class Main{
    AbstractAnimalFactory factory= new CatFactory();
    Animal cat = factory.createAnimal();
}
```

#### 抽象工厂模式

不满足开闭原则。

在前两种模式的基础上，使product的种类变多，工厂类需要同时管理多个product。

```java
public interface AbstractAnimalFactory {
    Animal createAnimal();
    Food createFood();
}
```

#### 源码分析

在Mybatis中，当需要调用Mapper接口执行sql的时候，需要先获取到SqlSession，通过SqlSession再获取到Mapper接口的动态代理对象，而SqlSession的构造过程比较复杂，所以就提供了SqlSessionFactory工厂类来封装SqlSession的创建过程。

```java
package org.apache.ibatis.session.defaults;
public class DefaultSqlSession implements SqlSession{
    ...
}
public class DefaultSqlSessionFactory implements SqlSessionFactory {
    @Override
  public SqlSession openSession() {
      return openSessionFromDataSource(configuration.getDefaultExecutorType(), null, false);
  }
    
  private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
    Transaction tx = null;
    try {
      // 准备参数
      final Environment environment = configuration.getEnvironment();
      final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
      tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);
      final Executor executor = configuration.newExecutor(tx, execType);
      // 参数准备完成
      return new DefaultSqlSession(configuration, executor, autoCommit);
    } catch (Exception e) {
      ...
    } finally {
      ...
    }
  }
}

package org.apache.ibatis.session;
public interface SqlSession extends Closeable{
    ...
}

public interface SqlSessionFactory{
    ...
}
```



### 建造者VS工厂

<img src="https://github.com/mycstudy/java/blob/main/design-pattern/design-pattern.assets/1.jpg" alt="1" style="zoom: 45%;" />

建造者模式有点类似于工厂模式，都是用来创建一个对象，但是有很大的区别，主要区别如下：

- 建造者模式用不同的属性值可以创造出不同的产品，而工厂模式创建出来的产品都是一样的。
- 建造者模式**使用者**需要知道这个产品有哪些属性，而工厂模式的使用者不需要知道，直接创建就行。
- 建造者模式先new product对象再确定参数，工厂模式先确定参数再new product对象。













## 参考

第一次写博客，也是边理解边写，侵权就删！

[新同事优化了他接手的垃圾代码，设计模式用的是真优雅呀！代码如诗！ (qq.com)](https://mp.weixin.qq.com/s/hOYGSVaipiFGFGdnQmSoKw)

[(48条消息) 建造者模式(Builder Pattern)_JFS_Study的博客-CSDN博客_建造者模式](https://blog.csdn.net/ChineseSoftware/article/details/123256575)