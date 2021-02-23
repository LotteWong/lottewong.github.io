---
title: "#Golang# Golang与编程范式之面向对象编程"
date: 2020-3-1 07:00:21
categories: Programming Language
tags:
- Golang
- Programming Paradigm
thumbnail: /images/golang.png
---



介绍 `Golang` 中的**面向对象**思想实践，主要讨论以下**四个部分**：

- **类和对象**
- **封装**
- **继承** 
- **多态**

---



<!-- more -->



# **目录 Table of Contents**

<!-- toc -->

---

## **简介**

> `Golang` 的起源受诸多早期编程语言的影响。类 `C` 让 `Golang` 本质上更倾向于是一门**面向过程**的语言，同时`Golang` 也借鉴了 `Alef` 来设计 `Golang` 的**函数式编程**特性，融合 `CSP` 中使用管道进行通信和控制同步的思想则很好地体现了如何**面向消息**编程。
>
> 虽然 `Golang` 不是一门传统的**面向对象**语言，但是 `Golang` 的设计却深受面向对象思想的影响。我们可以通过一种 `Golang` 的方式来实现面向对象的重要特性，这也是接下来将要讨论的重点。
>

*PS：本文 just 一点自己的见解，学识有限难免有误，也希望可以抛砖引玉，欢迎大家的勘误和讨论╰(￣ω￣ｏ)*

## **类和对象**

<br/>

> 众所周知🤫，**类和对象**是面向对象编程的灵魂（？类定义了一件事物的抽象特点，包含了**数据的形式和对数据的操作**；对象是类的实例，可以通过**构造函数和析构函数**来进行对生成和销毁的特殊处理。

### `C++` 的类和对象

```c++
class Person {
private:
    // 数据成员
    string name;
    
protected:
    // 数据方法
    string getName() {
        return this->name;
    }
    void setName(string name) {
        this->name = name;
    }
    
    
public:
    // 构造函数
    Person(string name) {
        this->name = name;
    }
    // 析构函数
    ~Person() {
        // ...
    }
};

int main() {
    Person* somebody = new Person("Bot");
    cout << somebody->getName() << endl; // Output: Bot
    somebody->setName("Exp");
    cout << somebody->getName() << endl; // Output: Exp
   
    return 0;
}
```

### `Golang`的“类和对象”

```go
type Person struct {
    // 数据成员
    name string
}

// 数据方法
func (this *Person) GetName() string {
    return p.name
}
func (this *Person) SetName(name string) {
    p.name = name
}

// 构造函数
func NewPerson(string name) *Person {
    return &Person{
        name: name,
    }
}

// 析构函数
// 由于 Golang 采用垃圾回收机制，一般不需要显式写析构函数

func main() {
    somebody := NewPerson("Bot")
    fmt.Println(somebody.getName()) // Output: Bot
    somebody.SetName("Exp")
    fmt.Println(somebody.getName()) // Output: Exp
}
```

### 区别联系

#### 耦合程度

- `C++` 的类是面向 `class` 而言的，**数据成员和数据方法都必须在 `class` 内修改**，可见耦合程度较高。

- `Golang` 的“类”是面向 `type` 而言的，**数据成员在 `struct` 内修改**，**数据方法则是可以在任意处增删 `(recv *receiver_type)` 对应的方法**，可见耦合程度较低。

  【PS：这里 `type` 的外延比 `class` 要广，`type` 除包括**自定义类型**外还支持**内置类型的别名**】

#### `this` 指针

- `C++` 对象的 `this` 指针常常是**隐式**的，每一个数据方法实际上都隐式传入了一个指向该对象的 `this` 指针：

  ```c++
  void setName([Person* this], string name) {
  	this->name = name;
  }
  ```

- `Golang` “对象”的 `this` 指针必须是**显式**的，不难看出 `this` 指针是连接 `Golang` 中类型和方法的关键桥梁：

  ```go
  func (this *Person) setName(string name) {
      this.name = name
  }
  ```

#### 构造与析构

