## Local static variable of an inline function in a shared library

### C
#### 1. With `inline` keyword

* Compiler

```sh
$ cc --version
Apple LLVM version 8.1.0 (clang-802.0.42)
Target: x86_64-apple-darwin16.7.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```

* Files

```sh
$ ls
Makefile   collect.c  collect.h  main.c
```

* main.c

```c
#include <stdio.h>
#include "collect.h"

int main()
{
  printf("%d\n", collect(10));
  printf("%d\n", get_sum());
  return 0;
}
```

* collect.h (note the `inline` keyword)

```c
#pragma once

inline int collect(int x)
{
  static int sum = 0;
  sum += x;
  return sum;
}

int get_sum();
```

* collect.c

```c
#include "collect.h"

int get_sum()
{
  return collect(0);
}
```

* Makefile

```make
all: libcollect.dylib main

libcollect.dylib: collect.c collect.h
        $(CC) $< -dynamiclib -o $@

main: main.c libcollect.dylib
        $(CC) $< -o $@ -L. -lcollect
```

* Compile & Run

```
$ make
cc collect.c -dynamiclib -o libcollect.dylib 
In file included from collect.c:1:
./collect.h:5:3: warning: non-constant static local variable in inline function may be different in different files [-Wstatic-local-in-inline]
  static int sum = 0;
  ^
./collect.h:3:1: note: use 'static' to give inline function 'collect' internal linkage
inline int collect(int x)
^
static 
1 warning generated.
Undefined symbols for architecture x86_64:
  "_collect", referenced from:
      _get_sum in collect-a79c3b.o
ld: symbol(s) not found for architecture x86_64
clang: error: linker command failed with exit code 1 (use -v to see invocation)
make: *** [libcollect.dylib] Error 1
```

#### 2. With `static inline` keyword

* collect.h (with `static inline`)

```c
#pragma once

static inline int collect(int x)
{
  static int sum = 0;
  sum += x;
  return sum;
}
int get_sum();
```

* Compile & run

```
$ make
cc collect.c -dynamiclib -o libcollect.dylib
cc main.c -o main -L. -lcollect
$ ./main
10
0
$
```


### C++

### 1. With `inline` keyword

* Compiler

```
$ c++ --version
Apple LLVM version 8.1.0 (clang-802.0.42)
Target: x86_64-apple-darwin16.7.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```

* Files

```
$ ls
Makefile    collect.cc  collect.h   main.cc
```

* main.cc

```c++
#include <cstdio>
#include "collect.h"

int main()
{
  printf("%d\n", collect(10));
  printf("%d\n", get_sum());
  return 0;
}
```

* collect.h

```c++
#pragma once

inline int collect(int x)
{
  static int sum = 0;
  sum += x;
  return sum;
}

int get_sum();
```

* collect.cc

```c++
#include "collect.h"

int get_sum()
{
  return collect(0);
}
```

* Makefile

```make
all: libcollect.dylib main

libcollect.dylib: collect.cc collect.h
        $(CXX) $< -dynamiclib -o $@ 

main: main.cc libcollect.dylib
        $(CXX) $< -o $@ -L. -lcollect
```

* Compile & Run

```sh
$ make
c++ collect.cc -dynamiclib -o libcollect.dylib
c++ main.cc -o main -L. -lcollect
$ ./main
10
10
```

### 2. With `static inline` keyword

* collect.h

```c++
#pragma once

static inline int collect(int x)
{
  static int sum = 0;
  sum += x;
  return sum;
}

int get_sum();
```

* Compile & Run

```
$ make
c++ collect.cc -dynamiclib -o libcollect.dylib
c++ main.cc -o main -L. -lcollect
$ ./main
10
0
```
