
## Lvalue and rvalue expressions

Lvalue expressions evaluate to an identifiable object. Rvalue expressions evaluate to a value. An lvalue will implicitly convert to an rvalue. This means an lvalue can be used anywhere an rvalue is expected. An rvalue, on the other hand, will not implicitly convert to an lvalue.
```cpp
int return5()
{
    return 5;
}

int main()
{
    int x{ 5 }; // 5 is an rvalue expression
    const double d{ 1.2 }; // 1.2 is an rvalue expression

    int y { x }; // x is a modifiable lvalue expression
    const double e { d }; // d is a non-modifiable lvalue expression
    int z { return5() }; // return5() is an rvalue expression (since the result is returned by value)

    int w { x + 1 }; // x + 1 is an rvalue expression
    int q { static_cast<int>(d) }; // the result of static casting d to an int is an rvalue expression

    return 0;
}
```

## Lvalue references

An **lvalue reference** (commonly just called a “reference” since prior to C++11 there was only one type of reference) acts as an alias for an existing lvalue (such as a variable). Once initialized, a reference in C++ cannot be **reseated**.
```cpp
int x { 5 };
int y { 6 };

int& ref { x }; // ref is now an alias for x

ref = y; // assigns 6 (the value of y) to x (the object being referenced by ref)
// The above line does NOT change ref into a reference to variable y!

std::cout << x << '\n'; // user is expecting this to print 5
```

If you try to bind a reference to an object that does not match its referenced type, the compiler will try to implicitly convert the object to the referenced type (temporary rvalue) and then bind the reference to that.
- non-const lvalue reference can only bind to lvalue
- const lvalue reference can both bind to modifiable lvalue, non-modifiable lvalue, and rvalue
```cpp
short x {1};
int& y {x};  // compile error
const int& z{x}; // okay, but binded to temp obj

--x;
std::cout << x; // 0
std::cout << z; // 1
```

Constexpr lvalue reference can only be bound to static variable. When defining a constexpr reference to a const variable, we need to apply both `constexpr` (which applies to the reference) and `const` (which applies to the type being referenced).
```cpp
int g_x { 5 };

int main()
{
    constexpr int& ref1 { g_x }; // ok, can bind to global

    static int s_x { 6 };
    constexpr int& ref2 { s_x }; // ok, can bind to static local

    int x { 6 };
    constexpr int& ref3 { x }; // compile error: can't bind to non-static object

	static const int s_x { 6 }; // a const int
    constexpr const int& ref2 { s_x }; // needs both constexpr and const

    return 0;
}
```

## Pass by const lvalue reference

The following are often passed by value (because it is more efficient):

- Enumerated types (unscoped and scoped enumerations).
- Views and spans (e.g. `std::string_view`, `std::span`).
- Types that mimic references or (non-owning) pointers (e.g. iterators, `std::reference_wrapper`).
- Cheap-to-copy class types that have value semantics (e.g. `std::pair` with elements of fundamental types, `std::optional`, `std::expected`).

Pass by reference should be used for the following:

- Arguments that need to be modified by the function.
- Types that aren’t copyable (such as `std::ostream`).
- Types where copying has ownership implications that we want to avoid (e.g. `std::unique_ptr`, `std::shared_ptr`).
- Types that have virtual functions or are likely to be inherited from (due to object slicing concerns)

Prefer passing strings using `std::string_view` (by value) instead of `const std::string&`, unless your function calls other functions that require C-style strings or `std::string` parameters or you're using C++14 or older.

## Null pointers

To check a null pointer:
1.  `if (ptr == nullptr)`
2. `if (ptr)`
3. `if (ptr == 0)`: legacy not recommended
4. `if (ptr == NULL)`: legacy not recommended

## Pointers and const

A **pointer to a const value** is a pointer that points to a constant value.
```cpp
const int x{ 5 };
const int y{ 6 };
const int* ptr { &x };
ptr = &y; // okay: ptr now points at const int y
```

A **const pointer** is a pointer whose address can not be changed after initialization.
```cpp
int x{ 5 };
int y{ 6 };
int* const ptr { &x };
ptr = &y; // error: once initialized, a const pointer can not be changed.
```

