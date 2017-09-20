## Pathname mangling in MSYS + MinGW with nested Makefile

For this article, we use an executable which prints out all of its arguments
* showargs.c
```c
#include <stdio.h>

int main(int argc, char** argv)
{
  int i;
  printf("Parameters\n");
  printf("----------\n");
  for (i = 0 ; i < argc ; ++i) {
    printf("[%d] %s\n", i, argv[i]);
  }
  return 0;
}
```

### Nested `Makefile`s

Create a directory structure with nested `Makefile`s.

* List of files
```sh
$ find .
.
./Makefile
./showargs.exe
./src
./src/Makefile
```

Contents of the `Makefile`s are as follows:

* Makefile
```make
all:
	cd src && $(MAKE)
```

* src/Makefile
```make
all:
	../showargs / /usr
```

### Results

* Using msys-make
```sh
$ make
cd src && make
make[1]: Entering directory '/home/user/test/src'
../showargs / /usr
Parameters
----------
[0] C:\msys64\home\user\test\showargs.exe
[1] C:/msys64/
[2] C:/msys64/usr
make[1]: Leaving directory '/home/user/test/src'
```

* Using mingw32-make
```sh
$ mingw32-make.exe
cd src && C:/msys64/mingw64/bin/mingw32-make
mingw32-make[1]: Entering directory 'C:/msys64/home/user/test/src'
../showargs / /usr
Parameters
----------
[0] ../showargs
[1] /
[2] /usr
mingw32-make[1]: Leaving directory 'C:/msys64/home/user/test/src'
```
