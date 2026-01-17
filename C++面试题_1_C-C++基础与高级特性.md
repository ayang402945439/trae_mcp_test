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
    Base* ptr = new Derived();  // 基类指针指向派生类对象
    ptr->show();  // 运行时绑定，调用Derived::show()
    
    delete ptr;
    return 0;
}
```

### 5. 介绍一下虚函数，虚函数怎么实现的
**答案**：虚函数是C++中实现多态的机制，通过在基类中使用`virtual`关键字声明，允许派生类重写这些函数。

实现机制：
1. 虚函数表（vtable）：每个包含虚函数的类都有一个虚函数表，存储该类所有虚函数的地址。
2. 虚表指针（vptr）：每个对象在创建时会分配一个隐藏的虚表指针，指向所属类的虚函数表。
3. 动态绑定：当通过基类指针或引用调用虚函数时，编译器会生成代码，通过对象的vptr找到对应的vtable，然后根据函数在表中的位置调用相应的函数。

**示例**：
```cpp
#include <iostream>

class Shape {
public:
    virtual double area() const = 0;  // 纯虚函数
    virtual ~Shape() {}
};

class Circle : public Shape {
private:
    double radius;
    
public:
    Circle(double r) : radius(r) {}
    
    double area() const override {  // 重写纯虚函数
        return 3.14159 * radius * radius;
    }
};

class Rectangle : public Shape {
private:
    double width;
    double height;
    
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    
    double area() const override {  // 重写纯虚函数
        return width * height;
    }
};