Prefer pointer-to-const function parameters over pointer-to-non-const function parameters, unless the function needs to modify the object passed in. Do not make function parameters const pointers unless there is some specific reason to do so. Prefer pass by reference to pass by address unless you have a specific reason to use pass by address.

## Pass by address

One of the more common uses for pass by address is to allow a function to accept an “optional” argument.
```cpp
#include <iostream>
void printIDNumber(const int *id=nullptr)
{
    if (id)
        std::cout << "Your ID number is " << *id << ".\n";
    else
        std::cout << "Your ID number is not known.\n";
}

int main()
{
    printIDNumber(); // we don't know the user's ID yet
    int userid { 34 };
    printIDNumber(&userid); // we know the user's ID now
    return 0;
}
```

However, in many cases, function overloading is a better alternative to achieve the same result:
```cpp
#include <iostream>
void printIDNumber()
{
    std::cout << "Your ID is not known\n";
}
void printIDNumber(int id)
{
    std::cout << "Your ID is " << id << "\n";
}

int main()
{
    printIDNumber(); // we don't know the user's ID yet
    int userid { 34 };
    printIDNumber(userid); // we know the user is 34
    printIDNumber(62); // now also works with rvalue arguments
    return 0;
}
```

 Pass pointers by reference if you want to modify the pointer itself:
 ```cpp
#include <iostream>
void nullify(int*& refptr) // refptr is now a reference to a pointer
{
    refptr = nullptr; // Make the function parameter a null pointer
}

int main()
{
    int x{ 5 };
    int* ptr{ &x }; // ptr points to x
    std::cout << "ptr is " << (ptr ? "non-null\n" : "null\n");
    nullify(ptr);
    std::cout << "ptr is " << (ptr ? "non-null\n" : "null\n");
    return 0;
}
```

`nullptr` has the type `std::nullptr_t`.

## Return by reference and return by address

- The object being returned by reference must exist after the function returns
- Reference lifetime extension does not work across function boundaries
- Avoid returning references to non-const local static variables
- Initializing a normal variable with a returned reference makes a copy
- Prefer return by reference over return by address unless the ability to return “no object” (using `nullptr`) is important.
```cpp
#include <iostream>
// takes two integers by non-const reference, and returns the greater by reference
int& max(int& x, int& y)
{
    return (x > y) ? x : y;
}

int main()
{
    int a{ 5 };
    int b{ 6 };
    max(a, b) = 7; // sets the greater of a or b to 7
    std::cout << a << b << '\n';
    return 0;
}
```

## In and out parameters

- Avoid out-parameters (except in the rare case where no better options exist).
- Prefer pass by reference for non-optional out-parameters.

## std::optional

```cpp
#include <iostream>
#include <optional> // for std::optional (C++17)

// Our function now optionally returns an int value
std::optional<int> doIntDivision(int x, int y)
{
    if (y == 0)
        return {}; // or return std::nullopt
    return x / y;
}

int main()
{
    std::optional<int> result1 { doIntDivision(20, 5) };
    if (result1) // if the function returned a value
        std::cout << "Result 1: " << *result1 << '\n'; // get the value
    else
        std::cout << "Result 1: failed\n";

    std::optional<int> result2 { doIntDivision(5, 0) };

    if (result2)
        std::cout << "Result 2: " << *result2 << '\n';
    else
        std::cout << "Result 2: failed\n";

    return 0;
}
```

```cpp
std::optional<int> o1 { 5 };            // initialize with a value
std::optional<int> o2 {};               // initialize with no value
std::optional<int> o3 { std::nullopt }; // initialize with no value
if (o1.has_value()) // call has_value() to check if o1 has a value
if (o2)             // use implicit conversion to bool to check if o2 has a value
std::cout << *o1;             // dereference to get value stored in o1 (undefined behavior if o1 does not have a value)
std::cout << o2.value();      // call value() to get value stored in o2 (throws std::bad_optional_access exception if o2 does not have a value)
std::cout << o3.value_or(42); // call value_or() to get value stored in o3 (or value `42` if o3 doesn't have a value)
```