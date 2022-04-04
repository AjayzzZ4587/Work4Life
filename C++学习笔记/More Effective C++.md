# More Effective C++

## Basic

### 条款1

当你知道你需要指向某个东西，而且绝不会改变指向其他东西，或是当你实现一个操作符而其语法需求无法由**pointers**达成，你就应该选择**reference**。任何其他时候，请使用**pointers**

### 条款2

最好使用C++转型操作符

### 条款3

多态与数组不要有联系，数组中的位置在调用时多态时，由于多态的关系，内存位置不一致，还有析构方式也会构成循环

### 条款4

非必要不要提供default constructor

## 操作符

### 条款5

对定制的“类型转换函数”保持警觉，operator+类型可以实现隐式转换，还有单个构造函数可能被当作隐式转换。

### 条款6

前置与后置++ --是有区别的，后置使用了const与添加一个int 0的来做区别，返回时，避免出现a++++的情况，所以返回一个内部旧有的oldvalue。在使用选择上，应该更加偏重于前置，毕竟后置中使用int为区别可能在某种程度上影响了效率。

### 条款7

由于重载后调用顺序未知带来的麻烦，所以&&，||，，千万不要去重载它们

### 条款8

#### new

如果你希望将对象产生于heap，请使用new operator.它不但分配内存，而且为该对象调用一个constructor。如果你只是打算分配内存，请调用operator new，那就没有任何constructor会被调用。如果你打算在heap objects产生时自己决定内存分配方式，请写一个自己的operator new,并使用new operator，它将会自动调用你所写的operator new. 如果你打算在已分配（并拥有指针）的内存中构造对象，请使用placement new。

new operator 中有构造与内存分配过程，operator中只有内存的分配的过程，且返回一个指针。

#### 删除与内存释放

在这其中，new operator 与operator new 与 delete operator 与operator delete是配套使用的，应该一组有construct一个组没有，在调用时，不应该搞混。

#### 数组

数组中的new operator调用的operator new []

## 异常

### 条款9

把资源封装在对象内，通常便可以在exceptions出现时避免泄露资源。

### 条款10

对象在构造时，如果出现部分构造就出现异常的情况，deconstructor不会被调用，也就是说析构不会被调用，这样会导致内存泄露，你可以在trycatch中的catch来对数据中的东西进行处理。也可以使用，const指针加类外函数块，对这一问题进行解决。然而都不能够完美解决。最好的方案是使用智能指针。

### 条款11

禁止异常流出destructors之外，可以利用trycatch空的形式进行。,如果有一方面在调用层级会出现程序的突然中止，也会出现内存泄漏。

### 条款12

一个对象被抛出作为exception时，总会发生复制。

在抛出时，由于是静态的，所以抛出的原来的对象例如

```C++
class Widget {...};
class SpecialWidget: public Widget { ... };
void passAndThrowWidget()
{
    SpecialWidget localSpecialWidget;
    ...
    Widget& rw = localSpecailWidget;
    throw rw;
}
```

由于此时Widget调用为静态的，不是动态的，则抛出的是Widget，复制动作永远是以对象的静态类型为本。

trycatch中并不会因为类型转换的变换，也就是说，如果throw一个int类型的exception，则catch为double的是接不到int的exception的

而这一转换仅发生于两类情况。1、继承 2.有型指针转为无型指针，所以一个针对const void* 指针而设计的catch子句，可以捕捉到任何指针类型的exception。还有抛出的情况，由于抛出在继承上的问题，符合的是first fit的原则，也就是说一个派生类 的exception，可以优先被base类的catch捕捉。

有三个差异

第一：exception objects总是会被复制，如果以by value方式捕捉，他们甚至会被复制两次。至于传递给函数参数的对象则不一定得到复制。

第二：“被抛出成为exceptions”的对象，其被允许的类型转换动作，比“被传递到函数去”的对象少。

第三，catch子句以其“出现于源代码的顺序”被编译器检验对比，其中第一个匹配成功者便执行；而当我们以某对象调用一个虚函数，被选中执行的是那个“与对象类型最吻合”的函数，不论它是不是源代码所列的第一个。

###  条款13

以by reference 方式捕捉exceptions

###  条款14

不应该将templates和exception specifications混合使用。小的包大的，可能会导致程序走向unexpected，其让其走向毁灭的道路。

技术点2：如果A函数内调用了B函数，而B函数无exception specifications,那么A函数本身也不要设定exception specifications.

技术点3：处理“系统”可能抛出的exceptions，其中最常见的就是bad_alloc

### 条款15

异常处理会带来成本上的问题，当然现在基本都是标配了

## 效率

### 条款16

谨记80-20法则

### 条款17

考虑使用lazy evalution,例如类中有调用数据库的多个东西，然而每次调用这个对象会造成全部数据库连接的调用，可以利用智能指针，mutable这样的操作来实现lazy evalution。或是矩阵操作中 m3= m1 +m2。在实际调用m3[4]时，可以考虑仅仅做相关操作的值，而不计算全部矩阵，这种缓式调用的方式可以加快计算。

### 条款18

分期摊还预期的计算成本，例如将max，min这些值在每步计算中计算出来，还有动态数组的实现中，每次多分一些空间，这样后续可以减少operator new的调用。

### 条款19

产生临时对象在于当隐式类型转换，或者是函数返回对象时，会产生临时对象的情况。

在隐式转换中，如果

```c++
void uppercasify(string& str);

char subtleBOOkPlug[] ="Effective C++";
uppercasify(subtleBookPlug);   //这是错误的
```

此时如果不出错的话，大写的操作将会应用于临时对象，针对这类references-to-non-const进行隐式转换，会允许临时对象被修改，这也时为什么被禁止的原因。而Reference-to-const就没有这方面的问题。

