# Understanding-DI / 理解依赖注入 以及依赖注入在 Angular和vue中的应用

## 先从一个例子开始

> 假设我们现在有汽车和引擎两个类, 任何一辆汽车都需要一个引擎才能启动

> 那么在不使用依赖注入的情况下, 汽车和引擎的关系就像是下述代码

```java
public class Car
{
    public Car()
    {
        GasEngine engine = new GasEngine();
        engine.Start();
    }
}

public class GasEngine
{
    public void Start()
    {
        Console.WriteLine("我吃汽油!");
    }
}
```

> 为了初始化汽车类我们需要这样的实例化

```java
Car car = new Car();
```

> 这段代码的问题是它`高度耦合`了car类和gasEngine类, 如果我们想要去变更我们gasEngine为electricEngine

> car类就必须被重写, 当应用的规模还很小时这样的状况是勉强可以接受的，但是当应用变得越来越大之后，接踵而来的问题和头痛将会让开发者痛不欲生

> 换句话说, 此时我们的​​高级car类依赖于较低级别的gasEngine类，它违反了SOLID的依赖性倒置原则（ Dependency Inversion Principle）

> 依赖倒置原则建议我们应该依赖于抽象，而不是具体的某个类 所以为了满足这个要求，我们引入了IEngine接口和代码的重构

```java
public interface IEngine
    {
        void Start();
    }

    public class GasEngine : IEngine
    {
        public void Start()
        {
            Console.WriteLine("我吃油!");
        }
    }

    public class ElectricityEngine : IEngine
    {
        public void Start()
        {
            Console.WriteLine("我吃电!");
        }
    }

    public class Car
    {
        private readonly IEngine _engine;
        public Car(IEngine engine)
        {
            _engine = engine;
        }

        public void Run()
        {
            _engine.Start();
        }
    }
```

> 现在我们的car类只依赖于IEngine接口，而不是特定的引擎实现

> 现在，唯一的问题是如何创建Car的实例并为其提供一个实际的具体Engine类，如gasEngine或electricEngine

> 此时DI 依赖注入登场

```java
    Car gasCar = new Car(new GasEngine());
    gasCar.Run();
    Car electroCar = new Car(new ElectricityEngine());
    electroCar.Run();
```

> 此时我们简单地将我们的依赖(engine实例)注入到car的构造器内，所以现在类在对象及其依赖项之间实现了松耦合

> 我们也不需要因为添加了新的engine类而去修改我们的car类

> 依赖注入最大的优势是类与类之间都是松耦合的，他们之间不存在`硬性代码依赖`(俗称写死的)

> 依赖注入遵循上面提到的依赖性倒置原则。类并不会引用特定的实现，而是请求在构造类时提供给它们的抽象`在java中通常是接口,在angular中是服务, 在vue中则是模块`

**依赖注入只是一种实现对象及其依赖关系之间松散耦合的技术**

> 不是直接实例化类所需的依赖关系的方式，而是通过向类的构造函数注入依赖关系

## 简单总结一下

> 依赖注入是一种模式, 允许应用`程序动态`地将`对象注入`需要它们的类，而不必强制要求这些类负责这些被其依赖的对象,它允许代码更松散地耦合
