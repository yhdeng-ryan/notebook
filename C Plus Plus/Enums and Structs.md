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