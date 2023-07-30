# C++

# 综述

- 简化构造函数、消除构造函数冗余代码的特性 8 非静态数据成员默认初始化、12 委托构造函数、13 继承构造函数

## 第八章 非静态数据成员默认初始化 C++11 C++ 20

C++ 11 以前非静态数据成员初始化需要用初始化列表，但是如果数据成员多，则构造函数不好写，因此C++11提出新的初始化方法：**声明非静态数据成员**的同时直接使用=或者{}初始化

```cpp
int a = 0;
int b{1};
```

由此，构造函数可以专注于特殊数据成员初始化。

**初始化优先级：初始化列表优先于声明时的默认初始化**

**注意：不能使用()对非静态数据成员进行初始化，不能使用auto声明和初始化非静态数据成员**

C++ 20对此特性又进行了扩充，可以对位域成员进行默认初始化

```cpp
struct S {
    int y : 8 = 11;
    int z :4 {7};
}
```

**注意：当表示位域的常量表达式是一个条件表达式的时候**

```cpp
int a;
struct S2{
    int y: true ? 8 : a = 42;
    int z : 1 || new int { 0 };
}    
```

**这段代码中并不存在默认初始化， 因为最大化识别标识符的解析规则让=42 和 {0}不可能存在于解析的顶层。于是上面的代码被解析为**

```cpp
int a;
struct S2{
    int y: (true ? 8 : a = 42);
    int z : (1 || new int { 0 });
}    
```

所以可以通过()来明确被解析的优先级

```cpp
int a;
struct S2{
    int y: (true ? 8 : a) = 42;
    int z : (1 || new int) { 0 };
}    
```

## 总结

非静态数据成员默认初始化一定程度解决了代码冗余问题，可读性更强初始化方法更简单直接。

## 第九章 列表初始化 C++11 C++20

使用括号初始化的方式是直接初始化。等于号是拷贝初始化，调用的依然是直接初始化对应的构造函数，只不过是隐式调用。如果将C (int a) 声明为explicit那么等号就会失败 C x2 = 4。new运算符和类构造函数的初始化列表就是直接初始化。而函数传参和return返回的是拷贝初始化。

使用大括号初始化是列表初始化，同样区分直接初始化和拷贝初始化。

```cpp
int x = {5};        // 拷贝初始化
int x1{8};          // 直接初始化
C X2 = {4}          // 拷贝初始化
C X3{2};            // 直接初始化
foo({8});           // 拷贝初始化
foo({"hello", 8});  // 拷贝初始化
C x4 = bar();       // 拷贝初始化
C *x5 = new C({"hi", 42}); // 直接初始化
```

支持隐式调用多参数的构造函数

STL容器之所以支持列表初始化，不仅是因为编译器支持，同时也是它们支持std::initializer_list为形参的构造函数。std::initializer_list是一个支持begin、end以及size成员函数的类模板。编译器负责将列表里的元素构造为initializer_list的对象，然后寻找标准容器中支持它的构造函数并调用它。

**注意：std::initializer_list的begin、end并不返回一个迭代器对象，而是一个常量对象指针 const T***

### 使用列表初始化的注意事项

#### 隐式缩窄转换问题

它是编写代码中稍不留意出现的，不一定引发错误，甚至可能没有警告

```cpp
int x = 12345;
char y = x;
```

如果使用列表初始化就可以避免这类现象，

```cpp
int x{12345};
char y{x};
```

MSVC和Clang就不会编译通过

哪些是隐式缩窄转换

- 从浮点类型转换为整数类型

- 从long double 转换为double或float，或从double转换为float。

- 从整数类型或非强枚举类型转换到浮点类型，除非转换源是常量表达式。

- 从整数类型或非强枚举类型转换到不能代表所有原始类型值的整数类型，除非源是一个常量表达式，其值在转换之后能够使和目标类型。

如果一个类同时拥有满足列表初始化的构造函数，且其中一个是以std::initializer_list为参数，那么优先这个。

### 指定初始化

为了提高数据成员初始化的可读性和灵活性，C++20引入指定初始化。

