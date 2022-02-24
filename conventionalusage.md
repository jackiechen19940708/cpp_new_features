# 三、移动语义与右值引用
## 1. C++值类别
### 左值(lvalue)
有标识符、可以取地址
■ 非常（non-const）左值可以放在赋值运算符的左侧
■ 常见情况
p 变量、函数名字
p 左值对象的成员
p 返回左值引用的表达式，如 ++x、x = 1、cout << ' '
p 字符串字面量，如 "hello world"
### 纯右值 (prvalue)
没有标识符、不能取地址的“临时对象”
■ 不可以放在赋值运算符的左侧
■ 常见情况
p 返回类型非引用的函数调用或运算符表达式，如 x++、1 + 2
p 除字符串字面量外的字面量，如 true、42
p lambda 表达式
### 将忘值 (xvalue)
■ C++11 引入，和纯右值合称为“右值”
■ 不可以放在赋值运算符的左侧
■ 常见情况
p 右值对象或数组的成员
p 返回右值引用的表达式，如 std::move(x)

### 特点
可以移动的为右值
有标识符的为广义左值
广义左值 glvalue : lvalue xvalue
右值  rvalue : xvalue pralue

### c++11 vs c++11
C++98
■ 左值可以绑定到左值引用
■ 纯右值可以绑定到常左值引用

C++11
■ 左值可以绑定到左值引用
■ 纯右值/将亡值优先绑定到右值引用

拷贝
T::T(const T& rhs);
T& T::operator=(const T& rhs);

移动
T::T(T&& rhs);
T& T::operator=(T&& rhs);

### 例子
Vector 的拷贝构造
Vector(const Vector& rhs) : Allocator(rhs)
{
size_t size = rhs.size();
if (size != 0) {
begin_ = Allocator::allocate(size);
copy(rhs.begin_, rhs.end_, begin_);
end_cap_ = end_ = begin_ + size;
}
}
Vector 的移动构造
Vector(Vector&& rhs) : Allocator(std::move(rhs))
{
begin_ = rhs.begin_;
end_ = rhs.end_;
end_cap_ = rhs.end_cap_;
rhs.begin_ = rhs.end_ = rhs.end_cap_ = nullptr;
}
