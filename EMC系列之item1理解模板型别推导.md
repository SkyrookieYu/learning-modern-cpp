# Effective Modern C++ 系列之 条款1理解模板型别推导

## 1. CPP模板类型推导

**函数模板一般形式如下:**
``` c++
template<typename T> 
void func(ParamType parm)
```

**而调用形式如下:**
``` c++
func(expr);
```
    在编译期，编译器会通过expr推导二个型别: 一个是T的型别，另一个是ParamType的型别.
  T 和 ParamType，这二个往往不一样。 因为ParmType一般会包含一些修辞符.

**T的型别不仅依赖expr的型别,还依赖ParamType的形式. 具体分为三种情况:**

- 情况一: ParamType具有指针或者引用型别,但不是个万能.
- 情况二: ParamType是个万能引用.
- 情况三: ParamType既非指针也非引用.
  
### 1.1 ParamType具有指针或者引用型别,但不是个万能
    
    
**此种情况下推导规则:**

- 若expr是引用类型，则先将引用部分忽略.
- 然后,对expr的型别和ParamType的型别执行模式匹配，来决定T的类型.

#### 1.1.1 param是个引用

``` c++
template<typename T>
void f(T& param);
```
    变量声明：
``` c++
int x = 27;
const int cx = x;
const int& rx = x;
```
**下面的推导的结果如下**
- f(x)  :   T-> int, param -> int&
- f(cx) :   T-> const int, param -> const int& 
- f(rx) :   T-> const int, param -> const int&

*注意:rx的引用会在推导过程中被忽略.* 

#### 1.1.2 param是个指针(或指向const对象的指针)

``` c++
template<typename T> 
void f(T* param); //param 是个指针

int x = 27;
const int* px = &x; 
```
- f(&x) :   T -> int, param -> int*
- f(px) :   T -> const int, param -> const int*

***C++的型别推导规则对于引用和指针形参都适用***


### 1.2 ParamType是个万能引用

**万能引用形参模板推导规则:**
- 如果expr是个左值,T 和 paramType都会被推导微左值引用. 
- 如果expr是个右值, 则应用情况1的规则.

例如：
``` c++
template<typename T>
void f(T&& param);   //param现在是万能引用了

int x = 27;
const int cx = x;
const int& rx = x;
```

- f(x) :  x是左值, T -> int& , param -> int&
- f(cx):  cx是左值, T -> const int&, param -> const int&
- f(rx):  rx是左值， T->const int& , param-> const int&
- f(27):  27是右值， T -> int,  param-> int&&.

***当遇到万能引用时,型别推导规则会区分实参时左值还是右值.***

### 1.3 ParamType既非指针也非引用
**当ParamType既非指针也非引用时,我们面对的就是所谓按值传递了**
``` c++
template<typename T>
void f(T param); //param 是按值传递.

f(expr); //调用f函数.
```

在c++中按值传递param会是一个副本，推导规则如下:
- 若 expr具有引用型别,则忽略其引用的部分.
- 忽略expr的引用性之后,若expr是个const或volatile对象，也忽略之.

例子：
``` c++
int x = 27;
const int cx = x;
const int& rx = x;
```

- f(x)  : T -> int, param -> int.
- f(cx) : T -> int, param -> int.
- f(rx) : T -> int, param -> int.

*注意：按值传递 param是个完全独立与cx和rx存在的对象，cx和rx的一个副本. expr的不可修改，并不能断定其副本也不可以修改.*

*const和volatile仅会在按值形参处忽略. 若形参时const的引用或者指针，expr的常量性会在推导过程中加以保留*

*特殊案例：expr是个指向到const对象的const指针，且expr按值传递给param.*

``` c++
template<typename T>
void f(T param);
const char* const ptr = "fun with pointers"; 
```
- f(ptr) - T -> const char*, param -> const char*型别.


### 1.4 数组实参
*数组型别有别于指针类型，尽管有时看起来它们可以互换. 形成这种假象的主要原因，在很多情况下，数组会退化成指向其首元素的指针.*
``` c++
const char name[] = "J. P Briggs";   //name的型别时const char[13].
const char* ptrToName = name;        //ptrToName的型别是const char* 。
```
上述代码在编译器上是能通过编译的.

#### 1.4.1 数组传递给持有按值形参的模板
``` c++
template<typename T>
void f(T param);   //持有按值形参的模板.
```
- f(name):  T -> const char*, param -> const char*型别.

#### 1.4.2 数组传递给按引用传递形参的模板
``` c++
template<typename T> 
void f(T& name); //按引用方式传递形参的模板.
```

- f(name): T -> const char[13], param -> const char(&)[13].

**可以利用声明数组引用这一能力创造出一个模板，用来推导处数组含有的元素个数.**
``` c++
template<typename T, std::size_t N> 
constexpr std::size_t array_size(T (&)[N]) noexcept {
  return N ;
}
```
例如：
``` c++
int keyVals[] = {1,2,3,4,5,5,6};
int mappedVals[array_size(keyVals)];
```

**在现代C++中，优先选用std::array:**

``` c++
std::array<int ,array_size(keyVals)> mappedVals;
```
### 1.5 函数实参
*在C++中, 函数型别也同样会退化为函数指针. 并且数组型别推导规则适用函数及函数指针的退化.*
```c++
void someFunc(int, double) // someFunc 是个函数, 其型别为void(int,double);
```
#### 1.5.1 持有按值形参传递的模板
``` c++
template<typename T> 
void f1(T param);
``` 
- f1(somFunc): T-> void(*)(int ,double) , param ->void(*)(int ,double).

#### 1.5.2 持有按引用形参传递的模板 
``` c++
template<typename T> 
void f1(T& param);
``` 
- f1(somFunc): T-> void(int ,double) , param ->void(&)(int ,double).


## 2. 总结

- 在模板型别推导过程中, 具有引用型别的实参会被当成非引用型别来处理,换言之，其引用性会被忽略.
- 对万能引用形参进行推导时,左值实参会进行特殊处理.
- 对按值传递的形参进行推导时,若实参型别中带有const或volatile修辞词,则它们还是会被当作不带const或者volatile的型别来处理.
- 在模板型别推导过程中,数组或者函数型别的实参会退化成对应的指针,除非它们被用来初始化引用.

## 3. 代码使用boost库中type_index
**部分代码如下:**
``` c++
#include <boost/type_index.hpp>
#include <iostream> 
#include <array>

using namespace std;
using namespace boost;
using boost::typeindex::type_id_with_cvr;

template<typename T> 
void f(T& param) {
	std::cout << "T =" << type_id_with_cvr<T>().pretty_name() << std::endl;
	std::cout << "param =" << type_id_with_cvr<decltype(param)>().pretty_name() << std::endl;
}

void someFunc(int, double) {
}

template<typename T, std::size_t N>
constexpr std::size_t array_size(T(&)[N]) noexcept {
	return N;
}

int main(int argc,char** argv) {
#if 0	
	int x = 27;
	const int cx = x;
	const int& rx = x;
	f(x); 
	f(27);
#endif
	//char const * const ptr = "fun with pointers";
	//f(ptr);
	//const char name[] = "J. P. Briggs";
	//f(name);
	//int keyVals[] = { 1,2,3,4,5,5,6 };
	//std::array<int, array_size(keyVals)> mappedVals = { 1,2,3,4,5,5,6 };
	//for(auto e: mappedVals)
	//{
	//	std::cout << e << std::endl;
	//}
	f(someFunc);
	return 0;
}
```