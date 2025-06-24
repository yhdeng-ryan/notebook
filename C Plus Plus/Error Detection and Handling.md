#error

Use `std::cout` for normal user-facing error messages; Use `std::cerr` or a logfile for status and diagnostic information.

`std::abort` is a better option to terminate program when error happens, as typically the developer will be given the option to start debugging at the point where the program aborted.

If the **assertation** evaluates to `false`, an error message is displayed and the program is terminated (via `std::abort`). A little trick to provide more information with `assert` is to AND a string literal: 
```cpp
#include <cassert> // for assert()
assert(gravity>0.0 && "gravity should be greater than 0.0");
```

Use `#define NDEBUG` to disable asserts (must be placed **before** any includes).

Use `static_assert` for compile time checks:
```cpp
static_assert(sizeof(long) == 8, "long must be 8 bytes");
```