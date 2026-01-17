## 一、C/C++基础与高级特性

### 1. 智能指针实现原理
**答案**：智能指针是C++中用于自动管理动态内存的类模板，通过RAII（资源获取即初始化）机制实现。它在构造时获取资源，在析构时自动释放资源，避免内存泄漏。
**示例**：
```cpp
// 简单智能指针实现
template<typename T>
class SmartPtr {
public:
    SmartPtr(T* ptr = nullptr) : m_ptr(ptr) {}
    ~SmartPtr() {
        delete m_ptr;
        m_ptr = nullptr;
    }
    // 禁止拷贝构造和赋值操作（简单实现）
    SmartPtr(const SmartPtr&) = delete;
    SmartPtr& operator=(const SmartPtr&) = delete;
    T& operator*() const { return *m_ptr; }
    T* operator->() const { return m_ptr; }
private:
    T* m_ptr;
};
// 使用示例
int main() {
    SmartPtr<int> ptr(new int(10));
    std::cout << *ptr << std::endl;  // 输出：10
    // 离开作用域时自动释放内存
    return 0;
}
```
### 2. 智能指针的计数器何时改变
**答案**：智能指针（如shared_ptr）的引用计数器在以下情况会改变：
- 创建shared_ptr时，计数器初始化为1
- 拷贝构造shared_ptr时，计数器+1
- 赋值给其他shared_ptr时，计数器+1
- shared_ptr被销毁时（离开作用域或手动reset），计数器-1
- 计数器为0时，自动释放所管理的资源
**示例**：
```cpp
#include <memory>
#include <iostream>
int main() {
    std::shared_ptr<int> ptr1(new int(10));
    std::cout << "ptr1计数: " << ptr1.use_count() << std::endl;  // 输出：1
    std::shared_ptr<int> ptr2 = ptr1;  // 拷贝构造，计数+1
    std::cout << "ptr1计数: " << ptr1.use_count() << std::endl;  // 输出：2
    std::cout << "ptr2计数: " << ptr2.use_count() << std::endl;  // 输出：2
    {
        std::shared_ptr<int> ptr3(ptr1);  // 拷贝构造，计数+1
        std::cout << "ptr1计数: " << ptr1.use_count() << std::endl;  // 输出：3
    }  // ptr3离开作用域，计数-1
    std::cout << "ptr1计数: " << ptr1.use_count() << std::endl;  // 输出：2
    ptr1.reset();  // 手动reset，计数-1
    std::cout << "ptr2计数: " << ptr2.use_count() << std::endl;  // 输出：1
    return 0;
}
```
### 3. 智能指针和管理的对象分别在哪个区
**答案**：智能指针本身是一个栈上的对象，而它所管理的资源（指针指向的对象）在堆区。利用栈对象超出生命周期后自动析构的特性，无需手动delete释放资源。
**示例**：
```cpp
#include <memory>
int main() {
    // smart_ptr是栈上对象
    std::shared_ptr<int> smart_ptr(new int(10));  // 管理的int对象在堆区
    // 离开作用域时，smart_ptr（栈对象）自动析构，同时释放堆上的int对象
    return 0;
}
```
### 4. 面向对象的特性：多态原理
**答案**：多态是C++面向对象的核心特性之一，允许基类指针或引用指向派生类对象，并在运行时根据实际对象类型调用相应的方法。
实现原理：
- 虚函数：基类中声明为`virtual`的函数
- 虚函数表（vtable）：每个包含虚函数的类都有一个虚函数表，存储该类所有虚函数的地址
- 虚表指针（vptr）：每个对象都有一个虚表指针，指向所属类的虚函数表
- 动态绑定：运行时通过vptr找到vtable，再调用相应的函数
**示例**：
```cpp
#include <iostream>
class Base {
public:
    virtual void show() {  // 虚函数
        std::cout << "Base::show()" << std::endl;
    }
    virtual ~Base() {}
};
class Derived : public Base {
public:
    void show() override {  // 重写虚函数
        std::cout << "Derived::show()" << std::endl;
    }
};
int main() {
    Base* base_ptr = new Derived();  // 基类指针指向派生类对象
    base_ptr->show();  // 运行时多态，调用Derived::show()
    delete base_ptr;
    return 0;
}
```
### 5. 智能指针为什么不能循环引用，如何解决
**答案**：循环引用是指两个或多个对象互相持有对方的shared_ptr，导致引用计数永远不为0，从而产生内存泄漏。

