#cast #using

## static_cast

Most preferred way for type conversion. Use `static_cast<new_type>(expression)` for explicit casting.
```cpp
void print(int x) { std::cout << x << std::endl; }

int main() {
	float y{3.14};
	print(static_cast<int>(y));
}
```

## C-style cast

```cpp
int x { 10 };
std::cout << (double)x << '\n';
std::cout << double(x) << '\n'; // function-style cast
```
Avoid C-style cast because it tries to perform the following C++ casts, in order:
- `const_cast`
- `static_cast`
- `static_cast`, followed by `const_cast`
- `reinterpret_cast`
- `reinterpret_cast`, followed by `const_cast`

## const_cast and reinterpret_cast

Avoid `const_cast` and `reinterpret_cast` unless you have a very good reason to use them.

## Initializing temporary objects

Prefer `static_cast` over initializing a temporary object when a conversion is desired.
```cpp
double x { 10 };
std::cout << int{x} << '\n'; // accecptable
std::cout << unsigned int(x) << '\n'; // compile error
using uint = unsigned int; // type alias
std::cout << uint{x} << '\n'; // accecptable
```

## Type aliases

New way with `using`:
```cpp
using Miles = long;
using FcnType = int(*)(double, char);
```

Traditional way with `typedef`:
```cpp
typedef Distance double;
typedef int (*FcnType)(double, char);
```

Prefer type aliases over typedefs.

A **numeric promotion** is the conversion of certain **smaller** numeric types to certain **larger** numeric types (typically `int` or `double`), so that the CPU can operate on data that matches the natural data size for the processor. Numeric promotions include both integral promotions and floating-point promotions. Numeric promotions are **value-preserving**, meaning there is no loss of value or precision. Not all widening conversions are promotions.

A **numeric conversion** is a type conversion between fundamental types that isn’t a numeric promotion. A **narrowing conversion** is a numeric conversion that may result in the loss of value or precision.