```cpp
struct Point {
    int x;
    int y;
}
Point p{ .x = 4, .y = 2};
```

**注意**

- 指定初始化要求对象必须是一个聚合类型即只有数据成员，

- 指定的数据成员必须是非静态数据成员，因为静态成员不属于某个对象。

- 每个非静态数据成员只能初始化一次。

- 非静态数据成员的初始化必须按照顺序初始化。

- 针对联合体的数据成员只能初始化一次，不能同时指定。

- 不能嵌套指定初始化，但是如果确实想嵌套初始化可以通过另一种方式。

- 在C++20中一旦使用嵌套初始化，就不能混用其他方法对数据成员进行初始化

- 指定初始化不能初始化数组

```cpp
struct Point3D{
    Point3D(){}
    int x;
    int y;
    int z;
}
```

上面这个就不是因为它有构造函数，可以使用直接初始化赋默认值。

```cpp
嵌套初始化
struct Line {
    Point a;
    Point b;
}
Line1 { .a {.y = 5} }
```

```cpp
指定初始化在c++中无法初始化数组，但是在C语言中可以，而拒绝的理由是与lambda表达法冲突了
int arr[3] = { [1] = 5 };
```

### 总结

它解决了容器初始化复杂的问题，使自定义容器支持列表初始化变得容易。C++20引入指定初始化一定程度简化了复杂聚合类型初始化工作，让初始化复杂聚合类型的代码变得容易。

## 第十章 默认和删除函数 C++11

编译器会默认添加六种成员函数，如果没有的话

1. 默认构造函数

2. 析构函数

3. 复制构造函数

4. 复制赋值运算符函数

5. 移动构造函数 C++11

6. 移动赋值运算符函数 C++11

简化代码编写的同时也会有一些麻烦：

1. 声明任何构造函数都会抑制默认构造函数的添加

2. 一旦用自定义构造函数代替默认构造函数，类就将转变为非平凡类型

3. 没有明确的办法彻底禁止特殊成员函数的生成（C++11之前）

如果想要禁止类对象的复制，C++11之前可以将复制构造函数和复制赋值运算符函数声明为private并且不予实现，但是这样的代码仍能通过编译，但却在链接的时候在报错。using说明符无法将基类的私有成员引入到子类中。在C++11之后添加了显示默认和显示删除解决上述问题。

### 显示默认和显示删除

在函数末尾添加=default和=delete。=dexfault既可以在类内部也可以在类外部声明。而=delete只能在类内部声明，如果在外部将会引起编译错误。

=delete从编译层面抑制了函数的生成。

### 显示删除的其他作用

显示删除不仅适用于类的成员函数，同样适用于普通函数。但是意义不是很大。

显示删除类的析构函数某种程度上和删除new运算符的目的正好相反，它阻止类通过自动变量、静态变量或者全局变量的方式创建对象，但却可以通过new的方式创建。因为删除析构之后，类无法析构。所以那些会隐式调用析构函数的对象就无法创建了，当然通过new运算符创建的对象也无法通过delete销毁了。所以这样的用法并不多见，一般只在单例模式中出现。

### explicit和=delete

在类的构造函数上同时使用explicit和=delete是不好的。它会导致代码行为混乱，难以理解。

### 总结

C++11引入显示默认和显示删除，是我们可以精确的控制类特殊成员函数的生成以及删除。

## 第十一章 非受限联合类型 C++11

### 联合类型在C++中的局限性

过去的C++规定联合类型的成员变量不可以是非平凡类型。

### 使用非受限联合类型 C++11

在C++11中去除了这种限制，但是引入了另一个问题，如何精确初始化联合类型成员对象。在C++11中如果有联合类型存在非平凡类型，那么这个联合类型的特殊成员函数将被隐式删除，也就是说必须自己至少提供联合类型的构造函数和析构函数。

```cpp
union U
{
    U() { }        // 存在非平凡类型成员，必须提供构造函数
    ~U() { }       // 存在非平凡类型成员，必须提供析构函数
    int x1;
    float x2;
    std::string x3;
    std::vector<int> x4;
}
```

