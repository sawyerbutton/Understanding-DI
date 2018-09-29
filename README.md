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

## 换一个例子我们重头再来一遍

### 在那之前需要了解一些知识

> `依赖反转原则` **dependency Inversion Principle(DIP)** 

> `控制反转` **Inversion of Controls(IOC)**

### 依赖反转原则

> 依赖倒置原则是一种软件设计原则，它为我们提供了编写松散耦合类的指导

> 根据依赖倒置原则的定义:

> 高阶模块不应该依赖于低阶模块, 两者都应该依赖于抽象

> 抽象不应该依赖于细节, 而细节应该依赖于抽象

#### **很难理解的话通过例子来看看**

> 假设我们编写一个在Web服务器上运行的Windows服务 此服务的唯一任务是`在IIS应用程序池`出现问题时在`事件日志中记录消息` 所以我们创建了两个类 一个用于监视应用程序池 另一个用于将错误信息写入日志中

```java
class EventLogWriter
{
    public void Write(string message)
    {
        //Write to event log here
    }
}

class AppPoolWatcher
{
    // Handle to EventLog writer to write to the logs
    EventLogWriter writer = null;

    // This function will be called when the app pool has problem
    public void Notify(string message)
    {
        if (writer == null)
        {
            // initialize an EnventLogWriter instance when writer is null
            writer = new EventLogWriter();
        }
        writer.Write(message);
    }
}
```

> 乍一眼看上去好像这段代码没什么问题, 基本功能也都满足了 但是实际上他违反了依赖反转原则

> 高阶模块`AppPoolWatcher`依赖于`EventLogWriter`, 而`EventLogWriter`是一个具体的类而并非一个抽象

> 这样的说法并不能让你信服，这段代码可能还是可以完美执行当前的任务， 但当我们的需求增加和出现变更的时候你就不会这么认为了

> 出现了问题肯定需要有人修复和处理,oncall就是这么来的, 所以我们下一步的需求是增加一个向网络管理员的邮箱发送和问题相关的邮件的功能, 我们如何基于上述代码实现该功能？

> 有个主意是创建一个用于发送电子邮件并在`AppPoolWatcher`中保留其处理方式(handler)的类，但在任何时候我们都只能使用一个对象要么是`EventLogWriter`或是`EmailSender`, 这就造成了冲突的状况

> 这样的状况可能会随着类似的需求的增加变得更加糟糕, 如果我们还需要像某些开发者发送SMS短信呢？ 按照上面的设想, 我们需要创建更多的实例在`AppPoolWatcher`中的类, 这会变得越来越不可理喻

> 按照赖倒置的原则 我们需要解耦这个系统 更高级别的模块，即我们应用中的`AppPoolWatcher`应当依赖于简单的抽象

### 控制反转

> 依赖倒置是一种软件设计原则, 它说明了两个模块之间应该如何相互依赖, 但是他怎么实现呢？

> 其答案是控制倒置 控制反转是一种实际机制, 使用它我们可以使更高级别的模块依赖于抽象而不是低级模块的具体实现

> 现在将控制反转应用于我们之前的例子中, 我们需要做的第一件事是创建一个可以让更高级别依赖的抽象

> 因此让我们创建一个接口，该接口将提供抽象用来对从`AppPoolWacther`接收的通知进行更具体的操作

```java
public interface INofificationAction
{
    public void ActOnNotification(string message);
}
```
 
> 现在让我们更改我们的更高级别模块`AppPoolWatcher`，以使用此抽象而不是更低级别的具体类

```java
class AppPoolWatcher
{
    // Handle to EventLog writer to write to the logs
    INofificationAction action = null;

    // This function will be called when the app pool has problem
    public void Notify(string message)
    {
        if (action == null)
        {
            // Here we will map the abstraction i.e. interface to concrete class 
        }
        action.ActOnNotification(message);
    }
}
```

> 那我们低级的具体类应该实现我们既定的抽象, 即之前定义的接口

```java
class EventLogWriter : INofificationAction
{
    public void ActOnNotification(string message)
    {
        // Write to event log here
    }
}
```

