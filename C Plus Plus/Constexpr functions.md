
```cpp
#include <iostream>

constexpr double calcCircumference(double radius) // now a constexpr function
{
    constexpr double pi { 3.14159265359 };
    return 2.0 * pi * radius;
}

int main()
{
	constexpr double radius { 3.0 };
    constexpr double pi { 3.14159265359 };
	constexpr double circumference_1 { 2.0 * radius * pi }; // not great

    constexpr double circumference_2 { calcCircumference(3.0) }; // fail to compile without the constexpr keyword on the function

    double circumference_3 { calcCircumference(3.0) }; // may evaluate at runtime or compile-time

    return 0;
}
```

Constexpr/consteval functions used in a single source file (.cpp) should be defined in the source file above where they are used.

Constexpr/consteval functions used in multiple source files should be defined in a header file so they can be included into each source file.

Always (required by the standard):
- Constexpr function is called where constant expression is required.
- Constexpr function is called from other function being evaluated at compile-time.

Probably (there’s little reason not to):
- Constexpr function is called where constant expression isn’t required, all arguments are constant expressions.

Possibly (if optimized under the as-if rule):
- Constexpr function is called where constant expression isn’t required, some arguments are not constant expressions but their values are known at compile-time.
- Non-constexpr function capable of being evaluated at compile-time, all arguments are constant expressions.

Never (not possible):
- Constexpr function is called where constant expression isn’t required, some arguments have values that are not known at compile-time.

C++20 introduces the keyword **consteval**, which is used to indicate that a function _must_ evaluate at compile-time, otherwise a compile error will result. Such functions are called **immediate functions**.
```cpp
#include <iostream>

consteval int greater(int x, int y) // function is now consteval
{
    return (x > y ? x : y);
}

int main()
{
    constexpr int g { greater(5, 6) };              // ok: will evaluate at compile-time
    std::cout << g << '\n';

    std::cout << greater(5, 6) << " is greater!\n"; // ok: will evaluate at compile-time

    int x{ 5 }; // not constexpr
    std::cout << greater(x, 6) << " is greater!\n"; // error: consteval functions must evaluate at compile-time

    return 0;
}
```