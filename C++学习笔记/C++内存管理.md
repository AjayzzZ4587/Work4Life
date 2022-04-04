# C++内存管理

#### array new,array delete

数组中有cookie（something about bookkeeping）,在数组分配过程中没有指定大小，则在delete时不能按照应有次数对cookie中的内存进行删除，反之亦然

当然，这种是对于类而言的，在普通的int数组的delete中不需要有[],也就是说放的是object时，是依据每一个进行创建的，也就需要对每一个进行删除，才知道调用几次析构

![image-20210517080633944](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210517080633944.png)

**在析构时和创建顺序相反**

#### Placement new

![image-20210518090735457](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210518090735457.png)

![image-20210518090936603](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210518090936603.png)

**等同于调用构造函数**

#### 重载

![image-20210519075347166](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210519075347166.png)

![image-20210519075950141](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210519075950141.png)

![image-20210519080215580](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210519080215580.png)

可以重载::operator new / ::operator delete

在重载对象内存管理时，需要设置为静态的，不然无法访问

#### 重载示例

类对象

![image-20210519081157444](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210519081157444.png)

#### 重载 new()/delete()

![image-20210519082214522](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210519082214522.png)

在重载new后，delete也应该重载

只有调用构造函数才会调用自己重写的delete，抛出了异常，也就调用起对应的某个版本delete，同时结束

![image-20210519083721878](C:\Users\ajay\AppData\Roaming\Typora\typora-user-images\image-20210519083721878.png)

#### Per-class allocat，1

不要每次调用malloc，减少malloc总是好的
还能降低cookie使用次数

设计一个指针指向自己 可以降低malloc调用次数，虽然影响不太大，当是可以去掉cookie，

![image-20210520092824972](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210520092824972.png)

用链表来作为内存

![image-20210520092943792](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210520092943792.png)

将其还回头指针

#### Per-class allocat，2

![image-20210524094847914](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210524094847914.png)

#### static allocator

设计新的allocator将上述的内存分配分装起来，只要调用allocator就可以对文件进行使用

##### 调用案例

![image-20210525084257908](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210525084257908.png)

静态的还需要再外头进行定义，需要记住

![image-20210525084408977](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210525084408977.png)

先要一块内存，然后一一切割，借用嵌入式指针进行切割。

#### Macro for static allocator

![image-20210526092607774](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210526092607774.png)

![image-20210526092629106](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210526092629106.png)

能够实现复用

在MFC中有所实现 

![image-20210526093024832](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210526093024832.png)

將前述allocator進一步發展為具備16條free-lists ,並因此不再以application classes内的static呈現,而是一個globalallocator _這就是 G2.9的std:alloc的雛形。

#### New Handler

分配内存后应该检查指针是不是零，来检查是不是应用正确

**异常的抛出可能是bad_alloc或返回0**

![image-20210527091005661](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210527091005661.png)

##### new hander的作用

让更多memory可用
调用abort()或exit()

![image-20210527091249607](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210527091249607.png)

寻找可以释放的内存，释放之后再来检查是否可以创建内存

![image-20210527091641944](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210527091641944.png)

### =default，=delete

指定函数是默认版本，只有拷贝构造函数，拷贝赋值函数，析构函数有默认版本。default可以指定默认版本，当然除了这两类外还可以适应其他的函数版本

### malloc()

![image-20210531090726716](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210531090726716.png)

在小块运用时一定会占用8字节，这样会造成内存的额外开支

### VC6标准分配器之实现

VC6+的allocator只是以::operator new和::operator delete完成allocate()和deallocate()沒有任何特殊設計。

### BC5标准分配器之实现

也和vc6一样，

**区块的大小，去掉cookie**

### G2.9标准分配器之实现

2.9也是如此

![image-20210602091446643](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210602091446643.png)

并没有被引入任何容器

G2.9容器使用的分配器，不是std::allocator而是std::alloc

![image-20210602091643607](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210602091643607.png)

静态才能这样实现![image-20210602091730488](C:\Users\ajay\Desktop\OneDrive\C++学习笔记\C++内存管理.assets\image-20210602091730488.png)

### G2.9std_allocVSG4.9__pull_alloc

属于编制外成员，但于G2.9类似

### G4.9pull alloc用例

标准版的allocator也是换汤不换药

### G2.9std alloc

### G2.9std alloc运行一瞥

要使用alloc需要记住自己需要使用的空间，比如容器的给定的类型，就是给定所需空间的意义

申請32bytes，由於pool為空，故索取並
成功向pool挹注32*20 *2 + RoundUp(0>>4)（除16的操作）
= 1280，從中切出1個區塊返給客戶，19
個區塊給list#3，餘640備用
。

战备池永远为20，当新申请时，从备用池中申请 

把内存用到山穷水尽时

当内存到极点了，合并的话，难度极大

通过对半处理，最后是可以把内存都用完

