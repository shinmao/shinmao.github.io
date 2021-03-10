# Static Link vs Dynamic Link


In this post, I would show the way to create static library and dynamic library and how they link to the executable files. I would not dig into the details of comparison, advantages, or disadvantages between static and dynamic library, please make sure you already have a whole picture before starting on this post.

## Create static library
First, create a source file without main function as our library. Attention, filename should start from `lib` so that compiler can identify it as library file.
```c
/* filename: lib_lib.c */
#include <stdio.h>
void func(void) {
    printf("hello");
}
```
Create header file for `lib_lib.c` so that we can include it in main program later.
```c
/* filename: lib_lib.h */
void func(void);
```
Compile the source program with `gcc -c lib_lib.c -o lib_lib.o`. After we get object file for static libraries, we can **finally create static library** with `ar rcs lib_lib.a lib_lib.o`, which would bundle several object file into single static library.  
I know that object file would link to static libraries at compilation time. Create a main program, and let compiler do it for us with `gcc -o main main.o -L. -l_lib` (`-L.` means static lib exists in current folder; `-lxxx` means link to the lib with name: `xxx`).

## Analyze static-linked Executables
![](/3-9-21/static-lib.png)
In picture above, `nm` command would show us the symbol in the executable, and you can see the symbol of `func`. We also know that compiler will copy the code of lib into executable at compilation time; therefore, I should see the whole code of `func()` if I dump the assembly code of executable.  
![](/3-9-21/static-lib-asm.png)  
In picture above, I dump the assembly code of `main` with `objdump -d main` and can find that the code of `func()` has been copied to `main`.

## Create dynamic library
We can compile the same source file as dynamic library with `gcc -fPIC -c *.c`. This command would generate object file for each source file, and the flag `fPIC` makes sure the code is **position-independent**. In this step, we would get a file: `foo.o`. Also, I would like to create a header file for later use: `foo.h`.  
Next, **create dynamic library** with `gcc *.o -shared -o libfoo.so`, then we can get a dynamic library: `libfoo.so`. Attention, filename should also start from `lib` and end with `.so`.  
Finally, we can link with shared library. To tell program where to search for the libraries, we can use `gcc -L$PWD -Wall -o main main.c -lfoo`, or just `export LD_LIBRARY_PATH=$PWD:$LD_LIBRARY_PATH`.

## Analyze dynamic-linked Executables
![](/static/3-9-21/dynamic-lib.png)  
Same result with `nm` command. After reading the command manual about `nm`, we can compare the result in last section with this section. In previous result of `nm` command, `func()` showed with label `T`, which means text section. In this result, `func()` showed with `U`, which means undefined, definition exists in another file.
![](/static/3-9-21/dynamic-lib-asm.png)
Here is the difference from static library that we cannot see the copied code of `func()`, but only the `func@plt` which would be replaced with the real address with lazy binding later.

## Conclusion
In addition to try from the perspective of compilation, it is also interesting to think about the way to trace the source of function if we get an executable. Welcome to correct me or leave your comments to discuss with me.