- `C++` 的构造函数和析构函数是比较容易理解的，构造函数在对象创建时被**自动调用**，析构函数在对象销毁时被**自动调用**。由于 `C++` 无垃圾回收机制，**对象的生命周期和作用域紧密相关**。
- `Golang` 严格上来说没有构造函数和析构函数的说法，可以**通过用来专门做初始化的函数**来模拟构造函数，而**`defer` 和 `finalizer`** 有类似析构的意味，但本质还是很不同的。由于 `Golang` 有垃圾回收机制，**对象的生命周期取决于何时被 `GC` 进程回收**，一般而言当变量不再被引用就会被垃圾回收掉。

## **封装**

<br/>

> **封装** aka **信息隐藏**，其实包含了两层意思：一是调用方**无须关心**实现细节，二是调用方**无法更改**实现细节。封装在编程语言中一般体现在**访问权限**中。

### `Java` 的封装

- 以经典的 OO 语言 `Java` 为例，由于 `Java` 同时存在**类**和**包**的概念，其访问权限需要考虑到两个维度，相对而言比较复杂。其中，`Java` 的访问权限**通过关键字定义**：
    - `public`：**公共可见**，所有类可见
    - `protected`：**继承可见**，必须为继承关系，允许跨包
    - `[default]`：**包内可见**，不要求继承关系，仅限同包
    - `private`：**私有可见**，仅本类可见

### `Golang` 的封装

- 而在 `Golang` 中没有所谓的类和对象概念（或者说可以用很 `Golang` 的方式类似实现），但引入了**包**管理机制，`Golang` 中只有简化的两层访问权限。其中，`Golang` 的访问权限**通过标识符大小写定义**：
    - `标识符首字母大写`：**包外可见**，所有的包均可见
    - `标识符首字母小写`：**包内可见**，本包文件均可见

### 一些说明

- `Golang` 中的 **`标识符首字母大写`** 类似于 `Java` 中的 **`public`**
- `Golang` 中的 **`标识符首字母小写`** 类似于 `Java` 中的 **`[default]`**

## **继承**

<br/>

> **继承**涉及三方面的内容：一是子类可以**使用**父类的属性和方法，避免重复编码；二是子类可以**覆盖**父类的属性和方法，是实现多态的必要条件之一；三是子类可以**追加**属于自己的属性和方法，完成子类的定制功能。

### `C++` 的继承

```c++
class Parent {
public:
    // ...
    
    virtual void parentFunc() {
        // ...
    }
    
    virtual void parentFunc(params) {
        // ...
    }
    
    virtual void overrideFunc() {
        // Parent class content
    }
}

class Child: public Parent {
public:
    // ...
    
    virtual void overrideFunc() {
        // Child class content
    }
    
    virtual void childFunc() {
        // ...
    }
};

int main() {
    Child* child = new Child();
    child->parentFunc(); // 使用父类方法
    child->overrideFunc(); // 使用已覆盖的子类方法
    child->Parent::overrideFunc(); // 使用未覆盖的父类方法
    child->childFunc(); // 使用子类方法
    
    return 0;
}
```

### `Golang` 的继承

```go
type Parent struct {
    // ...
}

func (p *Parent) ParentFunc() {
    // ...
}

func (p *Parent) OverrideFunc() {
    // Parent class content
}

type Child struct {
    parent Parent
    // ...
}

func (c *Child) OverrideFunc() {
    // Child class content
}

func (c *Child) ChildFunc() {
    // ...
}

func main() {
    child := &Child{}
    child.ParentFunc() // 使用父类方法
    child.OverrideFunc() // 使用已覆盖的子类方法
    child.parent.OverrideFunc() // 使用未覆盖的父类方法
    child.ChildFunc() // 使用子类方法
}
```

### 一些说明

- `C++` 的继承更像是**链式继承**，从父类到子类进行构造，从子类到父类进行访问
- `Golang` 的继承更像是**组合继承**，子类内嵌一个或多个父类
- 从 `Golang` 的继承机制容易看出它支持**多继承**，一些编程语言（如 `Java`）仅支持**单继承**