解决方法：
- 使用weak_ptr：weak_ptr是一种不增加引用计数的智能指针，可以观察shared_ptr管理的对象，但不拥有它。
- 打破循环：在适当的时机手动解除循环引用。

**示例**：
```cpp
#include <memory>
#include <iostream>

class B;
class A {
public:
    std::shared_ptr<B> m_ptr_b;
    ~A() { std::cout << "A被销毁" << std::endl; }
};

class B {
public:
    // 使用weak_ptr代替shared_ptr
    std::weak_ptr<A> m_ptr_a;
    ~B() { std::cout << "B被销毁" << std::endl; }
};

int main() {
    {
        std::shared_ptr<A> ptr_a(new A());
        std::shared_ptr<B> ptr_b(new B());
        
        ptr_a->m_ptr_b = ptr_b;
        ptr_b->m_ptr_a = ptr_a;  // weak_ptr不会增加引用计数
    }  // 离开作用域时，ptr_a和ptr_b的引用计数都为0，对象被正确销毁
    
    std::cout << "程序结束" << std::endl;
    return 0;
}
```
### 6. static_cast、dynamic_cast、const_cast、reinterpret_cast的区别和使用场景
**答案**：

| 类型转换 | 功能 | 使用场景 | 安全性 |
|---------|------|---------|-------|
| static_cast | 静态类型转换，编译时检查 | 基本类型转换、向上转型（子类转基类）、void*转换 | 不保证运行时安全 |
| dynamic_cast | 动态类型转换，运行时检查 | 向下转型（基类转子类）、交叉转型 | 安全，但有性能开销 |
| const_cast | 移除const/volatile属性 | 临时修改对象的const属性 | 需谨慎使用，可能导致未定义行为 |
| reinterpret_cast | 重新解释内存，强制类型转换 | 指针类型转换、整数与指针互转 | 最不安全，完全依赖程序员的判断 |

**示例**：
```cpp
#include <iostream>
class Base {
public:
    virtual void show() {}
};
class Derived : public Base {
public:
    void show() override {}
    void derived_func() { std::cout << "Derived::derived_func()" << std::endl; }
};

int main() {
    // static_cast
    int a = 10;
    double b = static_cast<double>(a);  // 基本类型转换
    
    // dynamic_cast
    Base* base_ptr = new Derived();
    Derived* derived_ptr = dynamic_cast<Derived*>(base_ptr);  // 向下转型
    if (derived_ptr) {
        derived_ptr->derived_func();
    }
    
    // const_cast
    const int c = 20;
    int* d = const_cast<int*>(&c);  // 移除const属性
    
    // reinterpret_cast
    int* e = &a;
    long f = reinterpret_cast<long>(e);  // 指针转整数
    
    delete base_ptr;
    return 0;
}
```
### 7. 左值引用和右值引用的区别
**答案**：

| 类型 | 符号 | 引用对象 | 主要用途 |
|------|------|---------|---------|
| 左值引用 | & | 可寻址的左值 | 避免拷贝、函数参数和返回值 |
| 右值引用 | && | 临时对象（右值） | 移动语义、完美转发 |

**示例**：
```cpp
#include <iostream>
#include <string>

void process_lvalue(const std::string& str) {
    std::cout << "左值引用: " << str << std::endl;
}

void process_rvalue(std::string&& str) {
    std::cout << "右值引用: " << str << std::endl;
}

int main() {
    std::string s1 = "Hello";
    process_lvalue(s1);  // 左值传递
    process_rvalue(std::move(s1));  // 强制转为右值
    process_rvalue("World");  // 临时对象是右值
    return 0;
}
```
### 8. 移动构造和移动赋值的作用
**答案**：移动构造和移动赋值是C++11引入的特性，用于优化性能，避免不必要的拷贝操作。它们通过"窃取"临时对象（右值）的资源来构造或赋值新对象，而不是复制资源。

