# 转换函数(conversion function)
  






指的是只有一个实参就可以
+ parameter
参数
+ statement
语句的意思
```
class Fraction {
public:
    Fraction (int num, int den = 1) : m_numerator(num), m_denominator(den) {}
    Fraction operator+(const Fraction &f) {
        return Fraction(...);
    }
private:
    int m_numerator;
    int m_denominator;
};
Fraction f(3, 5);
// 调用non-explicit ctor将4转为Fraction(4, 1)
// 然后调用operator+
Fraction d2 = f+4;
```

```
class Fraction {
public:
    Fraction (int num, int den = 1) : m_numerator(num), m_denominator(den) {}
    operator double() const {return double(m_numerator / m_denominator);}
    Fraction operator+(const Fraction &f) {
        return Fraction(...);
    }
private:
    int m_numerator;
    int m_denominator;
};
Fraction f(3, 5);
// [Error] ambiguous
// 既可以将f转换为double，也可以将4转换为Fraction
Fraction d2 = f+4;
```
```
class Fraction {
public:
    explicit Fraction (int num, int den = 1) : m_numerator(num), m_denominator(den) {}
    operator double() const {return double(m_numerator / m_denominator);}
    Fraction operator+(const Fraction &f) {
        return Fraction(...);
    }
private:
    int m_numerator;
    int m_denominator;
};
Fraction f(3, 5);
// [Error] conversion from 'double' to 'Fraction' requested
Fraction d2 = f+4;
```
# reference
+ 引用没有自己的地址<br>
Reference don't have their own address. The reference itself isn't an object(it has on identity; taking the address of a reference gives you the address of the reference; remember: the reference id its referent).
```
int x = 0;
// p is a point to x
int* p = &x;
// r is a reference to x
int& r = x;

// object和其reference的大小相同，地址也相同（全都是假象）
sizeof(r) = sizeof(x);
&x = &r;
```
+ 引用是一种漂亮的指针
+ reference的常见用途<br>
reference通常不用于声明变量，而用于参数类型(parameters type)和返回类型(return type)的描述
```
// 下面两个函数被视为具有相同签名(same signature)，所以不能同时存在
double imag(const double& im) {}
double imag(const double  im) {}

// 函数二和三可以同时存在，即const是函数签名的一部分
double imag(const double im) const {}
```

# 智能指针(pointer-like classes)
```
template<class T>
class shared_ptr {
public:
    T& operator*() const {return *px;}
    T* operator->() const {return px;}

    shared_ptr(T* p) : px(p) {}

private:
    T* px;
    long* pn;
...
}
```

# 迭代器(pointer-like classes)
```
template<class T>
struct __list_node {
    void* prev;
    void* next;
    T data;
};
template<class T, class Ref, class Ptr>
struct __list_iterator {
    typedef __list_iterator<T, Ref, Ptr> self;
    typedef Ptr pointer;
    typedef Ref reference;
    typedef __list_node<T>* link_type;
    link_type node;

    bool operator==(const self& x) const {return node == x.node;}
    bool operator!=(const self& x) const {return node != x.node;}
    reference operator*() const { return (*node).data;}
    pointer operator->() const {return &(operator*());}
    self& operator++() {node = (link_type)((*node).next); return *this;}
    self operator++(int) {self tmp = *this; ++*this; return tmp;}
    self& operator--() {node = (link_type)((*node).prev); return *this;}
    self operator--(int) {self tmp = *this; --*this; return tmp;}
};
```

# 仿函数(function-like classed)
```
template<class T>
struct identity {
    const T& operator()(const T& x) const {return x;}
};

template<class Pair>
struct select1st {
    const typename Pair::first_type& operator()(const Pair& x) const {return x.first;}
};

template<class Pair>
struct select2nd {
    const typename Pair::second_type& operator()(const Pair& x) const {return x.second;}
};

template<class T1, class T2>
struct pair {
    T1 first;
    T2 second;
    pair() : first(T1()), second(T2()) {}
    pair(const T1& a, const T2& b) : first(a), second(b) {}
    ...
};

template <class Arg, class Result>
struct unary_function {
    typedef Arg argument_type;
    typedef Result result_type;
};

template <class Arg1, class Arg2, class Result>
struct binary_function {
    typedef Arg1 first_argument_type;
    typedef Arg2 second_argument_type;
    typedef Result result_type;
};
```

