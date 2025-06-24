## Constexpr if
#constexpr

For C++17, the conditional of a constexpr-if-statement will be evaluated at compile-time.
```cpp
constexpr int a {1};

if constexpr (a == 1)
	std::cout << "one\n";
else
	std::cout << "not one\n";
```

## Switch case

Use `[[fallthrough]]` attributes to indicate intentional fall through:
```cpp
switch (2)
{
case 1:
	std::cout << 1 << '\n';
	break;
case 2:
	std::cout << 2 << '\n'; // Execution begins here
	[[fallthrough]]; // intentional fallthrough -- note the semicolon to indicate the null statement
case 3:
	std::cout << 3 << '\n'; // This is also executed
	break;
}
```

You can declare but not initialize variables inside switch, use code blocks if you must:
```cpp
switch (1)
{
int a; // okay
int b{ 5 }; // illegal

case 1:
{ // note addition of explicit block here
    int x{ 4 }; // okay, variables can be initialized inside a block inside a case
    std::cout << x;
    break;
}

default:
	int c{ 4 }; // illegal
	int d; // okay
    break;
}
```

## Goto statements

Avoid goto statements (unless the alternatives are significantly worse for code readability).
```cpp
#include <iostream>
void printCats(bool skip)
{
    if (skip)
        goto end; // jump forward; statement label 'end' is visible here due to it having function scope
    std::cout << "cats\n";
end:
    ; // statement labels must be associated with a statement
}
int main()
{
    printCats(true);  // jumps over the print statement and doesn't print anything
    printCats(false); // prints "cats"
    return 0;
}
```

## For statements

For loops with multiple counters:
```cpp
for (int x{ 0 }, y{ 9 }; x < 10; ++x, --y)
    std::cout << x << ' ' << y << '\n';
```

## Halts
#error 

A **halt** is a flow control statement that terminates the program:
```cpp
#include <cstdlib> // for std::exit()
#include <iostream>

void cleanup()
{
    // code here to do any kind of cleanup required
    std::cout << "cleanup!\n";
}

int main()
{
    // register cleanup() to be called automatically when std::exit() is called
    std::atexit(cleanup); // note: we use cleanup rather than cleanup() since we're not making a function call to cleanup() right now
    std::cout << 1 << '\n';

    std::exit(0); // terminate and return status code 0 to operating system

    // The following statements never execute
    std::cout << 2 << '\n';

    return 0;
}
```

In multi-threaded programs, consider `std::quick_exit()` and `std::at_quick_exit()`. The `std::abort()` function causes your program to terminate abnormally.

## Random number
#random

Mersenne Twister is best option in terms of performance and quality:
```cpp
#include <iostream>
#include <random> // for std::mt19937
#include <chrono> // for std::chrono

int main()
{
	// Seed our Mersenne Twister using steady_clock
	std::mt19937 mt{ static_cast<std::mt19937::result_type>(
		std::chrono::steady_clock::now().time_since_epoch().count()
		) };

	// Create a reusable random number generator that generates uniform numbers between 1 and 6
	std::uniform_int_distribution die6{ 1, 6 }; // for C++14, use std::uniform_int_distribution<> die6{ 1, 6 };

	std::cout << mt() << '\n';     // generate one 32-bit number
	std::cout << die6(mt) << '\n'; // generate a roll of the die 1~6
	
	return 0;
}
```

Only seed a given pseudo-random number generator once, and do not reseed it. `mt()` is a concise syntax for calling the function `mt.operator()`.