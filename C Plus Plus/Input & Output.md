
## Numbers

Convert `std::cout` to print numbers in certain format: `std::dec`, `std::oct`, and `std::hex`
Convert `std::cout` to print bool as string: `std::boolalpha`
Print numbers in binary: [[Bit Manipulation|std::bitset<#bits>{number}]] (temporary bitset)
Set floating number precision: `std::setprecision()`

## Strings

`operator>>` extracts string from `std::cin` until whitespace, to receive strings with whitespaces:
```cpp
#include <iostream>
#include <string>
int main() {
	std::string name{};
	std::getline(std::cin >> std::ws, name);
	std::cout << "Name is " << name << std::endl;
	return 0;
}
```
Note that `std::ws` is not preserved across calls unlike `std::setprecision()`