# 命名空间(namespace)
```
using namespace std;

#include <iostream>
#include <memory> // shared_ptr
namespace hh01 {
void test_member_template() {
    ...
}
} // namespace

#include <iostream>
#include <list>
namespace hh02 {
template <typename T>
using Lst = list<T, allocator<T>>;
void test_template_template_param() {
    ...
}
} // namespace

int main() {
    hh01::test_member_template();
    hh02::test_template_template_param();
}
```

# 类模板
```
template <typename T>
class complex {
public:
    complex(T r = 0, T i = 0) : re(r), im(i) {}
    complex& operator+=(const complex&);
    T real() const {return re;}
    T imag() const {return im;}
private:
    T re;
    T im;
    friend complex& __doapl (complex*, const complex&);
};

{
    complex<double> c1(2.5, 1.5);
    complex<int> c2(2, 6);
    ...
}
```

# 函数模板(function template)
```
class stone {
public:
    stone(int w, int h, int we) : _w(w), _h(h), _weight(we) {}

    bool operator<(const stone& rhs) const { return _weight < rhs._weight;}

private:
    int _w, _h, _weight;
}

template <class T>
inline const T& min(const T& a, const T& b) {return b < a ? b : a;}

// 编译器会对函数模板进行实参推导(argument deduction)
stone r1(2, 3), r2(3, 3), r3;
r3 = min(r1, r2);
```

# 成员模板(member template)
```
template <class T1, class T2>
struct pair {
    typedef T1 first_type;
    typedef T2 second_type;

    T1 first;
    T2 second;

    pair() : first(T1()), second(T2()) {}
    pair(const T1& a, const T2& b) : first(a), second(b) {}

    template <class U1, class U2>
    pair(const pair<U1, U2>& p) : first(p.first), second(p.second) {}
};

class Base1 {};
class Derived1 : public Base1 {};
class Base2 {};
class Derived2 : public Base2 {};
pair<Derived1, Derived2> p;
pair<Base1, Base2> p2(p);

pair<Base1, Base2> p2(pair<Derived1, Derived2>());
// 把一个有鲫鱼和麻雀构成的pair放入一个由鱼类和鸟类构成的pair中，是可以的，反之则不行

template <typename _Tp>
class shared_ptr : public __shared_ptr<_Tp> {
    ...
    template <typename _Tp1>
    explicit shared_ptr(_Tp1* __p) : __shared_ptr<_Tp>(__p) {}
    ...
}

// up-cast
Base1* ptr = new Derived1;
// 模拟up-cast
shared_ptr<Base1> sprt(new Derived1);
```

# 模板特化
+ 特化的反面就是泛化
泛化就是模板，使用时再指定
```
template <class Key>
struct hash {};

// 因为key被绑定为char，所以<>里面为空
template <>
// 改行显示绑定内容
struct hash<char> {
    size_t operator()(char x) const {return x;}
}

template <>
struct hash<int> {
    size_t operator()(int x) const {return x;}
}

template <>
struct hash<long> {
    size_t operator()(long x) const {return x;}
}

// 类型名后面跟小括号，创建临时对象
// 临时对象再调用函数调用符
cout << hash<long>()(1000);
```

+ 模板偏特化--个数的偏(partial specialization)
泛化又叫全泛化(full specialization)
```
// 两个模板参数
template <typename T, typename Alloc=...>
class vector {
    ...
};

template <typename Alloc=...>
class vector<bool, Alloc> {
    ...
};
```

+ 模板偏特化--范围的偏(partial specialization)
```
template <typename T>
class C {
    ...
};

template <typename T>
class C<T*> {
    ...
};

template <typename U>
class C<U*> {
    ...
};

C<string> obj1;
C<string*> obj2;
```

# 模板模板参数(template template parameter)
```
template <typename T,
          template <typename T>
          class Container>
class XCls {
private:
    Container<T> c;
public:
    ...
};

template <typename T>
using Lst = list<T, allocator<T>>

// 下面一行时错误的
XCls<string, list> mylst1;
// 下面一行可以编译通过
XCls<string, Lst> mylst2;

template <typename T,
          template <typename T>
          // 此处的class不可以用typename替换
          class SmartPtr>
class XCls {
private:
    SmartPtr<T> sp;
public:
    XCls() : sp(new T) {}
};

// 下面两行报错
XCls<double, unique_ptr> p2;
XCls<int, weak_ptr> p3;
// 下面两行可以运行
XCls<string, shared_ptr> p1;
XCls<long, auto_ptr> p4;
```

