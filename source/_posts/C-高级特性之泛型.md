---
title: 'C#高级特性（泛型）'
date: 2020-09-29
tags:
- 泛型 特性 高级
categories:
- C#高级特性
---



#### 一、什么是泛型

**泛型（Generic）** 允许延迟编写类或方法中的编程元素的数据类型的规范，直到实际在程序中使用它的时候。换句话说，泛型允许编写一个可以与任何数据类型一起工作的类或方法。

通过数据类型的替代参数编写类或方法的规范。当编译器遇到类的构造函数或方法的函数调用时，它会生成代码来处理指定的数据类型。

**特性**  

- 有助于最大限度地重用代码、保护类型的安全以及提高性能。
- 创建泛型集合类。.NET 框架类库在 *System.Collections.Generic* 命名空间中包含了一些新的泛型集合类。使用这些泛型集合类来替代 *System.Collections* 中的集合类。
- 创建自己的泛型接口、泛型类、泛型方法、泛型事件和泛型委托。
- 对泛型类进行约束以访问特定数据类型的方法。
- 关于泛型数据类型中使用的类型的信息可在运行时通过使用反射获取。

#### 二、为什么要使用泛型

**1.案例**

创建一个简单类具有三个方法，参数分别对应三种不同数据类型。

```c#
	/// <summary>
    /// 多类型示例方法
    /// </summary>
    class DemoMethod
    {
        public void ShowInt(int iP)
        {
            Console.WriteLine("传入的是：" + iP + "类型是：" +iP.GetType());
        }
        public void ShowString(string sP)
        {
            Console.WriteLine("传入的是：" + sP + "类型是：" +sP.GetType());
        }
        public void ShowDatetime(DateTime dtP)
        {
            Console.WriteLine("传入的是：" + dtP + "类型是："+dtP.GetType());
        }
    }

```

第二步在主程序中初始化参数来进行调用，有如下结果：