> 假设我们还需要发送邮件和SMS的抽象类, 那就继续实现上述接口

```java
class EmailSender : INofificationAction
{
    public void ActOnNotification(string message)
    {
        // Send email from here
    }
}

class SMSSender : INofificationAction
{
    public void ActOnNotification(string message)
    {
        // Send SMS from here
    }
}
```

> 迄今为止 我们已经实现控制反转为符合依赖性倒置原则 现在的高级模块仅依赖于抽象而不依赖于较低级别的具体实现 正如依赖性反转原则所述的那样

> 但是看看上面的代码, 我们好像还缺少了什么重要的部分, 我们应该在哪创建具体的类并将其分配给抽象呢？ 当然我们可以这样

```java
class AppPoolWatcher
{
    // Handle to EventLog writer to write to the logs
    INofificationAction action = null;

    // This function will be called when the app pool has problem
    public void Notify(string message)
    {
        if (action == null)
        {
            // Here we will map the abstraction i.e. interface to concrete class
            writer = new EventLogWriter();
        }
        action.ActOnNotification(message);
    }
}
```

> 但是这样我们又回到了最初的原点，创建出的具体的类仍然在更高级别的类中
> 我们能不能完全将具体的类从高阶类中解耦, 即使未来我们创造了新的实现了`INotificationAction`接口的类, 我们也无需修改 `AppPoolWatcher`这个高阶类

### 依赖注入

> 依赖注入主要用于将`具体的实现(低阶类)`注入到使用抽象`(内部接口)的(高阶)类`中

> 依赖注入的主要思想是减少类之间的耦合，并将抽象和具体实现的绑定移出依赖的类

`依赖注入可以通过三种方式实现`

1. 构造器注入
2. 方法注入
3. 属性注入

#### 构造器注入

> 方法如其名, 我们将具体类的对象传递到依赖类的构造器中

```java
class AppPoolWatcher
{
    // Handle to EventLog writer to write to the logs
    INofificationAction action = null;

    public AppPoolWatcher(INofificationAction concreteImplementation)
    {
        this.action = concreteImplementation;
    }

    // This function will be called when the app pool has problem
    public void Notify(string message)
    {   
        action.ActOnNotification(message);
    }
}
```

> 在上面的代码中, 构造函数将获取具体的类对象并将其绑定到接口处理器上

> 因此如果我们需要将`EventLogWriter`的具体实现传递给`AppPoolWatcher`类，我们只需要

```java
EventLogWriter writer = new EventLogWriter();
AppPoolWatcher watcher = new AppPoolWatcher(writer);
watcher.Notify("Sample message to log");
```

> 现在, 如果我们希望这个类发送电子邮件或短信, 我们需要做的就是传递相应类的对象给`AppPoolWatcher`类的构造器

##### _注意！ 当我们知道依赖类的实例在其整个生命周期中将使用相同的具体类时，构造器方法将会很有用_

#### 方法注入

> 在构造函数注入中，我们看到依赖类将在其整个生命周期中使用相同的具体类

> 现在，如果我们需要在每次调用方法时传递单独的具体类，我们就需要在方法中传递依赖项

> 在方法注入中，我们将具体类的对象传递给实际调用该操作的依赖类的方法

```java
class AppPoolWatcher
{
    // Handle to EventLog writer to write to the logs
    INofificationAction action = null;

    // This function will be called when the app pool has problem
    public void Notify(INofificationAction concreteAction, string message)
    {
        this.action = concreteAction;
        action.ActOnNotification(message);
    }
}
```

> 在上面的代码中 `Notify`方法将采用具体的类对象并将其绑定到接口处理器

> 所以当我们传递`EventLogWriter` 的具体类对象到`AppPoolWatcher`中时, 我们只需要做的是

```java
EventLogWriter writer = new EventLogWriter();
AppPoolWatcher watcher = new AppPoolWatcher();
watcher.Notify(writer, "Sample message to log");
```

> 实现邮件或SMS发送的原理和上述一致

### 属性注入

> 考虑了上述的状况, 如果选择具体类和调用方法的责任在不同的地方, 我们就需要属性注入了

