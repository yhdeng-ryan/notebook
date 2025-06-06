
#compiler #linker

- `Compiler`: checks the source code for errors and translates them into `object file` per `cpp` file.
- `Object file`: with the file extension `.o` or `.obj` is a intermediate file the contains machine language instructions translated form the source codes, data for `linker`, and data for `debugging`.
- `Linker`: verifies all `object file`, standard `library`, and 3rd party `library` are properly linked, then outputs executables, library files, etc.
- `Build`: refers to the full process of converting source code to executable. It can be automated with tools such as `make` or `build2`.