# C++标准库
算法+数据结构=程序
Algorithms + Data Structures = Programs
+ 容器(containers)
+ 迭代器(iterators)
+ 算法(algorithms)
+ 仿函数(functors)
```
#define __cplusplus 201103L

#define __cplusplus 199711L

#include "stdafx.h"
#include <iostream>

using namespace std;

int main() {
    cout << __cplusplus << endl;
    return 0;
}
```

# 数量不定的参数模板(varidaic templates)
```
void print() {}

template <typename T, typename... Types>
void print(const T& firstArg, const Types&... args) {
    cout << firstArg << endl;
    print(args...);
}

// inside variadic templates, sizeof...(args) yields the number of arguments
```
...就是一个所谓的包(pack)，可以是模板参数包(template parameters pack)，可以是函数参数类型包(function parameter types pack)，可以是函数参数包(function parameters pack)

# auto
```
list<string> c;
...
list<string>::iterator ite;
ite = find(c.begin(), c.end(), target);

list<string> c;
...
auto ite = find(c.begin(), c.end(), target);

// 下面一行是错误的，没有赋值，无法推导类型
auto ite;
```
# 范围for循环(ranged-base for)
```
for (decl : coll) {
    statement
}

for (int i : {2, 3, 6, 8}) {
    cout << i << endl;
}

vector<double> vec;
...
for (auto elem : vec) {
    cout << elem << endl;
}

for (auto& elem : vec) {
    elem *= 3;
}
```

# 继承关系下的构造和析构(inheritance)
子类对象里面有父类的成分
+ 构造由内而外
Derived的构造函数首先调用Base的default构造函数，然后才执行自己
`Derived::Derived(...) : Base() {...};`
+ 析构由外而内
Derived的析构函数首先执行自己，然后才调用Base的析构函数
`Derived::~Derived(...) {... ~Base()};`

# 复合关系下的构造和析构
+ 构造由内而外
Container的构造函数首先调用Component的default构造函数，然后才执行自己
`Container::Container(...) Component() {...};`
+ 析构由外而内
Container的析构函数首先执行自己，然后才调用Component的析构函数
`Container::~Container(...) {... ~Component()};`

# 继承+复合关系下的构造和析构(inheritance+composition)
+ 构造由内而外
Derived的构造函数首先调用Base的default构造函数，然后调用Component的default构造函数，最后才执行自己
`Derived::Derived(...) : Base(), Component() {...};`
+ 析构由外而内
Derived的析构函数首先执行自己，然后调用Component的析构函数，最后调用Base的析构函数
`Derived::~Derived {... ~Component(), ~Base()};`

# 虚指针和虚表(vptr, vtbl)
代码层次看不到虚指针和虚表，如果只有继承关系而没有虚函数，作用**不大**<br>
继承函数继承的是**调用权**，不是函数所占用的内存<br>
动态调用需要满足的条件：
+ 必须通过指针调用
+ 指针必须时向上转型up-cast
+ 调用的时虚函数
动态绑定又称为多态，故多态、虚函数、动态绑定指的是同一件事情
普通函数静态绑定`call(xxx)`，通过**地址**调用函数
```
// 编译器在编译代码时，检查虚函数位置，确认n的值
// 头脑风暴，调用小猫叫函数时，叫这个动作的位置已经是确定的
// 如果确定是1，调用小狗叫时，这个位置也是1
// 即叫这个动过在整个继承关系中，虚表中位置都是1
// 子类中被重写的虚函数放到父类中同样位置
// 函数在虚表中排列位置则按照首次定义顺序
(*(p->vptr)[n])(p);
// 或者下面一行
(* p->vptr[n])(p);
```
```
class A {
public:
    virtual void vfunc1();
    virtual void vfunc2();
    void func1();
    void func2();
private:
    int m_data1, m_data2;
};

class B : public A {
public:
    virtual void vfunc1();
    void func2();
private:
    int m_data3;
};

class C : public B {
public:
    virtual void vfunc1();
    void func2();
private:
    int m_data1, m_data4;
};
```
|a(A object)|b(B object)|c(C object)|
|:---:|:---:|:---:|
|0x409004|0x409014|0x409024|
|m_data1|m_data1|classA::m_data1|
|m_data2|m_data2|m_data2|
||m_data3|m_data3|
|||m_data1||
|||m_data4||
|A::func1|B::func2|C::func2|
|A::func2|||
|A::vfunc1(0x401ED0)|B::vfunc1(0x401F80)|C::vfunc1(0x401FF0)|
|A::vfunc2(0x401F10)|A::vfunc2(0x401F10)|A::vfunc2(0x401F10)|

