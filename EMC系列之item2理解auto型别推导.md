# Effective Modern C++ 系列之 条款2: auto 型别推导

**在理解模板型别推导规则后，那么auto型别推导不是什么问题了，除了一个奇妙的例外情况以外，auto型别推导就是模板型别推导.**

``` c++
template<typename T>
void f(ParamType param);
```
而一次调用形如:

``` c++
f(expr);
```
在f调用中，编译会根据expr来推导T和ParamType的型别.
当变量采用auto来声明时, *auto就扮演了模板中T的角色, 而变量的型别修辞词则扮演的是ParamType的角色*.
例如:

``` c++
auto x = 27;        (1)  x的修饰词就是auto自身 .
const auto cx = x;  (2)  cx的修饰词 const auto .
const auto& rx = x; (3)  rx的修饰词 const auto& .
```
在C++中auto型别推导和模板型别推导是一模一样的(除了在一个例外情况下)，可以利用模板型别推导规则*模拟*auto型别推导过程如下:

``` c++
template<typename T>   //模拟auto型别推导过程.
void func_for_x(T param);

func_fox_x(27);   // 推导得出param的型别就是X的型别.

template<typename T>
void func_for_cx(const T param);

func_for_cx(x)  // 推导得出param的型别就是cX的型别.

template<typename T>
void func_for_rx(const T& param);

func_for_rx(rx);  //推导得出的param的型别就是rx的型别.
``` 

## 1. auto型别推导与模板型别推导一模一样的规则.

**采用auto进行变量声明中，型别修饰词取代ParamType，所以也存在三种情况:**

### 1.1 型别修饰词是指针或引用,但不是万能引用.
``` c++
auto x = 27 ;  
const auto cx = x ;
const auto& rx = x ;
```
### 1.2 型别修饰词是万能引用.

``` c++
auto&& uref1 = x;    // x的型别是int， 且是左值,所以uref1的型别是int&。
auto&& uref2 = cx;   // cx的型别是const int, 且是左值. 所以uref2的型别是const int&.
auto&& uref3 = 27;   // 27的型别是int, 且是右值. 所以uref3的型别是int&&.
```

### 1.3 型别修饰词既非指针也非引用.
**在条款1中，数组和函数名字如何在非引用修饰词的前提下退化成指针，也同样适合auto型别推导**
``` c++
const char name[] = "J. P. Briggs";  //name的型别是const char[13].
auto arr1 = name;   // arr1的型别是const char*.
auto& arr2 = name;  // arr2的型别const char(&)[13].

void someFunc(int, double);     //someFunc是个函数，型别是void(int, double).
auto func1 = someFunc;          //func1的型别是void(*)(int, double).
auto& func2 = someFunc;         //func2的型别是void(&)(int, double).    
```

## 2. auto型别推导与模板型别推导不同存在差异的情况.

**auto声明变量的初始化表达式使用{}时，推导所得的型别属于std::initializer_list**

```c++
auto x = {1,2,3,4,3.0} //错误，推导不出std::initializer_list<T>中T.
```
**模板传入一个{}的初始化表达式，型别推导就会失败**
``` c++
auto x = {1,2,3}  //x的型别是std::initializer_list<int>

template<typename T>
void f(T param)   

f({1,2,3,4}); // 无法推导T的型别.
```

**注意:** auto 和 模板型别推导真正的唯一区别在于,auto会假定用大括号括起来的初始化表达式代表一个std::initializer_list，但模板型别推导却不会.

**在函数返回值或者lambda式的形象中使用auto，意思是使用模板型别推导而非auto型别推导.**

``` c++
auto createInitList {
    return {1,2,3,4};  //错误,无法为{1,2,3}完成型别推导.
}

std::vector<int> v;
...
auto resetv = [&v](const auto& newValue) { v = newValue }; //c++14
...

restv({1,2,3}); //错误！无法为{1,2,3}完成型别推导.
```

## 4. 总结

- 在一般情况下，auto型别推导和模板型别推导是一模一样的,但是auto型别推导会假定用大括号括起来的初始化表达式代表一个std::initializer_list,但是模板型别推导却不会.
-  在函数返回值或lambda式的形参中使用auto,意思是使用模板型别推导而非auto型别推导.

## 5. 验证代码

``` c++
#include <boost/type_index.hpp>
#include <vector>
#include <iostream>

using namespace std;
using namespace boost;
using boost::typeindex::type_id_with_cvr;

int main(int argc, char** argv) {
	auto&& x = 12;
	std::cout << "param =" << type_id_with_cvr<decltype(x)>().pretty_name() << std::endl;
	return 0;
}

``` 