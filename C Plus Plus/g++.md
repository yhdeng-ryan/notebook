
#compiler

**g++** is the command-line interface for the **GNU C++ Compiler**, part of the larger **GNU Compiler Collection (GCC)**.

terminal example
```
g++ .\source.cpp -o executable -std=c++17
```

###### Include directory
Use `-I` for alternate include directory:
```
g++ -o main -I./source/includes main.cpp
```
Note that there is no space after `-I`.