尽管可以通过编译了但是还是运行会出错，因为构造和析构函数什么也没做，做一下修改

```cpp
union U
{
    U(): x3() { }        // 存在非平凡类型成员，必须提供构造函数
    ~U() { x3.!basic_string() }       // 存在非平凡类型成员，必须提供析构函数
    int x1;
    float x2;
    std::string x3;
    std::vector<int> x4;
}
```

使用空括号初始化x3，调用了string的默认构造函数，析构函数中手动调用了string的析构函数。

但是无法知道那个联合成员会被调用，因此推荐构造函数和析构函数为空，在调用处实现构造和析构

```cpp
```cpp
union U
{
    U() { }      
    ~U() { }     
    int x1;
    float x2;
    std::string x3;
    std::vector<int> x4;
}

int main(){
    U u;
    new(&u.x3) std::string("hello world");
    u.x3.!basic_string();

    new(&u.x4) std::vector<int>;
    u.x4.push_back(58);
    std::cout << u.x4[0] << std::endl;
    u.x4.~vector();
}
```

```
**注意：上面的代码使用了placement new的技巧来初始化构造x3和x4，在使用完之后调用对应的析构函数**

非受限联合类型对静态成员变量的支持。联合类型的静态成员变量不属于联合类型的任何对象，所以并不是对象构造时定义的，不能在联合类型的内部初始化。实际上这一点和类的静态成员变量是一样的。他的初始化方法和类的静态成员变量一致。即外部初始化

### 总结

尽管对于现在16GB内存的PC来说，内存可能并不需要太过于担心，但是C++11对非受限联合类型的修改，表达了C++对其设计理念的坚持，当然在一些生产环境中仍然需要节省内存。如果支持C++17，则大部分情况下将使用std::variant来代替联合体。



## 第十二章 委托构造函数 C++11

### 冗余的构造函数

多次调用相同的代码。为了简化代码，如果将成员初始化放到这个相同的函数中的话，还可能造成性能损失。过去的C++没有提供一种复用相同类型构造函数的方法，也就是说无法让一个构造函数将初始化的一部分工作委托给同类型的另外的构造函数。这导致程序员不得不编写重复繁琐的代码。

### 委托构造函数

执行顺序是先执行代理构造函数的初始化列表，代理构造函数主体，最后委托构造函数的主体。

需要注意以下5点

1. 每个构造函数都可以委托另一个构造函数为代理，也就是说，每个构造函数既可能是委托构造函数也可能是代理构造函数。

2. 不要递归循环委托，因为不会被编译器报错，随之而来的是程序运行时发生未定义行为。最常见的是栈空间被耗尽。

3. 如果一个构造函数是委托构造函数，那么其初始化列表中就不能对数据成员和基类初始化。

4. 执行顺序是先执行代理构造函数的初始化列表，代理构造函数主体，最后委托构造函数的主体

5. 如果代理构造函数执行完成后，委托构造函数出现了异常，则自动调用该类型的析构函数。

### 委托模板构造函数

委托模板构造函数是指一个构造函数将控制权委托到同类型的一个模板构造函数，简单说，就是代理构造函数是一个函数模板。意义是泛化了构造函数，减少了冗余代码。

### 捕获委托构造函数的异常

当使用Function-try-block去捕获委托构造函数异常的时候，其过程和捕获初始化列表异常如出一辙。如果一个异常在代理构造函数的初始化列表或主体中抛出，那么委托构造函数主体不再执行，控制权将会教导catch代码块中。链式调用以相反顺序抛出异常

### 委托参数较少的构造函数

通常情况下将参数较少的构造函数委托给参数较多的构造函数。因为这样自由度更高，但从多到少也是有意义的，比如fstream。

### 总结

委托构造函数是减少代码冗余的重要手段之一，也是最重要的方法。



## 第十三章 继承构造函数 C++11

### 继承关系中构造函数的困局

当一个基类有很多构造函数，且在未来需要对其进行派生的时候，需要定义同样多的构造函数，而目的仅仅是转发。这样会导致大量的代码冗余且容易出错。因此需要让编译器完成这样的简单重复的工作。