**示例**：
```cpp
#include <iostream>
#include <vector>

class MyString {
public:
    // 默认构造函数
    MyString() : m_data(nullptr), m_size(0) {
        std::cout << "默认构造函数" << std::endl;
    }
    
    // 构造函数
    MyString(const char* str) {
        m_size = strlen(str);
        m_data = new char[m_size + 1];
        strcpy(m_data, str);
        std::cout << "构造函数" << std::endl;
    }
    
    // 拷贝构造函数
    MyString(const MyString& other) {
        m_size = other.m_size;
        m_data = new char[m_size + 1];
        strcpy(m_data, other.m_data);
        std::cout << "拷贝构造函数" << std::endl;
    }
    
    // 移动构造函数
    MyString(MyString&& other) noexcept {
        m_data = other.m_data;
        m_size = other.m_size;
        other.m_data = nullptr;
        other.m_size = 0;
        std::cout << "移动构造函数" << std::endl;
    }
    
    // 拷贝赋值运算符
    MyString& operator=(const MyString& other) {
        if (this != &other) {
            delete[] m_data;
            m_size = other.m_size;
            m_data = new char[m_size + 1];
            strcpy(m_data, other.m_data);
        }
        std::cout << "拷贝赋值运算符" << std::endl;
        return *this;
    }
    
    // 移动赋值运算符
    MyString& operator=(MyString&& other) noexcept {
        if (this != &other) {
            delete[] m_data;
            m_data = other.m_data;
            m_size = other.m_size;
            other.m_data = nullptr;
            other.m_size = 0;
        }
        std::cout << "移动赋值运算符" << std::endl;
        return *this;
    }
    
    // 析构函数
    ~MyString() {
        if (m_data) {
            delete[] m_data;
        }
        std::cout << "析构函数" << std::endl;
    }
    
    const char* c_str() const {
        return m_data;
    }
    
private:
    char* m_data;
    size_t m_size;
};

int main() {
    MyString s1 = "Hello";
    MyString s2 = std::move(s1);  // 使用移动构造函数
    MyString s3;
    s3 = std::move(s2);  // 使用移动赋值运算符
    return 0;
}
```
### 9. 虚函数为什么不能是静态成员函数
**答案**：虚函数需要通过对象的虚表指针（vptr）来调用，而静态成员函数不属于任何对象，没有this指针，因此无法访问虚表指针。此外，虚函数的调用需要动态绑定，而静态成员函数是在编译时确定的。

**示例**：
```cpp
class Base {
public:
    // 错误：虚函数不能是静态的
    // static virtual void func();
    
    virtual void virtual_func() {}  // 正确
    static void static_func() {}  // 正确
};
```
### 10. 内联函数为什么不能是虚函数
**答案**：内联函数在编译时展开，而虚函数在运行时通过虚表动态绑定。内联函数需要在编译时确定函数体，而虚函数的调用需要在运行时才能确定，两者存在矛盾。因此，编译器通常会忽略虚函数的inline关键字。

**示例**：
```cpp
class Base {
public:
    // 虚函数的inline关键字通常会被编译器忽略
    inline virtual void func() {
        // 函数体
    }
};
```
### 11. 指针和引用的区别
**答案**：

| 特性 | 指针 | 引用 |
|------|------|------|
| 定义方式 | T* ptr; | T& ref = var; |
| 初始化 | 可以不初始化，默认为nullptr | 必须初始化 |
| 空值 | 可以为nullptr | 不能为null |
| 指向对象 | 可以指向不同对象 | 只能指向初始化时的对象 |
| 算术运算 | 支持指针运算 | 不支持 |
| 多级 | 支持多级指针（如T**） | 不支持多级引用 |
|  sizeof | 等于指针大小（与平台有关） | 等于被引用对象的大小 |

**示例**：
```cpp
int main() {
    int a = 10;
    
    // 指针
    int* ptr = &a;
    *ptr = 20;  // 修改a的值
    ptr = nullptr;  // 可以指向空
    
    // 引用
    int& ref = a;
    ref = 30;  // 修改a的值
    // int& ref2;  // 错误：必须初始化
    // ref = nullptr;  // 错误：不能指向空
    
    return 0;
}
```
### 12. 指针数组和数组指针的区别
**答案**：

| 类型 | 定义 | 含义 | 大小 |
|------|------|------|------|
| 指针数组 | T* arr[N]; | 一个包含N个T*类型指针的数组 | N * sizeof(T*) |
| 数组指针 | T (*ptr)[N]; | 一个指向包含N个T类型元素的数组的指针 | sizeof(T*) |

**示例**：
```cpp
int main() {
    int a[3] = {1, 2, 3};
    int b[3] = {4, 5, 6};
    
    // 指针数组：包含2个int*指针的数组
    int* arr[2] = {a, b};
    std::cout << "arr[0][1] = " << arr[0][1] << std::endl;  // 输出：2
    
    // 数组指针：指向包含3个int元素的数组的指针
    int (*ptr)[3] = &a;
    std::cout << "(*ptr)[1] = " << (*ptr)[1] << std::endl;  // 输出：2
    
    return 0;
}
```
### 13. 堆和栈的区别
**答案**：

