---
title: C++特性集
date: 2024-03-02 23:44:50
tags:
---

# C++ 17 / C ++ 14 / C++ 11 新特性

<!-- more -->

## C ++ 11

- 自动类型推导（auto）：

C++11引入了auto关键字，可以用来让编译器自动推导变量的类型。

- 基于范围的for循环：

用于遍历容器的新语法，使代码更简洁、更易读。

- nullptr关键字：

用于表示空指针，替代了以往的NULL宏。

- 初始化列表（Initializer lists）：

允许使用花括号{}初始化对象。

- 统一的初始化语法（Uniform initialization）：

初始化现在可以用一种统一的方式进行，减少了过去不同初始化方式的混淆。

- 委托构造函数（Delegating constructors）：

构造函数可以调用同类的其他构造函数来避免代码重复。

- 移动语义和右值引用（Move semantics and rvalue references）：

通过&&引入了右值引用，使得开发人员可以更有效地处理临时对象，减少不必要的对象复制。

- 智能指针（Smart pointers）：

如std::unique_ptr，std::shared_ptr，和std::weak_ptr，提供了自动的内存管理。

- 线程支持库（Thread support library）：

引入了对原生线程的支持。

- Lambda表达式（Lambdas）：

允许定义匿名函数，也是实现闭包的一种方式。

- 静态断言（Static assertions）：

编译时断言，用于在编译阶段进行检查。

- 长整型常量（Long long integer types）：

引入了新的整型long long和unsigned long long，保证至少64位。

- 用户自定义字面量（User-defined literals）：

允许程序员为字面量定义自己的解释。

- 无序容器（Unordered containers）：

引入了基于哈希表的容器，如unordered_map，unordered_set。

- constexpr

## C++ 14

1. 变量模板：允许模板用于变量的声明。

2. 泛型 Lambda 表达式：Lambda 表达式现在可以拥有自动推导类型的参数。

3. 返回类型推导：对于普通函数，C++14支持使用auto关键字推导返回类型，类似于Lambda中所使用的方式。

4. 二进制字面量：允许直接在代码中使用二进制格式表示数字。

5. 数字分隔符：可以在数字字面量中使用单引号'作为分隔符增强可读性。

6. **decltype(auto)**：支持decltype(auto)用于类型推导，这对返回类型或变量类型是依赖于表达式的情况特别有用。

7. 弃用属性：新增了[[deprecated]]属性，用于标记弃用的函数、类型或变量。

8. 扩展的constexpr：放宽了constexpr函数的限制，允许它们包含更多类型的语句，如局部变量和循环。

9. 初始化捕获：在Lambda表达式中允许创建一个新的变量，并将其初始化为捕获列表中的值。

10. 大小不定的数组作为类成员：C++14允许非静态数据成员为大小不定的数组，但这类数组的大小必须在构造对象时确定。

11. 标准化用户定义字面量：改进了用户定义字面量的规则，使得它们更容易使用。

12. 共享指针的改进：std::shared_ptr获得了一些新的工厂函数，如std::make_shared和std::allocate_shared的变体。

13. 松散的constexpr限制：在constexpr函数中使用局部变量、循环和分支语句等成为可能。

14. 弃用之前的动态异常规格：C++14正式弃用了动态异常规格（如throw()），并引入了noexcept作为替代。

## C++17

1. **结构化绑定（Structured Bindings）**：
   允许从数组或元组中一次性解构出多个变量。

2. **内联变量（Inline Variables）**：
   提供了内联静态成员变量，有助于解决多个对象文件中静态成员的多份定义问题。

3. **编译时if语句（constexpr if）**：
   允许在编译时根据模板参数选择代码路径，减少模板元编程的复杂性。

4. **文件系统库（std::filesystem）**：
   标准化了文件和目录的操作。

5. **std::optional、std::variant 和 std::any**：
   提供了更多的类型安全和灵活的数据处理方式。

6. **折叠表达式（Fold Expressions）**：
   简化了可变参数模板的递归和迭代处理。

7. **并行算法（Parallel Algorithms）**：
   标准库的算法现在可以指定执行策略来利用并行执行的好处。

8. **改进的模板推导**：
   函数模板现在可以通过参数类型自动推导模板参数类型，无需显式指定。

9. **std::byte 类型**：
   提供了一个表示内存字节的类型，而不是使用 char 或 unsigned char。

10. **新的属性语法和属性**：
    比如 [[nodiscard]], [[maybe_unused]] 等属性，用于提供更多的编译器指导和警告控制。

11. **选择初始化（Aggregate Initialization）的扩展**：
    允许使用花括号初始化更多种类的对象。

12. **constexpr Lambda 表达式**：
    允许在编译时执行Lambda表达式。

13. **std::string_view**：
    提供了对字符串常量的轻量级、非拥有的视图。

14. 移除了 std::auto_ptr 和其他不推荐的类型