### 使用继承构造函数

由于C++可以使用using将基类的函数引入到子类，因此在C++11中对using关键字的能力进行扩展，使其可以引入基类的构造函数。

```cpp
class Base {
    public:
        Base(){}
        Base(int a) {}
        ....
}

class Derived : public Base {
    public:
        using Base::Base;
}
```

**但是还需要注意六条规则**

1. **派生类是隐式继承基类的构造函数，所以只有在程序中使用了这些构造函数，编译器才会为派生类生成继承构造函数的代码**

2. **派生类不会继承基类的默认构造函数和复制构造函数**

3. **继承构造函数不会影响派生类默认构造函数的隐式声明**

4. **在派生类中声明签名相同的构造函数会禁止继承相应的构造函数**

5. **派生类继承多个签名相同的构造函数会导致编译失败** 多继承时出现了二义性

6. **继承构造函数的基类构造函数不能为私有**

### 总结

该特性让派生类能够直截了当的使用基类构造函数。

## 第十四章 强枚举类型 C++11 C++17 C++20

### 枚举类型弊端

enum破坏了C++的类型安全，因为枚举类型可以提升为整数类型，且不同枚举类型之间可以比较，因为它可以被提升为整数类型。其次是枚举类型的作用域，枚举类型可以把其内部的枚举标识符导出到枚举被定义的作用域，这导致的命名冲突。

对于以上问题可以通过将enum封装为类私有数据成员

优点：

保证外界无法访问。只能通过对应的常量静态对象访问。

可以通过重载运算符来实现枚举类型之间的运算

缺点：

要敲很多代码

枚举类型本身是POD类型而类破坏了这种特性。

### 强枚举类型

由于枚举类型确实存在一些类型安全问题，因此在C++11中对其进行了重大升级，另外为了兼容性也保持了之前的特性。强枚举类型具备以下3个新特性

1. 枚举标识符属于强枚举类型的作用域

2. 枚举标识符不会隐式转换为整型

3. 能指定强枚举类型的底层类型，底层类型默认为int类型

定义强枚举类型的方法是使用enum class 可以使用 enum class T : **unsigned int {}**来指定底层类型。利用它可以消除不同编译器带来的歧义性 。 对于枚举类型同样可以指定。

### 列表初始化有底层类型枚举对象

从C++17开始，对有底层类型的枚举类型对象可以直接使用列表初始化。

### 使用using打开强枚举类型

C++20标准扩展了using功能，他可以打开强枚举类型的命名空间。在一些情况下，这样做会让代码更简洁。

### 总结

强枚举类型不仅修正了枚举类型的缺点并且全面的扩展了枚举类型的特性。

## 第十五章 扩展的聚合类型 C++17 C++20

### 聚合类型的新定义

C++17标准对聚合类型的定义做出了大幅修改，即从基类公开且非虚继承的类也可能是一个聚合。同时聚合类型还需要满足常规条件。

1. 没有用户提供的构造函数

2. 没有私有和受保护的非静态数据成员

3. 没有虚函数

在新的扩展中，如果类存在继承关系，则额外满足以下条件

1. 必须是公开的基类，不能是私有或者受保护的基类

2. 必须是非虚继承

派生类是否是聚合类型与基类没有关系

### 聚合类型的初始化

由于聚合类型定义的扩展，聚合对象的初始化方法也发生了变化。过去想要初始化派生类的基类，需要在派生类中提供构造函数。而现在由于聚合类型的扩展可以进行简化

1. 删除派生类中用户提供的构造函数

2. 直接初始化

删除用户提供的构造函数使类称为聚合类型，然后使用大括号直接初始化。

### 扩展聚合类型的兼容问题

以前不是聚合类型的在C++17可能是聚合类型，这导致了一些问题。

### 禁止聚合类型使用用户声明的构造函数

## 第十六章 override和final说明符 C++11

### 重写、重载和隐藏

重写、重载和隐藏是三个完全不同的概念

1. 重写的意思更接近覆盖

2. 重载，通常指在同一个类中有两个或两个以上的函数，他们的函数名相同，函数签名不同