| 特性 | 栈 | 堆 |
|------|------|------|
| 管理方式 | 由编译器自动管理 | 由程序员手动管理（new/delete, malloc/free） |
| 空间大小 | 较小（通常几MB） | 较大（通常几GB） |
| 分配方式 | 静态分配（编译时）和动态分配（运行时） | 动态分配（运行时） |
| 分配效率 | 高（CPU指令直接操作） | 低（需要查找可用内存块） |
| 内存碎片 | 无 | 有 |
| 生长方向 | 向下生长 | 向上生长 |
| 存储内容 | 函数参数、局部变量、返回地址等 | 动态分配的对象 |

**示例**：
```cpp
int global_var;  // 全局变量，存储在静态区

int main() {
    int local_var = 10;  // 局部变量，存储在栈区
    int* ptr = new int(20);  // 动态分配的int，存储在堆区
    
    delete ptr;
    return 0;
}
```
### 14. 多线程编程中，如何正确使用智能指针
**答案**：在多线程编程中使用智能指针需要注意以下几点：

1. **shared_ptr的线程安全性**：
   - shared_ptr的引用计数操作是原子的，线程安全
   - 但shared_ptr指向的对象本身不是线程安全的，需要额外的同步机制

2. **unique_ptr的线程安全性**：
   - unique_ptr不支持拷贝，只能移动，通常在单个线程中使用
   - 如果需要在多个线程间传递unique_ptr，可以使用移动语义

3. **注意事项**：
   - 避免在多个线程中同时修改同一个shared_ptr指向的对象
   - 使用互斥锁或其他同步机制保护shared_ptr指向的对象
   - 不要在多个线程中同时delete同一个指针

**示例**：
```cpp
#include <memory>
#include <thread>
#include <mutex>
#include <iostream>

class SharedData {
public:
    SharedData(int value) : m_value(value) {}
    void increment() {
        std::lock_guard<std::mutex> lock(m_mutex);
        m_value++;
    }
    int getValue() const {
        std::lock_guard<std::mutex> lock(m_mutex);
        return m_value;
    }
private:
    int m_value;
    mutable std::mutex m_mutex;
};

int main() {
    std::shared_ptr<SharedData> data = std::make_shared<SharedData>(0);
    
    // 多个线程共享data指针
    std::thread t1([data]() {
        for (int i = 0; i < 10000; i++) {
            data->increment();
        }
    });
    
    std::thread t2([data]() {
        for (int i = 0; i < 10000; i++) {
            data->increment();
        }
    });
    
    t1.join();
    t2.join();
    
    std::cout << "最终值: " << data->getValue() << std::endl;
    return 0;
}
```
### 15. sizeof是在编译期还是在运行期确定
**答案**：sizeof运算符的结果在编译期确定，它返回类型或变量所占用的内存字节数。
**示例**：
```cpp
#include <iostream>
int main() {
    int a = 10;
    int arr[5];
    std::cout << "sizeof(int): " << sizeof(int) << std::endl;          // 编译期确定
    std::cout << "sizeof(a): " << sizeof(a) << std::endl;              // 编译期确定
    std::cout << "sizeof(arr): " << sizeof(arr) << std::endl;          // 编译期确定
    std::cout << "sizeof(arr[0]): " << sizeof(arr[0]) << std::endl;    // 编译期确定
    std::cout << "Array length: " << sizeof(arr) / sizeof(arr[0]) << std::endl;  // 编译期计算
    return 0;
}
```
### 16. 函数重载的机制。重载是在编译期还是在运行期确定
**答案**：函数重载是指在同一作用域内，可以有一组具有相同函数名、不同参数列表（参数类型、个数或顺序不同）的函数。函数重载的机制是**编译期多态**，编译器在编译时根据函数调用的实参类型和数量，确定调用哪个具体的函数。
**示例**：
```cpp
#include <iostream>
#include <string>

// 函数重载：相同函数名，不同参数类型
void print(int num) {
    std::cout << "Integer: " << num << std::endl;
}

void print(double num) {
    std::cout << "Double: " << num << std::endl;
}

void print(const std::string& str) {
    std::cout << "String: " << str << std::endl;
}

// 函数重载：相同函数名，不同参数个数
int add(int a, int b) {
    return a + b;
}

int add(int a, int b, int c) {
    return a + b + c;
}

int main() {
    print(10);          // 调用print(int)
    print(3.14);        // 调用print(double)
    print("Hello");     // 调用print(const std::string&)
    
    std::cout << add(1, 2) << std::endl;       // 调用add(int, int)
    std::cout << add(1, 2, 3) << std::endl;    // 调用add(int, int, int)
    
    return 0;
}
```