临时对象可能很耗成本，因此应该尽可能消除它们。

### 条款20

协助完成“返回值优化（RVO）”，在这其中，在函数中返回对象的手法会产生临时对象，而C++允许编译器将临时对象优化，进而让它们不存在，这种方法可以实现对此等的优化。如果加上inline，那么就会成为最有效率的版本。这项技术叫做 return value optimization。

### 条款21

```c++
class UPint {
    public:
           UPInt();
      	   UPInt(int value);
        ...
};

UPInt upi1, upi2;
UPInt upi3 = upi1 + upi2;

//下面这两种之所以可以成功，原因便在于产生了临时对象，进而将整数10转换成为UPInts
upi3 = upi1 + 10;
upi3 = 10 + upi2;

```

这个问题可以借助重载技术避免隐式类型转换

### 条款22

复合版本则是直接将结果写入其左端自变量，不需要产生一个临时对象来放置返回值。

所以考虑以操作符符合形式(op=)取代其独身形式（op）

### 条款23

考虑使用其他程序库

### 条款24

了解virtual functions、mulitiple inheritance、virtual base classes、runtime type identification的成本

virtual functions -> vtbl,内部放一个vtbl来实现，其中调用产生于指针。

菱形继承可能让virtual base classes 可能导致对象内的隐藏指针增加。

## 技术

### 条款25

将constructor 和non-member functions虚化，利用list<NLComponent*> components的形式来实现这一需求。

利用virtual NLComponent * clone（） const = 0;的形式实现virtual copy constructor



**non-member functions虚化**：写一个虚函数做实际工作，再写一个什么都不做的非虚函数，只负责调用虚函数。

### 条款26

#### 允许零个或一个对象

利用private压制对象的诞生，再用将全局函数声明成友元函数，让其可以调用private，最后内含一个static，也就是说永远只有一个被调用。

#### 不同的对象构造状态

类中派生类与类同时产生，则产生两个不同的对象。而将其设置为private constructor后，就被禁止派生。

可以利用这一性质，在当出现需要有private constructor时，利用pesudo constructors来实现其想要允许任意数量的对象产生的需求。

#### 允许对象生生灭灭

利用pesudo consructors可以实现对任意数量对象生成的控制，极具优势。

#### 一个用来计算对象个数的Base Class

可以把计数等方面的功能全部封装了，再由private继承来引进类内进行使用。

### 条款27

要求（或禁止）对象产生于heap之中

#### 要求对象产生于heap中

将construtors设定为public，而将destructor设定为private，当然这般后续就无法内含或者继承

如果想要实现继承，就可以将destructor设定为protect，内含就可以使用指针来操作。



#### 判断某个对象是否位于Heap中

可以建立一个abstract mixin base class，来符合我们的需求，只是这一种类是不能够成为内建类的。

#### 禁止对象产生于heap之中

可以借助将operator new 和delete设定为private实现，如果后续希望能够在假设说想要继承使用的话，其实可以利用内含的形式来实现，可以就可以被创建于heap中。

### 条款28

**智能指针**

#### 实现Dereferencing Operators(解引操作符)

以返回地址的方式会比较合适

#### 测试Smart Pointers 是否为Null

如果不实现内部就调用必然会出现错误

如果以void*()的隐式类型转换的形式来实现，则会出现当利用两个不同类型来比较时，也能够通过的现象，这两种实现方式都不太合适。

还有一种是对！操作符进行重载，也符合后续操作需求。

#### 将Smart Pointers 转换为Dumb Pointers

**不要提供对dumb pointers的隐式转换操作符，除非不得已**

#### Smart Pointers和“与继承有关的”类型转换

利用templates可以实现向上转型的问题，也可以完成在const与non-const上的问题。

### 条款29 

#### 引用计数的实现

在类内加一个struct,利用它来充当那个大家一起引用的东西，假如而在赋值过程中，如果已经存在，则应当仅仅是对计算value的增加，而不是重新构造新的。

#### Copy-on-Write（写时才复制）

读与写不同，是两个不同的操作，读是const，写会对内部进行修改。应该做一个判断，当内部计数大于1时，应该先减去其值，然后重新创建空间，让我们来使用。表达的意思在于当我们需要修改成我们的版本时，再对其进行创建新的空间与赋值。

#### Pointers、Reference、以及Copy-on-Write

可能会有一个指针同时执行现有的指向，此时可以将对其设立一个flag来监视其操作。而且其实现上也简单。

#### 一个引用计数基类

将value类进行提取

#### 自动操作引用次数

利用智能指针以组合的形式来实现引用次数。

### 条款30

#### 实现二维数组

无法对不存在的operator[][]进行重载，因此只能在其中加一个代理类，先实现它的[],再实现外部的[]。这种操作也叫做proxy classes。

#### 区分operator[]的读写动作

也是利用代理来实现，区分左值可改，右值不可更改的性质。

#### 限制

假设此时出现另外一种情况时，也就是他拿去做++,+=操作时，也会带来问题。

### 条款31

怎么让函数根据一个以上的对象类型来决定如何虚化

### 虚函数+RTTI（运行时期类型辨识）

自身来去做判断，再传入其他碰撞物体，接触后转型成原本的样子，再做判断，然而带来的问题是，破坏了封装性。

#### 只使用虚函数

用虚函数+重载也可以实现这一目的，然而实际情况中，如果有这一现象，最好是修改需求。

#### 自行仿真虚函数表格

利用虚函数的实现原理来实现这一需求。

#### 使用“非成员函数”的碰撞处理函数

当碰撞时，有对象1与2，如果处理1和2的关系成为关键，采用中立函数来处理，说不定是一个好方法。
