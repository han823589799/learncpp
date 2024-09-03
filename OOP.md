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