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
  code:
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

### 1.2 ParamType是个万能引用


### 1.3 ParamType既非指针也非引用


### 1.4 数组实参


## 2. 总结


## 3. 代码附件