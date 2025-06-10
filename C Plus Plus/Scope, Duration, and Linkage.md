
## Scope operator

- Using the scope resolution operator with no name prefix
```cpp
void print() { std::cout << " there\n"; }

namespace Foo {
	void print() { std::cout << "Hello"; }
}

namespace A {
	namespace B {
		int ab {1};
	}
}

namespace C::D {
	int cd {2};
}

int main()
{
	Foo::print(); // Hello
	::print();    // there
	namespace AB_ns = A::B;
	std::cout << AB_ns::ab;  // 1
	std::cout << C::D::cd;   // 2
	return 0;
}
```

## Linkage

Internal linkage variables can only be access with in the file. All `const` and `constexpr` variables are automatically internal linked. To make a non-constant global variable internal, we use the `static` keyword.

Although functions have external linkage by default, you need forward declaration to tell the compiler it exist so that the linker can connect them. Use `extern` to make constant variable external. Also, use `extern` to forward declare variable that is defined in another file.

Only use `extern` for global variable forward declarations or const global variable definitions.  
Do not use `extern` for non-const global variable definitions (they are implicitly `extern`).

- An identifier with **no linkage** means another declaration of the same identifier refers to a unique entity. Entities whose identifiers have no linkage include:
    - Local variables
    - Program-defined type identifiers (such as enums and classes) declared inside a block
- An identifier with **internal linkage** means a declaration of the same identifier within the same translation unit refers to the same object or function. Entities whose identifiers have internal linkage include:
    - Static global variables (initialized or uninitialized)
    - Static functions
    - Const global variables
    - Unnamed namespaces and anything defined within them
- An identifier with **external linkage** means a declaration of the same identifier within the entire program refers to the same object or function. Entities whose identifiers have external linkage include:
    - Non-static functions
    - Non-const global variables (initialized or uninitialized)
    - Extern const global variables
    - Inline const global variables
    - Namespaces

## Sharing global constants
### Method #1 header file (worse)
`constants.h:`
```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

namespace constants
{
    // Global constants have internal linkage by default
    constexpr double pi { 3.14159 };
}
#endif
```
`main.cpp:`
```cpp
#include "constants.h" // include a copy of each constant in this file
#include <iostream>

int main()
{
    std::cout << "Pi: " << constants::pi << '\n';
    return 0;
}
```

Advantages:
- Works prior to C++17.
- Can be used in constant expressions in any translation unit that includes them.

Downsides:
- Changing anything in the header file requires recompiling files including the header.
- Each translation unit including the header gets its own copy of the variable.

### Method #2 C++ file (ok)
`constants.h:`
```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

namespace constants
{
    // Since the actual variables are inside a namespace, the forward declarations need to be inside a namespace as well
    // We can't forward declare variables as constexpr, but we can forward declare them as (runtime) const
    extern const double pi;
}

#endif
```
`constants.cpp:`
```cpp
#include "constants.h"

namespace constants
{
    // We use extern to ensure these have external linkage
    extern constexpr double pi { 3.14159 };
}
```
`main.cpp:`
```cpp
#include "constants.h" // include all the forward declarations
#include <iostream>

int main()
{
    std::cout << "Pi: " << constants::pi << '\n';
    return 0;
}
```

Advantages:
- Works prior to C++17.
- Only one copy of each variable is required.
- Only requires recompilation of one file if the value of a constant changes.

Disadvantages:
- Forward declarations and variable definitions are in separate files, and must be kept in sync.
- Variables not usable in constant expressions outside of the file in which they are defined.

### Method #3 `inline` (best)
`constants.h:`
```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

// define your own namespace to hold constants
namespace constants
{
    inline constexpr double pi { 3.14159 }; // note: now inline constexpr
}
#endif
```
`main.cpp:`
```cpp
#include "constants.h"
#include <iostream>

int main()
{
    std::cout << "Pi: " << constants::pi << '\n';
    return 0;
}
```
Advantages:
- Can be used in constant expressions in any translation unit that includes them.
- Only one copy of each variable is required.

Downsides:
- Only works in C++17 onward.
- Changing anything in the header file requires recompiling files including the header.

Avoid the use of the `inline` keyword unless you have a specific, compelling reason to do so (e.g. you’re defining those functions or variables in a header file). The compiler needs to be able to see the full definition of an inline function wherever it is used, and all definitions for an inline function (with external linkage) must be identical (or undefined behavior will result).

## Qualified names

A **qualified name** is a name that includes an associated scope. Avoid `using namespace xxx`.
```cpp
#include <iostream>
int foo {"foo"};
int main()
{
   using std::cout; // identifier cout is qualified by namespace std
   cout << "Hello world!\n"; // so no std:: prefix is needed here!
   cout << ::foo; // identifier foo is qualified by the global namespace
   return 0;
} // the using declaration expires at the end of the current scope
```

## Unnamed and inline namespaces

An **unnamed namespace** (also called an **anonymous namespace**) is a namespace that is defined without a name. All identifiers inside an unnamed namespace are treated as if they have internal linkage, which is identical to `static`.

An **inline namespace** is a namespace that is typically used to version content. Anything declared inside an inline namespace is considered part of the parent namespace. However, unlike unnamed namespaces, inline namespaces don’t affect linkage.

```cpp
#include <iostream>
inline namespace V1 // declare an inline namespace named V1
{
    void doSomething()
    {
        std::cout << "V1\n";
    }
}
namespace V2 // declare a normal namespace named V2
{
    void doSomething()
    {
        std::cout << "V2\n";
    }
}
int main()
{
    V1::doSomething(); // calls the V1 version of doSomething()
    V2::doSomething(); // calls the V2 version of doSomething()
    doSomething(); // calls the inline version of doSomething() (which is V1)
    return 0;
}
```