## **多态**

<br/>

> **多态**指**同一操作**作用于**不同的对象**，可以有不同的解释，产生**不同的执行结果**。当我们讨论多态时，我们常常会讨论**重载**以及**重写和动态绑定**。

### 语言基础

#### 接口

- **类型**绑定**方法集**，**接口**定义**方法集**。如果类型绑定的方法集和接口定义的方法集重合，那么**类型**实现了**接口**。
- **类型内**定义了有什么**属性**，**接口内**定义了有什么**操作**，两者产生**关联**的关键是**方法**是否都被实现。
- 接口是**隐式实现**的，不需要显式声明；接口是一种特殊的类型，它可以**被赋值成实例的指针或引用**。
- 基于以上事实，我们可以知道：
  - **不同的接口**可以完成**不同的组合操作**；
  - **多个类型**可以实现**同个接口**，**一个类型**可以实现**多个接口**；
  - **不同的类型**完成**不同的组合操作**，看起来却是**同一个接口**，这就是多态！

#### 断言

- 虽然我们提供对外提供了统一接口调用的方案，但是对内我们到底如何从接口出发辨别纷繁的类型呢？给定一个类型我们又该如何确定它是否实现了某个接口？

- **类型断言和类型选择：**给定接口确定类型

    ```go
    // 类型断言
    if _, ok := varI.(T); ok {
    // ...
    }

    // 类型选择
    switch t := varI.(type) {
    case T:
    // ...
    case nil:
    // ...
    default:
    // ...
    }
    ```

- **接口断言：**给定类型确定接口

    ```go
    // 接口断言
    if _, ok := varT.(I); ok {
    // ...
    }
    ```

### `Golang` 的多态

#### 重载

- **重载**是指根据不同的方法签名调用不同的函数实现。

- `Golang` 的设计思想中是**不允许任何形式的重载**的，这是为了强化显式化的风格。

  【PS：不同类型的接收器绑定的同名方法，严格来说不算重载】

- 尽管 `Golang` 本身不提供重载的机制，我们还是可以**借助接口**来实现类似的功能。

    ```go
    func Speak(persons ...interface{}) {
        for _, person := range persons {
            switch t := person.(type) {
            case Chinese:
                // Speak Chinese
            case American:
                // Speak English
            case nil:
                // Error handler
            default:
                // Default handler
            }
        }
    }

    func main() {
        chinese := Chinese{}
        american := american{}
        Speak(chinese) // Speak Chinese
        Speak(american) // Speak English
        Speak(chinese, american) // chinese Speak Chinese, american Speak English
    }
    ```

#### 重写和动态绑定

- **重写和动态绑定**是为了允许将子类类型的指针赋值给父类类型的指针，在运行时可以通过指向父类的指针来调用实现子类中的方法。
- 既然`Golang`中不存在严格的类和对象，重写和动态绑定的理论其实并不太适合 `Golang`，我们只需要关心怎么将**利用一个接口访问可以定位到具体的类型**就可以了。
- `Golang` 实现了编译时**静态接口判断（类型是否实现接口）**，**运行时动态类型选择（到底是哪种类型）**。

    ``` go
    type Animal interface {
        move()
        // ...
    }

    type Bird struct {
        // ...
    }

    func (b *Bird) move() {
        // fly
    }

    type Pig struct {
        // ...
    }

    func (p *Pig) move() {
        // walk
    }

    func main() {
        var animal Animal
        bird := Bird{}
        pig := Pig{}

        animal = bird
        animal.move() // fly

        animal = pig
        animal.move() // walk
    }
    ```

## **参考链接**

- [《Go语言圣经》](https://books.studygolang.com/gopl-zh/)
- [《Go入门指南》](https://learnku.com/docs/the-way-to-go)

## **致谢**

> 感谢王同学坚持不懈的“八点钟检查”以及一点都不嫌弃的“康康博客”，让我得以在快要写不下去的时候还坚持着做一些有意义的复读，XOXO。

---