|non-virtual<br>member function|virtual function|
|:---:|:---:|
|A::func1|A::vfunc1|
|A::func2|A::vfunc2|
|B::func2|B::vfunc1|
|C::func2|C::vfunc1|

|A vtbl|B vtbl|C vtbl|
|:---:|:---:|:---:|
|0x401ED0|0x401F80|0x401FF0|
|0x401F10|0x401F10|0x401F10|
|0|0|0|

只能放入指向父类的指针<br>
$A\\\uparrow\\B\\\uparrow\\C$<br>
`list<A*> myLst;`

# 关于this指针
通过对象调用函数，对象的地址就是this指针<br>
所有的成员函数一定有隐藏的this指针
```
// Application framework
CDocument::OnFileOpen() {
    ...
    // this->Serialize();
    // 动态绑定(Dynamic Binding)
    //(*(this->vptr)[n])(this);
    Serialize();
    ...
}

virtual Serialize();
```
```
class CMyDoc : public CDocument {
    virtual Serialize() {
        // 定义具体功能
        ...
    }
};

int main() {
    CMyDoc mydoc;
    ...
    // CDocument::OnFileOpen(&mydoc);
    // (&mydoc)就是this
    mydoc.OnFileOpen();
}
```

# 关于动态绑定(dynamic binding)
$A\\\uparrow\\B\\\uparrow\\C$
```
B b;
A a = (A)b;
// 通过对象调用，不是指针
// 静态绑定
a.vfunc1();

A* pa = new B;
pa->vfunc1();

pa = &b;
pa->vfunc1();
```
$\downarrow$
```
118: B b;
00401CDE lea    ecx, [b]
00401CE1 call   @ILT+535(B::B) (0040121c)
119: A a = (A)b;
00401CE6 lea    eax, [b]
00401CE9 push   eax
00401CEA lea    ecx, [ebp-114h]
00401CF0 call   @ILT+830(A::A) (00401343)
00401CF5 push   eax
00401CF6 lea    ecx, [a]
00401CFC call   @ILT+830(A::A) (00401343)
120: a.vfunc1();
00401D01 lea    ecx, [a]
00401D07 call   @ILT+420(A::vfunc1) (004011a9)
123: pa->vfunc1();
00401D68 mov    eax, dword ptr [pa]
00401D6E mov    edx, dword ptr [eax]
00401D70 mov    esi, esp
00401D72 call   dword ptr [edx]
00401D7A cmp    esi, esp
00401D7C call   __chkesp (00423590)
124:
125: pa = &b;
00401D81 lea    eax, [b]
00401D84 mov    dword ptr [pa], eax
126: pa->vfunc1();
00401D8A mov    ecx, dword ptr [pa]
00401D90 mov    edx, dword ptr [ecx]
00401D92 mov    esi, esp
00401D94 mov    ecx, dword ptr [pa]
00401D9A call   dword ptr [edx]
00401D9C cmp    esi, esp
00401D9E call   __chesp (00423590)
127:
```

# const
+ 常量成员函数(const member functions)
只能放在成员函数后面，表示该函数不改变成员变量，只读取变量，不改变
+ const属于签名的一部分
字符串底层时可以共享的
```
class complex {
public:
    complex(double r = 0, double i = 0) : re(r), im(i) {}
    complex& operator+=(const complex&);
    double real() const {return re;}
    double imag() const {return im;}
private:
    double re, im;

    friend complex& __doapl(complex*, const complex&);
};

{
    complex c1(2, 1);
    cout << c1.real();
    cout << c1.imag();
}

{
    const complex c2(3, 2);
    cout << c2.real();
    cout << c2.imag();
}
```
||const object<br>(data members不得改变)|non-const object<br>(data members可变动)|
|:---:|:---:|:---:|
|const member functions<br>(保证不更改data members)|$\checkmark$|$\checkmark$|
|non-const member functions(不保证data members不变)|$\times$|$\checkmark$|

如果当初设计string::print()时未指明const，那么下面便由const object调用non-const member function，会报错。<br>
non-const member function可以调用const member functions，反之则不行。
`const string str("hello world");\\str.print();`

```
// COW: Copy On Write
class template std::basic_string<...>
charT operator[](size_type pos) const {.../*不必考虑COW*/}

reference operator[](size_type pos) {.../*必须考虑COW/*}
```

