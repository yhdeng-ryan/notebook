#enum #struct

- A program-defined type used in only one code file should be defined in that code file as close to the first point of use as possible.
- A program-defined type used in multiple code files should be defined in a header file with the same name as the program-defined type and then included into each code file as needed.

## Unscoped enumerations

- Enumerations don’t have to be named, but unnamed enumerations should be avoided in modern C++.
- Name your enumerated types starting with a capital letter. Name your enumerators starting with a lower case letter.
- Prefer putting your enumerations inside a named scope region (such as a namespace or class) so the enumerators don’t pollute the global namespace.

## Unscoped enumerator integral conversions

- Avoid assigning explicit values to your enumerators unless you have a compelling reason to do so.
- If an enumeration is zero-initialized, the enumeration will be given value `0`, even if there is no corresponding enumerator with that value.
- Specify the base type of an enumeration only when necessary.
```cpp
#include <cstdint>  // for std::int8_t
#include <iostream>

// Use an 8-bit integer as the enum underlying type
enum Color : std::int8_t
{
    black,
    red,
    blue,
};

int main()
{
    Color c{ black };
    std::cout << sizeof(c) << '\n'; // prints 1 (byte)
    return 0;
}
```
- explicitly convert an integer to an unscoped enumerator using `static_cast`
- enumerators are constexpr, their values must also be constexpr.

## Overloading the I/O operators for enumerations
#overload

```cpp
#include <iostream>
#include <limits>
#include <optional>
#include <string>
#include <string_view>


enum Color
{
	black,
	red,
	blue,
};

constexpr std::string_view getColorName(Color color)
{
	switch (color)
	{
	case black: return "black";
	case red:   return "red";
	case blue:  return "blue";
	default:    return "???";
	}
}

constexpr std::optional<Color> getColorFromString(std::string_view sv)
{
	if (sv == "black") return black;
	if (sv == "red")   return red;
	if (sv == "blue")  return blue;
	return {};
}

std::ostream& operator<<(std::ostream& out, Color color)
{
	return out << getColorName(color);
}

std::istream& operator>>(std::istream& in, Color& pet)
{
	std::string s{};
	in >> s;
	std::optional<Color> match { getColorFromString(s) };
	if (match)
	{
		pet = *match;
		return in;
	}
	// set input stream to fail state
	in.setstate(std::ios_base::failbit);
	return in;
}

int main()
{
	std::cout << "Enter a color: black, red, or blue: ";
	Color shirt{};
	std::cin >> shirt;
	if (std::cin)
		std::cout << "Your shirt is " << shirt << '\n';
	else
	{
		std::cin.clear(); // reset the input stream to good
		std::cin.ignore(std::numeric_limits<std::streamsize>::max(), '\n');
		std::cout << "Your color was not valid\n";
	}

	return 0;
}
```

## Scoped enumerations (enum classes)

To make a scoped enumeration, we use the keywords `enum class`. They won’t implicitly convert to integers, and the enumerators are _only_ placed into the scope region of the enumeration.
```cpp
#include <iostream>
int main()
{
    enum class Color
    {
        red,
        blue,
    };

    enum class Fruit
    {
        banana,
        apple,
    };

    Color color { Color::red };
    Fruit fruit { Fruit::banana };

    if (color == fruit) // compile error: the compiler doesn't know how to compare different types Color and Fruit
        std::cout << "color and fruit are equal\n";
    else
        std::cout << "color and fruit are not equal\n";

    return 0;
}
```

- Use `static_cast` to convert between enum class and integer. 
- `using enum` statement imports all of the enumerators from an enum into the current scope. (C++20)

## Structs, members, and member selection
#overload

- Aggregates use a form of initialization called **aggregate initialization**, which allows us to directly initialize the members of aggregates.
- The initializers are applied to the members in order of declaration.
- If the member has a default member initializer, that is used.
- Missing initializers will be value-initialized to 0
- Overload `operator<<` to print an enumeration
- const (or constexpr), and just like all const variables, they must be initialized.
- assign values to structs using an initializer list
- minimize padding by defining your members in decreasing order of size see [data structure alignment](https://en.wikipedia.org/wiki/Data_structure_alignment)
```cpp
struct Employee
{
    int id {};
    int age {};
    double wage {60000.0};
};

std::ostream& operator<<(std::ostream& out, const Employee& e)
{
    out << "id: " << e.id << " age: " << e.age << " wage: " << e.wage;
    return out;
}

int main()
{
    Employee joe {};
    joe.age = 32;  // use member selection operator (.) to select the age member of variable joe
    Employee frank = { 1, 32, 60000.0 }; // copy-list initialization using braced list
    Employee joe { 2 };     // list initialization using braced list (preferred)
    Employee ray{ .id{3}, .wage{10} };  // designated initializers (C++20)
    joe = { joe.id, 33, joe.wage };
    return 0;
}
```

## Member selection with pointers and references
#pointer #reference

C++ offers a **member selection from pointer operator (->)** (also sometimes called the **arrow operator**) that can be used to select members from a pointer to an object.
```cpp
#include <iostream>

struct Employee
{
    int id{};
    int age{};
    double wage{};
};

int main()
{
    Employee joe{ 1, 34, 65000.0 };

    ++joe.age;
    joe.wage = 68000.0;

    Employee* ptr{ &joe };
    std::cout << (*ptr).id << '\n'; // Not great but works: First dereference ptr, then use member selection
    std::cout << ptr->id << '\n'; // Better: use -> to select member from pointer to object

    return 0;
}
```

## Class templates with struct
#template

Mixed type with self-defined struct
```cpp
#include <iostream>

template <typename T, typename U>
struct Pair
{
    T first{};
    U second{};
};

template <typename T, typename U>
void print(Pair<T, U> p)
{
    std::cout << '[' << p.first << ", " << p.second << ']';
}

int main()
{
    Pair<int, double> p1{ 1, 2.3 }; // a pair holding an int and a double
    Pair<double, int> p2{ 4.5, 6 }; // a pair holding a double and an int
    Pair<int, int> p3{ 7, 8 };      // a pair holding two ints

    print(p2);

    return 0;
}
```

Because working with pairs of data is common, the C++ standard library contains a class template named `std::pair` (in the `<utility>` header).
```cpp
#include <iostream>
#include <utility>

template <typename T, typename U>
void print(std::pair<T, U> p)
{
    // the members of std::pair have predefined names `first` and `second`
    std::cout << '[' << p.first << ", " << p.second << ']';
}

int main()
{
    std::pair<int, double> p1{ 1, 2.3 }; // a pair holding an int and a double
    std::pair<double, int> p2{ 4.5, 6 }; // a pair holding a double and an int
    std::pair<int, int> p3{ 7, 8 };      // a pair holding two ints
    std::pair p4{ 1, 2.0f }; // class template argument deduction

    print(p2);

    return 0;
}
```

## Template argument deduction guides
#template

With compiler before C++17, class template argument deduction needs to be provided.
```cpp
template <typename T, typename U>
struct Pair
{
    T first{};
    U second{};
};

// deduction guide
template <typename T, typename U>
Pair(T, U) -> Pair<T, U>;

int main()
{
    Pair<int, int> p1{ 1, 2 }; // explicitly specify class template Pair<int, int> (C++11 onward)
    Pair p2{ 1, 2 };           // CTAD used to deduce Pair<int, int> from the initializers (C++17)

    return 0;
}
```