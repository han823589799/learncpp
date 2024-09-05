# 转换函数(conversion function)
转换函数operator与double之间有空格
`operator double() const {return amount;}`<br>
操作符重载operator与+之间没有空格
`Complex operator+(Compex &other);`

# 隐式转换(non-explicit-one-argument ctor)
+ argument
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

# 模板偏特化--个数的偏(partial specialization)
泛化又叫全泛化(full specialization)