int main() {
    Shape* shapes[] = {
        new Circle(5.0),
        new Rectangle(3.0, 4.0)
    };
    
    for (int i = 0; i < 2; i++) {
        std::cout << "Area: " << shapes[i]->area() << std::endl;
#include <iostream>
#include <vector>

class Animal {
public:
    virtual void makeSound() const = 0;
    virtual ~Animal() {}
};

class Dog : public Animal {
public:
    void makeSound() const override {
        std::cout << "Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() const override {
        std::cout << "Meow!" << std::endl;
    }
};

// 多态使用场景：统一处理不同类型的对象
void letAnimalsSpeak(const std::vector<Animal*>& animals) {
    for (const auto& animal : animals) {
        animal->makeSound();  // 多态调用
    }
}

int main() {
    std::vector<Animal*> animals;
    animals.push_back(new Dog());
    animals.push_back(new Cat());
    
    letAnimalsSpeak(animals);  // 统一处理不同动物
    
    for (auto animal : animals) {
        delete animal;
    }
    
    return 0;
}

### 6. 多态和继承在什么情况下使用
**答案**：
- **继承**：当需要创建一个新类，该类需要复用现有类的属性和方法，并可能添加新的属性和方法时使用继承。继承体现了"is-a"的关系。
- **多态**：当需要对不同类型的对象进行统一处理，或者需要在运行时根据对象的实际类型调用相应的方法时使用多态。多态提高了代码的灵活性和可扩展性。

**示例**：
```cpp
#include <iostream>
#include <vector>

class Animal {
public:
    virtual void makeSound() const = 0;
    virtual ~Animal() {}
};

class Dog : public Animal {
public:
    void makeSound() const override {
        std::cout << "Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void makeSound() const override {
        std::cout << "Meow!" << std::endl;
    }
};

// 多态使用场景：统一处理不同类型的对象
void letAnimalsSpeak(const std::vector<Animal*>& animals) {
    for (const auto& animal : animals) {
        animal->makeSound();  // 多态调用
    }
}

int main() {
    std::vector<Animal*> animals;
    animals.push_back(new Dog());
    animals.push_back(new Cat());
    
    letAnimalsSpeak(animals);  // 统一处理不同动物
    
    for (auto animal : animals) {
        delete animal;
    }
    
    return 0;
}
```

### 7. 除了多态和继承还有什么面向对象方法
**答案**：除了多态和继承，面向对象编程还有以下核心方法：
- **封装**：将数据和操作数据的方法封装在一起，隐藏内部实现细节，只暴露必要的接口。
- **抽象**：提取共同特征形成抽象类或接口，定义行为规范而不提供具体实现。
- **组合**：通过将其他对象作为成员变量来构建更复杂的对象，体现"has-a"的关系。
- **聚合**：一种特殊的组合关系，表示整体与部分的关系，但部分可以独立于整体存在。

**示例（封装与组合）**：
```cpp
#include <iostream>
#include <string>

// 封装：Person类封装了name和age属性，只通过公共方法访问
class Person {
private:
    std::string name;
    int age;
    
public:
    Person(const std::string& n, int a) : name(n), age(a) {}
    
    std::string getName() const { return name; }
    int getAge() const { return age; }
};

// 组合：Car类包含Person对象作为成员
class Car {
private:
    std::string brand;
    Person owner;  // 组合关系：Car有一个Person作为所有者
    
public:
    Car(const std::string& b, const Person& o) : brand(b), owner(o) {}
    
    void showInfo() const {
        std::cout << "Car brand: " << brand << std::endl;
        std::cout << "Owner: " << owner.getName() << ", Age: " << owner.getAge() << std::endl;
    }
};

int main() {
    Person person("Alice", 30);
    Car car("Toyota", person);
    
    car.showInfo();
    
    return 0;
}
```

### 8. C++内存分布。什么样的数据在栈区，什么样的在堆区
**答案**：C++程序的内存分布主要分为以下几个区域：
- **栈区（Stack）**：由编译器自动分配和释放，存放函数的参数值、局部变量等。特点是速度快，但空间有限。
- **堆区（Heap）**：由程序员手动分配（new/malloc）和释放（delete/free），若不手动释放，程序结束后由操作系统回收。空间较大，但分配和释放速度较慢。
- **全局/静态区（Global/Static）**：存放全局变量和静态变量，程序结束后由操作系统释放。
- **常量区（Constant）**：存放常量字符串等，程序结束后由操作系统释放。
- **代码区（Code）**：存放程序的二进制代码。

**示例**：
```cpp
#include <iostream>

// 全局变量：全局/静态区
int globalVar = 10;

int main() {
    // 局部变量：栈区
    int localVar = 20;
    
    // 静态局部变量：全局/静态区
    static int staticVar = 30;
    
    // 常量：常量区
    const char* constStr = "Hello World";
    
    // 动态分配内存：堆区
    int* heapVar = new int(40);
    
    std::cout << "Global var: " << globalVar << std::endl;
    std::cout << "Local var: " << localVar << std::endl;
    std::cout << "Static var: " << staticVar << std::endl;
    std::cout << "Const string: " << constStr << std::endl;
    std::cout << "Heap var: " << *heapVar << std::endl;
    
    delete heapVar;  // 手动释放堆内存
    return 0;
}
```

### 9. C++内存管理（RAII机制）
**答案**：RAII（Resource Acquisition Is Initialization，资源获取即初始化）是C++中用于管理资源的重要机制。其核心思想是：
- 在对象构造时获取资源
- 在对象析构时自动释放资源
- 利用栈对象的自动销毁特性，确保资源在任何情况下（包括异常）都能被正确释放

**示例**：
```cpp
#include <iostream>
#include <fstream>

// RAII实现文件资源管理
class FileHandler {
private:
    std::ofstream file;
    
public:
    FileHandler(const std::string& filename) : file(filename) {
        if (!file.is_open()) {
            throw std::runtime_error("Failed to open file");
        }
        std::cout << "File opened: " << filename << std::endl;
    }
    
    ~FileHandler() {
        if (file.is_open()) {
            file.close();
            std::cout << "File closed" << std::endl;
        }
    }
    
    // 禁止拷贝构造和赋值（简单实现）
    FileHandler(const FileHandler&) = delete;
    FileHandler& operator=(const FileHandler&) = delete;
    
    void write(const std::string& content) {
        file << content << std::endl;
    }
};

int main() {
    try {
        FileHandler file("test.txt");  // 构造时打开文件
        file.write("Hello RAII!");      // 使用文件
        // 离开作用域时自动调用析构函数关闭文件
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;
    }
    
    return 0;
}
```

### 10. C++从源程序到可执行程序的过程
**答案**：C++源程序到可执行程序的过程主要包括四个阶段：
1. **预处理（Preprocessing）**：处理预编译指令（如#include、#define等），生成.i文件。
2. **编译（Compilation）**：将预处理后的.i文件编译成汇编语言文件（.s文件）。
3. **汇编（Assembly）**：将汇编语言文件转换成机器语言目标文件（.o文件或.obj文件）。
4. **链接（Linking）**：将多个目标文件和库文件链接成一个可执行程序（.exe文件或ELF文件）。

**示例**：
```bash
# 假设源文件为main.cpp

# 1. 预处理
g++ -E main.cpp -o main.i

# 2. 编译
g++ -S main.i -o main.s

# 3. 汇编
g++ -c main.s -o main.o

# 4. 链接
g++ main.o -o main.exe

# 一步完成所有过程
g++ main.cpp -o main.exe
```

### 11. 一个对象=另一个对象会发生什么（赋值构造函数）
**答案**：当使用一个对象给另一个对象赋值时，会调用赋值运算符重载函数（赋值构造函数）。如果没有自定义赋值运算符，编译器会生成默认的赋值运算符，执行浅拷贝。

**示例**：
```cpp
#include <iostream>
#include <cstring>

class String {
private:
    char* data;
    int length;
    
public:
    // 构造函数
    String(const char* str = "") {
        length = strlen(str);
        data = new char[length + 1];
        strcpy(data, str);
    }
    
    // 拷贝构造函数
    String(const String& other) {
        length = other.length;
        data = new char[length + 1];
        strcpy(data, other.data);
    }
    
    // 赋值运算符重载（赋值构造函数）
    String& operator=(const String& other) {
        if (this != &other) {  // 防止自赋值
            // 释放原有资源
            delete[] data;
// 分配新资源并拷贝数据
            length = other.length;
            data = new char[length + 1];
            strcpy(data, other.data);
        }
        return *this;
    }
    
    // 析构函数
    ~String() {
        delete[] data;
    }
    
    const char* c_str() const {
        return data;
    }
};

int main() {
    String str1("Hello");
    String str2("World");
    
    std::cout << "str1: " << str1.c_str() << std::endl;
    std::cout << "str2: " << str2.c_str() << std::endl;
    
    str2 = str1;  // 调用赋值运算符重载
    
    std::cout << "After assignment:" << std::endl;
    std::cout << "str1: " << str1.c_str() << std::endl;
    std::cout << "str2: " << str2.c_str() << std::endl;
    
    return 0;
}

# 一级标题：C++面试题与答案

## 二级标题：分类（如"一、C/C++基础与高级特性"）

### 三级标题：题目（如"1. 智能指针实现原理"）
**答案**：使用智能指针或RAII机制可以避免这种情况。智能指针在构造时获取资源，在析构时自动释放资源，即使函数提前return或发生异常，也能确保资源被正确释放。

**示例**：
```cpp
#include <iostream>
#include <memory>

void functionWithRisk() {
    // 使用智能指针管理动态内存
    std::unique_ptr<int> ptr(new int(10));
    
    // 模拟可能的错误情况
    bool errorOccurred = true;
    if (errorOccurred) {
        std::cout << "Error occurred, returning..." << std::endl;
        return;  // 智能指针自动释放内存，不会泄漏
    }
    
    // 如果没有错误，继续使用ptr
    std::cout << *ptr << std::endl;
}

int main() {
    functionWithRisk();
    // 离开functionWithRisk后，ptr自动析构，内存被释放
    std::cout << "Program continues, no memory leak" << std::endl;
    return 0;
}
```

### 13. C++11的智能指针有哪些。weak_ptr的使用场景。什么情况下会产生循环引用
**答案**：C++11引入的智能指针主要有三种：
- **unique_ptr**：独占所有权的智能指针，不允许拷贝，只能移动。
- **shared_ptr**：共享所有权的智能指针，使用引用计数管理资源。
- **weak_ptr**：弱引用的智能指针，不增加引用计数，用于解决shared_ptr的循环引用问题。

**weak_ptr的使用场景**：
- 解决shared_ptr的循环引用问题
- 观察资源是否存在，但不影响资源的生命周期

**循环引用示例**：
```cpp
#include <iostream>
#include <memory>

class B;  // 前向声明

class A {
public:
    std::shared_ptr<B> b_ptr;  // A持有B的shared_ptr
    
    ~A() {
        std::cout << "A destroyed" << std::endl;
    }
};

class B {
public:
    std::shared_ptr<A> a_ptr;  // B持有A的shared_ptr
    
    ~B() {
        std::cout << "B destroyed" << std::endl;
    }
};

int main() {
    {
        std::shared_ptr<A> a(new A());
        std::shared_ptr<B> b(new B());
        
        // 循环引用
        a->b_ptr = b;
        b->a_ptr = a;
        
        // 引用计数都为2
        std::cout << "a use_count: " << a.use_count() << std::endl;  // 2
        std::cout << "b use_count: " << b.use_count() << std::endl;  // 2
    }  // 离开作用域后，引用计数都变为1，不会调用析构函数，导致内存泄漏
    
    std::cout << "Main continues, but A and B are not destroyed!" << std::endl;
    return 0;
}
```

**使用weak_ptr解决循环引用**：
```cpp
#include <iostream>
#include <memory>

class B;  // 前向声明

class A {
public:
    std::shared_ptr<B> b_ptr;  // A持有B的shared_ptr
    
    ~A() {
        std::cout << "A destroyed" << std::endl;
    }
};

class B {
public:
    std::weak_ptr<A> a_ptr;  // B持有A的weak_ptr，不增加引用计数
    
    ~B() {
        std::cout << "B destroyed" << std::endl;
    }
};

int main() {
    {
        std::shared_ptr<A> a(new A());
        std::shared_ptr<B> b(new B());
        
        a->b_ptr = b;
        b->a_ptr = a;  // 使用weak_ptr，不增加a的引用计数
        
        // a的引用计数为1，b的引用计数为2
        std::cout << "a use_count: " << a.use_count() << std::endl;  // 1
        std::cout << "b use_count: " << b.use_count() << std::endl;  // 2
    }  // 离开作用域后，a先被销毁，然后b被销毁，没有内存泄漏
    
    std::cout << "Main continues, A and B are properly destroyed!" << std::endl;
    return 0;
}
```

### 14. 多线程中线程的同步方式有哪些
**答案**：C++中线程同步的主要方式包括：
- **互斥量（mutex）**：用于保护共享数据，确保同一时间只有一个线程访问。
- **条件变量（condition_variable）**：用于线程间的通信，允许一个线程等待另一个线程的信号。
- **原子操作（atomic）**：用于无锁的线程安全操作。
- **读写锁（shared_mutex/C++17）**：允许多个线程同时读取，但写入时独占。
- **信号量（semaphore/C++20）**：用于控制对资源的访问数量。

**示例（互斥量和条件变量）**：
```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <queue>

std::queue<int> data_queue;  // 共享队列
std::mutex mtx;              // 互斥量
std::condition_variable cv;  // 条件变量
bool done = false;           // 结束标志

// 生产者线程
void producer() {
    for (int i = 0; i < 10; ++i) {
        {
            std::unique_lock<std::mutex> lock(mtx);  // 加锁
            data_queue.push(i);
            std::cout << "Produced: " << i << std::endl;
        }  // 自动解锁
        
        cv.notify_one();  // 通知消费者
        std::this_thread::sleep_for(std::chrono::milliseconds(100));
    }
    
    {
        std::unique_lock<std::mutex> lock(mtx);
        done = true;
    }
    
    cv.notify_one();  // 通知消费者结束
}

// 消费者线程
void consumer() {
    while (true) {
        std::unique_lock<std::mutex> lock(mtx);
        
        // 等待队列非空或结束标志
        cv.wait(lock, [] { return !data_queue.empty() || done; });
        
        // 检查是否结束
        if (done && data_queue.empty()) {
            break;
        }
        
        // 处理数据
        int data = data_queue.front();
        data_queue.pop();
        std::cout << "Consumed: " << data << std::endl;
    }
}

int main() {
    std::thread producer_thread(producer);
    std::thread consumer_thread(consumer);
    
    producer_thread.join();
    consumer_thread.join();
    
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

