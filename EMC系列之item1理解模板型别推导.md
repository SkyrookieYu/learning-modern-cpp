### 1. CPP模板类型推导

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

*??? 待验证 特殊案例：expr是个指向到const对象的const指针，且expr按值传递给param.*
``` c++
template<typename T>
void f(T param);
const char* const ptr = "fun with pointers"; 
```
- f(ptr) - ??? T -> char*, param -> const char* const型别.


### 1.4 数组实参


### 1.5 函数实参


## 2. 总结

- 在模板型别推导过程中, 具有引用型别的实参会被当成非引用型别来处理,换言之，其引用性会被忽略.
- 对万能引用形参进行推导时,左值实参会进行特殊处理.
- 对按值传递的形参进行推导时,若实参型别中带有const或volatile修辞词,则它们还是会被当作不带const或者volatile的型别来处理.
- 在模板型别推导过程中,数组或者函数型别的实参会退化成对应的指针,除非它们被用来初始化引用.

## 3. 代码附件