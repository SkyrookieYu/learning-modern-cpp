### 1. 改进template表达式内的空格
``` c++
    std::vector<list<int> > // OK in each C++ version
    std::vector<list<int>>   //OK since C++11.
```

### 2.null_ptr 和 std::nullptr_t
``` c++
    在C++11中允许你使用null_ptr取代0或者NULL，用来表示一个pointer（指针）指向所谓的no value.
    C++11 的此项新特性能够帮助你在"null pointer 时候被C++编译器解析为一个整数值" 避免被误解.
    例如: 有二个函数声明
    void f(int )
    void f(void* )

    f(0)  //call f(int)
    f(NULL) //call f(int) if NULL is 0,ambiguous.
    f(nullptr) //call f(void* )
    nullptr 是C++11的新的关键词. 它为自动转换为各种pointer类型. 但不会转为任何整数类型. 
    它拥有std::nullptr_t 类型。
    定义在头文件<cstddef>中. 
    在C++11中，std::nullptr_t 被视为一个基础类型.
```

### 3. auto 完成类型自动推导

### 4. 一致性初始化与初值列

### 5. range-base for 循环

### 6. move语义 和 rvalue reference

### 7. noexcept

### 8. constexpr

### 9. 模板template新特性

### 10. Lamdbda

### 11. std::bind  / std::function

### 12 