3. 隐藏，指基类成员函数，无论它是否为虚函数，当派生类出现同名函数时，如果派生类函数签名不同于基类函数，则基类函数会被隐藏。另外，如果还想使用基类函数可以使用using关键字引入

### 重写引发的问题

重写基类虚函数的时候有时候可能会容易写错函数名，而这样的编码错误不会编译报错，直到运行的时候才会发现。

### 使用override说明符

说明符放在虚函数末尾，明确告诉编译器，这个虚函数是覆盖了基类的虚函数。

### 使用final说明符

阻止派生类去继承基类的虚函数。同样声明在尾部。

有时候final和override会同时出现，表示这个函数是派生类继承自基类，但是不希望其后的派生类继续修改。

final同时还可以修饰类，表示这个类不希望作为基类被继承。

### 总结

override和final避免了因粗心大意而造成的错误

## 第十七章 C++11 C++17 C++20

### 繁琐的容器遍历

C++11以前容器的遍历十分繁琐

### 基于范围的for循环语法

C++11引入基于范围的for循环特性，即引号遍历。一般使用const auto &而不用auto，防止调用赋值构造函数。

### begin和end函数不必返回相同类型

C++11的for展开时

```cpp
auto && __range = range_expression;
for(auto __begin = begin_expr, __end = end_expr;__begin!=__end;__begin++){...}
```

而C++17中将__begin和__end的声明分开auto __begin auto __end因此begin和end不必返回相同类型了。

### 临时范围表达式的陷阱

无论C++11还是C++17，基于范围的for循环伪代码都是由

```cpp
auto && __range = range_expression;
```

开始的。而这里有一个陷阱auto &&。如果range__expression是一个纯右值，那么右值引用会扩展其生命周期，保证整个for循环过程中的访问安全性。但如果它是一个泛左值，那结果就是不确定的。

```cpp
class T {
    vector<int> data_;
public:
    vector<int>& items() {return data_;}
};
T foo(){
    T t;
    return t;
}
for (auto & : foo().items(){} // 未定义的行为
```

因为foo.items()返回的是一个泛左值类型，于是右值引用就无法扩展其生命周期，导致for循环发生未定义的行为。

```cpp
T thing = foo();
for (auto & item: thing.items()){...}
```

将数据复制出来是一种解决办法。

在C++20中，增加了对初始化语句的支持,可以将代码简化

```cpp
for (T thing = foo();auto & x : thing.items()){...}
```

### 实现一个支持基于范围的for循环的类

满足以下条件

1. 该类型必须由一组和其类型相关的begin和end函数，它们可以是类型的成员函数，也可以是独立函数

2. begin和end需要返回一组类似迭代器的对象，并且这组对象必须支持operator *、operator != 和operator ++运算符函数。 

### 总结

基于范围的for解决了遍历容器繁琐的问题。使用时需要注意临时范围表达式的生命周期。

## 第十八章 支持初始化语句的if和switch C++17

### 支持初始化语句的if

生命周期往后延申直至if-else结束。if和else if中可以初始化

### 支持初始化语句的switch

switch后可以初始化，生命周期贯穿整个switch块。

### 总结

带初始化语句的if和switch其实就是语法糖，可以被等价替换。增加了代码可读性和可维护性。

## 第十九章 static_assert声明

### 运行时断言

在静态断言出现之前，使用的是动态断言，即运行到这里时才触发断言。

### 静态断言的需求

如果想在模板实例化的时候对模板实参进行约束，运行时断言是做不到的。尽管可以采取一些方法模拟静态断言，但都是有缺陷的。

### 静态断言

用于在程序编译阶段评估常量表达式并对返回false的表达式断言。

1. 所有处理在编译期执行

2. 简单的语法

3. 断言失败可以显示丰富的错误诊断信息

4. 可以在命名空间、类或代码块内使用

5. 失败的断言会在编译阶段报错

使用static_assert需要传入两个实参：常量表达式和诊断消息字符串。**第一个实参必须是常量表达式，因为编译器无法计算运行时才能确定结果的表达式。**

### 单参数stati_assert

