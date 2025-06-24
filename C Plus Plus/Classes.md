#class 

## Class vs struct
#struct 
- Both class and struct have members (variables or methods)
- Struct members are public by default; class members are private by default
- Class cannot be initialized with **aggregate initialization** like struct

## Member functions overloading
#overload #const 

Methods can be overloaded with arguments or const-ness
```cpp
#include <iostream>
#include <string_view>

struct Date
{
	int year {};
	int month {};
	int day {};

	void print()
	{
		std::cout << "non-const " << year << '/' << month << '/' << day << '\n';
	}

	void print() const
	{
		std::cout << "const " << year << '/' << month << '/' << day << '\n';
	}

	void print(std::string_view prefix) const
	{
		std::cout << "const " << prefix << year << '/' << month << '/' << day << '\n';
	}
};

int main()
{
	const Date today { 2020, 10, 14 };

	today.print(); // "const 2020/10/14"
	today.print("The date is: "); // "const The date is: 2020/10/14"

	Date tomorrow { 2020, 10, 15};
	++tomorrow.day;
	tomorrow.print(); // "non-const 2020/10/16"

	return 0;
}
```

## Access level and specifier

| Access level | Access specifier | Member access | Derived class access | Public access |
| ------------ | ---------------- | ------------- | -------------------- | ------------- |
| Public       | public:          | yes           | yes                  | yes           |
| Protected    | protected:       | yes           | yes                  | no            |
| Private      | private:         | yes           | no                   | no            |
