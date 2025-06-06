
## Initialization

A variable can only be initialized once. There are 5 different ways to initialize a variable:
1. Default initialization (no initializer)
```cpp
int a;
```

2. Copy initialization (initial value after equals sign)
```cpp
int a = 1;
```
This is a traditional way and is less efficient with version prior to C++17.

3. Direct initialization
```cpp
int a(1);
```
This is a traditional way to efficiently initialize complex objects, but it also works with basic types. The preferred way in modern C++ is direct list, but it's still used in specific cases such as `static_cast`.

4. Direct list initialization
```cpp
int a{1};
```
This is the modern and recommended way to initialize everything. This is also called uniform initialization or brace initialization. A primary benefits of it is that it disallow narrowing conversions.
```cpp
int a (4.5);  // a is 4
int b = 4.5;  // b is 4
int c {4.5};  // compile error
```

5. Value initialization
```cpp
int a{};
```
This is a special case of direct list initialization where the variable will be implicitly initialized to zero (or whatever value is closest to zero for a given type). It's also called zero initialization.

## Assignment

A variable can be assigned multiple times:
```cpp
a = 1;
a = 2;
```

## Maybe unused

The compiler warns about unused variables, but one can tell it to skip one with the attribute introduced in C++17:
```cpp
[[maybe_unused]] int a {2};
```

## Type conversion

Use `static_cast<new_typoe>(expression)` for  explicit casting.
```cpp
void print(int x) { std::cout << x << std::endl; }

int main() {
	float y{3.14};
	print(static_cast<int>(y));
}
```

## Constants

- `const` means that the value of an object cannot be changed after initialization. The value of the initializer may be known at compile-time or runtime. The `const` object can be evaluated at runtime. const only works on integral variables. Any constant variable whose initializer is not a constant expression (making it a runtime constant) should be declared as `const`.
- `constexpr` means that the object can be used in a constant expression. The value of the initializer must be known at compile-time. The `constexpr` object can be evaluated at runtime or compile-time. Any constant variable whose initializer is a constant expression should be declared as `constexpr`. `constexpr` can also be used on functions.

## Strings

`std::string` automatically change its length

`std::string::length()` returns unsigned integral value, use `static_cast` to convert to signed if need.

Do not pass `std::string` by value; use a `std::string_view` parameter instead

`std::string_view` provides a read-only value of a string. One can assign a new `std::string` for `std::string_view` to point while the old one stays. It is a pointer and a size.

`"foo"` is the C-style string literal that includes `\0` in the end; `"bar"s` is the `std::string` literal; `"bar"sv` is the `"std::string_view` literal

### Function parameters
- Use `std::string` when:
	- The function needs to modify the string passed in as an argument without affecting the caller. This is rare.
	- You are using language standard earlier than C++14.
- Use `std::string_view` when:
	- The function needs a read-only string.
	- The function needs to work wit non-null-terminated strings.
- Use `const std::string&` when:
	- You are using language standard C++14 or older, and the function needs a read-only string to work with (as `std::string_view` is not available until C++17).
	- You are calling other functions that require a `const std::string`, `const std::string&`, or const C-style string (as `std::string` may not be null-terminated).
- Use `std::string&` when:
	- You are using a `std::string` as an out-parameter
	- You are calling other functions that require a `std::string&`, or non-const C-style string.

### Return types
- Use `std::string` when:
	- The return value is a `std::string` local variable or function parameter.
	- The return value is a function call or operator that returns a `std::string` by value.
- Use `std::string_view` when:
	- The function returns a C-style string literal or local `std::string_view` that has been initialized with a C-style string literal.
	- The function returns a `std::string_view` parameter.
	- You are writing an accessor for a `std::string_view` member.
- Use `std::string&` when:
	- The function returns a `std::string&` parameter.
- Use `const std::string&` when:
	- The function returns a `const std::string&` parameter.
	- You are writing an accessor for a `std::string` or `const std::string` member.
	- The function returns a static (local or global) `const std::string`.