让常量表达式作为错误诊断信息字符串，C++17中引入，GCC指定C++11也支持，而MSVC必须指定C++17。

### 总结

C++11之前boost、loki等代码库都实现了静态断言，在C++11以及后续的C++17中引入了static_assert完美满足了静态断言的各种需求

## 第二十章 结构化绑定 C++17 C++20

### 使用结构化绑定

在python中可以返回多个值（元组中），而C++11也引入了元组，可以通过元组返回多个值，但使用起来不如python简洁。

引入结构化绑定

```cpp
#include <iostream>
#include <tuple>

auto return_multiple_values(){
    return std::make_tuple(11, 7);
}

int main() {
    auto [x, y] = return_multiple_values();
    std::cout << x << y << endl;
}
```

使用C++17标准编译这段代码，可以得到正确输出。右边的表达式不必必须是函数返回的结果，它可以是任意一个合理的表达式。比如一个结构体。会按照结构体内元素的顺序赋值，可以是不同类型。

真实情况是：在结构化绑定中编译器会根据限定符生成一个等号右边对象的匿名副本，而绑定的对象正是这个副本而非原对象本身。另外，这里的别名真的是单纯的别名，别名的类型和绑定目标对象的子对象类型相同。

如果结构化绑定声明为const auto&[x, y] = bt,那么x=11是不被允许的，因为x是const引用，而bt.b = "other string";是OK的，因为bt本身没有const限制。

同时，结构化绑定的别名无法再同一个作用域中重复使用

auto[x, ignore] = t;

auto [y, ingore] = t; //编译失败，命名重复。

### 结构化绑定的三种类型

结构化绑定可以作用于三种类型，包括**原生数组，结构体和类对象，元组和类元组的对象**

#### 绑定到原生数组

个数一致，编译器必须知道原生数组的元素个数，一旦退化为指针，就将失去这个属性。

#### 绑定到结构体和类对象

将标识符列表中的别名，分别绑定到结构体和类的非静态成员变量上。首先个数必须相同，其次必须是公有的（C++20修改了此项规则）；然后这些成员必须是在一个类或者基类中；最后，绑定的类或结构体中不能存在匿名联合体。

#### 绑定到元组和类元组的对象

绑定到元组就是将标识符列表中的别名分别绑定到元组对象的各个元素。

实际上绑定元组和类元组有一系列抽象的条件：对于元组或者类元组类型T：

1. 需要满足std::tuple_size<T>::value是一个符合语法的表达式，并且该表达式获得的整数值与标识符列表中的别名个数相同。

2. 类型T还需要保证std::tuple_element<i,T>::type 也是一个符合语法的表达式，其中i是小于std::tuple_size::value的整数，表达式代表了类型T中第i个元素的类型。

3. 类型T必须存在合法的成员函数模板get<i>()或者函数模板get<i>(t)，其中i是小于std::tuple_size::value的整数，t是T的实例，get<i>()和get<i>(t)是返回t中第i个元素的值。

只要满足上述条件的就可以被结构化绑定，比如pair，array，或者自定义的实现了上述几个函数的特化或偏特化即可。

### 实现一个类元组类型

```cpp
#include <iostream>
#include <tuple>

class BindBase3 {
    public:
        int a{42};
};

class BindTest3 : public BindBase3 {
    public:
        double b = 11.7;
};

namespace std {
    template<>
    struct tuple_size<BindTest3> {
        static constexpr size_t value = 2;
    };

    template<>
    struct tuple_element<0, BindTest3>{
        using type = int;
    };

    template<>
    struct tuple_element<1, BindTest3> {
        using type = double;
    };
}

template<std::size_t Idx>
auto& get(BindTest3 &bt) = delete;    // 告诉编译器不要生成除了特化的版本以外的任何函数实例。

template<>
auto& get<0>(BindTest3 &bt) { return bt.a; }


template<>
auto& get<1>(BindTest3 &bt) { return bt.b; }


int main(){
    BindTest3 bt3;
    auto& [x3, y3] = bt3;
    x3 = 78;
    std::cout << bt3.a << std::endl;
}
```