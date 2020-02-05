### 1. CPP模板类型推导

**函数模板一般形式如下:**
``` c++
    template<typename T> 
    void func(ParmType parm)
```

**而调用形式如下:**
``` c++
    func(expr);
```
    在编译期，编译器会通过expr推导二个型别: 一个是T的型别，另一个是ParmType的型别.
  T 和 ParmType，这二个往往不一样。 因为ParmType一般会包含一些修辞符.

**T的型别不仅依赖expr的型别,还依赖ParamType的形式. 具体分为三种情况:**

- 情况一: ParamType具有指针或者引用型别,但不是个万能.
- 情况二: ParamType是个万能引用.
- 情况三: ParamType既非指针也非引用.