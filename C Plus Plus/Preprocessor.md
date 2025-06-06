Use double quotes to include header files that you’ve written or are expected to be found in the current directory. Use angled brackets to include headers that come with your compiler, OS, or third-party libraries you’ve installed elsewhere on your system.

Use the standard library header files without the .h extension. User-defined headers should still use a .h extension.

![[g++#Include directory]]

Order the includes in the following order:
1. The paired header file for this code file
2. Other headers from the same project
3. 3rd party library headers
4. Standard library headers

One should put the function definition in one of the .cpp files so that the header just contains a forward declaration.

Modern compilers support auto header guards with:
```cpp
#pragma once
```

