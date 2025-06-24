#overload #template 

## Function overload resolution

1. Exact match:
```cpp
void foo(int) {}
void foo(double) {}
int main()
{
    foo(0);   // exact match with foo(int)
    foo(3.4); // exact match with foo(double)
    return 0;
}
```

2. **trivial conversions** are considered as exact match:
```cpp
void foo(const int) {}
void foo(const int&) {}
void foo(const double&) {}

int main()
{
    int x { 1 };
    foo(x); // ambiguous match with foo(int) and foo(const int&)

    double d { 2.3 };
    foo(d); // d trivially converted from double to const double& (non-ref to ref conversion)

    return 0;
}
```

3. numeric promotions
4. numeric conversions
5. user defined conversions

## Deleting functions

In cases where we have a function that we explicitly do not want to be callable, we can define that function as deleted by using the **= delete** specifier.
```cpp
#include <iostream>

void printInt(int x)
{
    std::cout << x << '\n';
}
void printInt(char) = delete; // calls to this function will halt compilation
void printInt(bool) = delete; // calls to this function will halt compilation

// This function template will take precedence for arguments of other types
// Since this function template is deleted, calls to it will halt compilation
template <typename T>
void printInt(T x) = delete;

int main()
{
    printInt(97);   // okay
    printInt('a');  // compile error: function deleted (match)
    printInt(true); // compile error: function deleted (match)
    printInt(5.0);  // compile error: function deleted (template)
    return 0;
}
```

## Templates

The scope of a template parameter declaration is strictly limited to the function template (or class template) that follows. we can use **template argument deduction** to have the compiler deduce the actual type that should be used from the argument types in the function call. We can tell the compiler that instantiation of function templates with certain arguments should be disallowed with **= delete**.
```cpp
template <typename T>
T max(T x, T y)
{
    return (x < y) ? y : x;
}

template <>
const char* max(const char* x, const char* y) = delete;

int main()
{
    std::cout << max<int>(1, 2) << '\n';
    std::cout << max<>(1, 2) << '\n'; // template argument deduction
    std::cout << max<>("abc", "cde") << '\n'; // compile error
    return 0;
}
```

When a static local variable is used in a function template, each function instantiated from that template will have a separate version of the static local variable.

Non-type template parameters are used primarily when we need to pass constexpr values to functions (or class types) so they can be used in contexts that require a constant expression. For example, `std::bitset` uses a non-type template parameter to define the number of bits to store because the number of bits must be a constexpr value.
```cpp
#include <iostream>

template <int N> // declare a non-type template parameter of type int named N
void print()
{
    std::cout << N << '\n'; // use value of N here
}

int main()
{
    print<5>(); // 5 is our non-type template argument

    return 0;
}
```

Since functions implicitly instantiated from templates are implicitly inline, templates that are needed in multiple files should be defined in a header file, and then included wherever needed.