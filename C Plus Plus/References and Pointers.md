
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
    [[maybe_unused]] constexpr const int& ref2 { s_x }; // needs both constexpr and const

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