[![0ZdsGd.png](https://s1.ax1x.com/2020/09/29/0ZdsGd.png)](https://imgchr.com/i/0ZdsGd)

OK 让我们回到方法本身，可以看到除了传入参数不同以外，内部实现完全一致，那我们是不是可以进行一波优化？来看下面：

我们知道**object**是一切类型的父类，并且任何父类出现的地方都可以用其子类代替。了解这个情况那我们来改进其中的方法：


```c#
public void ShowObject(object oP) {
            Console.WriteLine("传入的是：" + oP + "类型是：" + oP.GetType());
}
```

执行效果如下：

![0ZwNWj.png](https://s1.ax1x.com/2020/09/29/0ZwNWj.png)

根据结果看到确实达到了我们的需求，那既然这样的话我们还可以在继续延伸一下，这里问个大家问题 object类型有没有缺点？

对当然有缺点了，来，我们看下面：

1. 装箱拆箱

   装箱：当传入一个int值时，在栈上，object又在堆上，就会把int值copy到堆里；
   拆箱：当使用int值时，又copy到栈里

2. 损耗程序的性能

3. 类型安全问题，传递的对象没有限制

针对以上问题，微软在C#2.0的时候推出了泛型。

#### 三、如何使用泛型

上面的object方法可以修改为以下形式：

```
public void ShowT<T>(T t) {
	Console.WriteLine("传入的是：" + t + "类型是：" + t.GetType());
}
```

T为泛型的类型参数，但在代码中指定T的具体类型时，泛型中的所有T都替换为具体的类型。

主方法中执行结果如下：

[![0ZRKqH.png](https://s1.ax1x.com/2020/09/29/0ZRKqH.png)](https://imgchr.com/i/0ZRKqH)



- 泛型声明方法时，一开始没有确定类型，T是什么都不清楚；
- T要等调用的时候才能去确定；
- 正是因为没有定死，才有无限可能；
- 设计思想--延迟加载：例如分布式缓存队列、EF的延迟加载；

上面使用泛型的是方法，同样类也是可以使用的泛型的，比如：

```c#
class DemoClass<T>
{
    public T t;
}

```

主函数使用：

![0ZOM5T.png](https://s1.ax1x.com/2020/09/29/0ZOM5T.png)

还有泛型接口：

```c#
namespace DemoOne
{
    interface DemoInterface<T>
    {
        T getT(T t);
    }
}
```

泛型委托：

```c#
namespace DemoOne
{
    class Demodelegate
    {
        public delegate void HellowWord<T>(T t);
    }
}
```

#### 四、泛型的约束

表格来至微软官方文档

**C#的值类型**：整型:Int; 长整型:long; 浮点型:float; 字符型:char; 布尔型:bool;

枚举:enum; 结构:struct; 在C#中所有的值类型都继承自:System.ValueType

**C#引用类型：**数组、委托、接口、object、字符bai串、用户定义的类。



| **约束**            | **描述**                                                     |
| ------------------- | ------------------------------------------------------------ |
| where T：结构       | 类型参数必须是值类型。 可以指定除 Nullable 以外的任何值类型。 |
| where T：类         | 类型参数必须是引用类型；这同样适用于所有类、接口、委托或数组类型。 |
| where T：new()      | 类型参数必须具有公共无参数构造函数。 与其他约束一起使用时，new() 约束必须最后指定。 |
| where T：<基类名称> | 类型参数必须是指定的基类或派生自指定的基类。                 |
| where T：<接口名称> | 类型参数必须是指定的接口或实现指定的接口。 可指定多个接口约束。 约束接口也可以是泛型。 |
| where T：U          | 为 T 提供的类型参数必须是为 U 提供的参数或派生自为 U 提供的参数。 |

下面通过几个简单的例子来看下具体的约束：

```C#
  public class Person
  {
        public int Id { get; set; }
        public string Name { get; set; }
        public void Hellow()
        {
            Console.WriteLine("Hellow");
        }
   }
```

```c#
/// <summary>
/// 值类型类型约束 
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="t"></param>
/// <returns></returns>
public static T Get<T>(T t) where T : struct
{
	return t;
}

/// <summary>
/// 引用类型约束
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="t"></param>
/// <returns></returns>
public static T GetClass<T>(T t) where T : class
{
	return t;
}


/// <summary>
/// new()约束
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="t"></param>
/// <returns></returns>
public static T GetNew<T>(T t) where T : new()
{
	return t;
}


/// <summary>
/// 基类约束 约束类T必须是定义的Person或者其子类
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="t"></param>
public static void Show<T>(T t) where T : Person
{
	Console.WriteLine($"{t.Id}_{t.Name}");
	t.Hellow();
}

/// <summary>
/// 接口约束
/// </summary>
/// <typeparam name="T"></typeparam>
/// <param name="a"></param>
/// <param name="b"></param>
/// <returns></returns>
public static T Max<T>(T a, T b) where T : IComparable<T>
{
	return a.CompareTo(b) > 0 ? a : b;
}
```

**注意**：有多个泛型约束时，new()约束一定是在最后。

#### 五、泛型的协变和逆变

协变和逆变是在.NET 4.0的时候出现的，只能放在接口或者委托的泛型参数前面，out 协变covariant，用来修饰返回值；in：逆变contravariant，用来修饰传入参数。

先看下面的一个例子（部分用例和图片转自如下引用 如有侵权请联系本人删除）：

>  cnblogs.com/VVStudy/p/11404300.html
>



```c#
	interface IFoo<out T>
    {
        T GetName();
    }

    class Foo : IFoo<string>
    {
        public string GetName()
        {
            return GetType().Name;
        }
    }
```

主函数：

```c#
	
   IFoo<string> fooStr = new Foo();
   IFoo<object> fooObj =  fooStr;
   object name = fooObj.GetName();

```

一图胜千言：

![](http://5b0988e595225.cdn.sohucs.com/images/20190830/2b7efd63ceb04afa93679ff4f3d52126.png)



```C#
    interface IBar<in T> {
        void Print(T t);
    }

    class Bar : IBar<object>
    {
        public void Print(object t)
        {
            Console.WriteLine(t);
        }
    }
```

```
	IBar<object> barObj = new Bar();
	IBar<string> barStr = barObj;

	barStr.Print("hellow word");
```

![](http://5b0988e595225.cdn.sohucs.com/images/20190830/12182b1ee49a46b8a137a4f51c0691b9.png)

涉及小问题：

**协变、逆变 为什么只能针对泛型接口或者委托？而不能针对泛型类**？

因为它们都只能定义方法成员（接口不能定义字段），而方法成员在创建对象的时候是不涉及到对象内存分配的，所以它们是类型（内存）安全的。

为什么不针对泛型？因为泛型类是模板类，而类成员是包含字段的，不同类型的字段是影响对象内存分配的，没有派生关系的类型它们是不兼容的，也是内存不安全的。



**协变、逆变 为什么是类型安全的？**

本质上是里氏替换原则，由里氏替换原则可知：派生程度小的是派生程度大的子集，所以子类替换父类的位置整个程序功能都不会发生改变。



自此关于C#泛型介绍到这里，针对部分问题还需认真思考，多去琢磨。