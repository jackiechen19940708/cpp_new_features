# 三、移动语义与右值引用
## 1. C++值类别
https://en.cppreference.com/w/cpp/language/value_category
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

### 移动的意义
■ 允许资源的传递
■ 允许返回大对象和容器
p 一般同时使用异常来表示错误

临时字符串的拼接
string result = string("Hello, ") + name + ".";
C++98：低效，不可接受 ；C++11：无不必要的额外开销

## 移动和noexcept
下列成员函数一般不允许抛异常

• 析构函数
• 移动构造函数 《= 若没有标noexcept，c++ vector动态调整甚至不会调用移动构造函数
• 移动赋值运算符
• 交换函数swap

## 五法则
因为用户定义析构函数、拷贝构造函数或拷贝赋值运算符的存
在阻止移动构造函数和移动赋值运算符的隐式定义，所以任何
想要移动语义的类应当声明全部五个特殊成员函数
Clang-Tidy
■ cppcoreguidelines-special-member-functions
■ 常用选项
p AllowMissingMoveFunctions 允许不声明移动函数
p AllowMissingMoveFunctionsWhenCopyIsDeleted 允许删除拷
贝函数时仍不声明移动函数


## 右值引用的误用
Obj&& wrong_move()
{
Obj obj;
// 未定义⾏为
return std::move(obj);
}
Obj bad_move()
{
Obj obj;
// move 会禁⽌ NRVO
return std::move(obj);
}
直接返回Obj就行


## 可能使人惊讶的地方
Vector(Vector&& rhs) : Allocator(std::move(rhs))
{
begin_ = rhs.begin_;
end_ = rhs.end_;
end_cap_ = rhs.end_cap_;
rhs.begin_ = rhs.end_ = rhs.end_cap_ = nullptr;
}
rhs是左值（右值引用【变量】有标识符!!）；使用右值引用调其他函数通常需要
再加 std::move

## 通用函数模板
分别写两个不同的重载？
template <typename T>
void bar(T& s)
{
// Do something
foo(s);
}
template <typename T>
void bar(T&& s)
{
// Do something
foo(std::move(s));
}
  
## 坍缩规则和转发引用
■ 坍缩规则：
p T& & à T&, T& && à T&, T&& & à T&, T&& && à T&&
■ T&& 是转发引用（仅当其出现在函数模板的参数或变量声明中时）
■ std::move(x) 把 x 转换成右值引用
■ std::forward<T>(x) 保持 x 的引用类型
  
## std::forward 的定义
template <class _Tp>
inline _Tp&&
forward(typename remove_reference<_Tp>::type& __t) noexcept
{
return static_cast<_Tp&&>(__t);
}
template <class _Tp>
inline _Tp&&
forward(typename remove_reference<_Tp>::type&& __t) noexcept
{
static_assert(!is_lvalue_reference<_Tp>::value,
"can not forward an rvalue as an lvalue");
return static_cast<_Tp&&>(__t);
}
  
## forward 使用示例
template <typename T>
void bar(T&& s)
{
foo(std::forward<T>(s));
}
int main()
{
circle temp;
bar(temp);
bar(circle());
}