# 关于new, delete
这里的new，delete叫做表达式，分解之后的叫operator new, operator delete<br>
表达式行为不能改变，不能重载，operator new和operator delete可以重载
+ new
new: 先分配内存(memory)，再调用构造函数(ctor)
`string* ps = new string("hello");`
`void* mem = operator new(); // 内部调用malloc(n)`
+ delete
delete: 先调用dtor，再释放内存(memory)
`delete ps;`
`operator delete(ps); // 内部调用free(ps)`
+ array new和array delete
array new一定要搭配array delete
```
string* p = new string[6];
...
delete[] p; // 唤起6次dtor
delete p; // 唤起1次dtor
```

# 重载operator new和operator delete
这些影响非常大
+ ::operator new
+ ::operator new[]
+ ::operator delete
+ ::operator delete[]

```
void* myAlloc(size_t size) {
    return malloc(size);
}

void myFree(void* ptr) {
    return free(ptr);
}
```

不能被声明于一个namespace内
```
inline void* operator new(size_t size) {
    cout << "hh global new()\n";
    return myAlloc(size);
}

inline void* operator new[](size_t size) {
    cout << "hh global new[]()\n;
    return myAlloc(size);
}

inline void operator delete(void* ptr) {
    cout << "hh golbal delete()\n";
    myFree(ptr);
}

inline void operator delete[](void* ptr) {
    cout << "hh global delete[]()\n";
    myFree(ptr);
}
```

+ 重载成员operator new和operator delete
```
class Foo {
public:
    void* operator new(size_t);
    void operatoe delete(void*, size_t);
};

Foo* p = new Foo;
try {
    void* mem = operator new(sizeof(Foo));
    p = static_cast<Foo*>(men);
    p->Foo::Foo();
}
...
delete p;
p->~Foo();
operator delete(p);
```

+ 重载成员operator new[]和operator delete[]
```
class Foo {
public:
    void* operator new[](size_t);
    void operator delete[](void*, size_t);
}

Foo* p = new Foo[N];
try {
    void* mem = operator new[](sizeof(Foo)*N + 4);
    p = static_cast<Foo*>(mem);
    p->Foo::Foo(); // N次
}

delete[] p;
p->~Foo(); // N次
operator delete(p);
```

# 测试operator new和operator delete
```
class Foo {
public:
    int _id;
    long _data;
    string _str;

public:
    Foo() : _id(0) {cout << "default ctor.this = " << this << " id = " << _id << endl;}
    Foo(int i) : _id(i) {cout << "ctor.this = " << this << "id = " << _id << endl;}

    // virtual
    ~Foo() {cout << "dtor.this = " << this << " id = " << _id << endl;}

    static void* operator new(size_t size) {
        Foo* p = (Foo*)malloc(size);
        cout << "Foo operator new\n";
        return p;
    }

    static void operator delete(void* pdead, size_t size) {
        cout << "Foo operator delete\n";
        free(pdead);
    }

    static void* operator new[](size_t size) {
        Foo* p = (Foo*)malloc(size);
        cout << "Foo operator new[]\n";
        return p;
    }

    static void operator delete[](void* pdead, size_t size) {
        cout << "Foo operator delete[]\n";
        free(pdead);
    }
};

// 若无members就调用globals
Foo *pf = new Foo;
delete pf;

// 强制使用globals
Foo* pf = ::new Foo; // void* ::operator new(size_t);
::delete pf; // void ::operator delete(void*);

// 分配内存多一个整数大小，作为计数(counter)，统计创建对象数量
Foo* pArray = new Foo[6]; // 调用5次构造函数
delete[] pArray; // 调用5次析构函数
// 若写为下面delete，则只调用1次析构函数，引起内存泄漏
delete pArray;
```

# 重载new(), delete()
+ placement new
我们可以重载class member operator new()，写出多个版本，前提是每一个版本的声明都必须有**独特的参数列**，其中第一参数必须是size_t，其余参数以new所指定的placement arguments为初值。出现再new(...)小括号内的便是所谓placement arguments。
`Foo* pf = new(300, 'c')Foo;`
+ placement delete
我们也可以重载class member operator delete()，写出多个版本。但他们绝不会被delete调用。只有当new所调用的ctor抛出**exception(异常)**，才会调用这些重载的operator delete()。他也只可能这样被调用，组要用来归还未能完全创建成功的object所占用的memory。