> 在这种方法中，我们通过依赖类公开的setter属性传递具体类的对象

> 我们需要做的是在依赖类中使用一个Setter属性或函数，它将获取具体的类对象并将其分配给该类正在使用的接口处理器

```java
class AppPoolWatcher
{
    // Handle to EventLog writer to write to the logs
    INofificationAction action = null;

    public INofificationAction Action
    {
        get
        {
            return action;
        }
        set
        {
            action = value;
        }
    }

    // This function will be called when the app pool has problem
    public void Notify(string message)
    {   
        action.ActOnNotification(message);
    }
}
```

> 在上面的代码中，Action的setter属性将获取具体的类对象并将其绑定到接口处理器中

> 如果我们需要将EventLogWriter的具体实现传递给这个类，我们只需要

```java
EventLogWriter writer = new EventLogWriter();
AppPoolWatcher watcher = new AppPoolWatcher();
// This can be done in some class
watcher.Action = writer;

// This can be done in some other class
watcher.Notify("Sample message to log");
```

> 如果我们希望这个类发送电子邮件或短信，我们需要做的就是在`AppPoolWatcher类`公开的setter中传递相应的类的对象

#### 对于三种依赖注入的区别的理解(alpha)

> 对于构造器注入而言
1. 当类需要一个或多个依赖项时, 构造器注入是此场景最常见的方法
2. 构造器注入可以构建一个稳定的依赖关系, 贯穿整个类或组件的生命周期
3. 构造器注入的方式支持unit test
4. 构造器注入使依赖项显式化并强制客户端(client)提供实例并且可以保证客户端以后无法更改实例
5. 构造器注入必须在构造器函数中添加一个参数, 这可能是一个因人而异的缺点

> 对于属性注入而言
1. 属性注入允许用户自己操控注入的时间节点, 换句话说, 某些消耗巨大'造价昂贵'的服务或者资源可以尽可能迟的情况下注入, 从而节省了一部分的资源
2. 属性注入可能难以识别需要注入什么依赖, 但是却不需要添加和修改构造器
3. 需要注意的是, 在使用属性注入的时候需要确保主实体是已经存在而不是null
4. 属性注入在某种程度上可以增加依赖的灵活性，
5. 属性注入在希望类/实体默认创建一个真实的数据存储库时会有优势, 因为可以在之后的测试中您可以使用setter将其替换为测试实例, 并且属性在实体/类的生命周期中可以替换, 也就意味着依赖在生命周期中可以替换, 这样的便利同样也带来了安全性地问题
6. 属性注入在包含多个参数依赖的配置类中可能会更加常见
7. 显而易见, 构造器注入的注入时间总是早于属性注入的

> 对于方法注入而言
1. 方法注入是最不常见的依赖注入方式, 通常只会在特定的情况下出现
2. 总结来说, 当你的实体/类 只需要这样一个方法而不是依赖的时候使用方法注入
3. 这听起来很荒谬, 既然我都只需要一个方法了那我为什么还需要类呢，但是总有各种各样古怪的情况出现

> spring 支持构造器注入和属性注入两种依赖注入方式

#### _注意！在不支持属性的语言中,可以存在一个单独的函数来设置依赖项 这种方法也称为设置器注射 在这种方法中需要注意的重要一点是,可能已经创建了依赖类,但是还没有设置具体的类依赖。如果我们尝试在这种情况下调用操作,那么我们应该将一些默认依赖项映射到依赖类(保证依赖不为空),或者使用某种机制来确保应用程序正常运行。_

### IOC Container / 控制反转容器

> 依赖注入的所有三种方法只有一个依赖级的状况下都是好用的

> 但是如果一些具体类也依赖于其他一些抽象的情况依然存在(比如在angular中service依赖于其他的service)

> 如果我们有链接和嵌套的依赖项，实现依赖注入将变得非常复杂

> 这就是我们可以使用IoC容器的地方, 当我们有链接或嵌套依赖项时,IoC容器将帮助我们轻松映射依赖项

> IOC Container在前端中的更适合拿Angular作为介绍
