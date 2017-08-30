## Building ITensor using MSYS2 and MinGW-x64 toolchain on Windows

In this guide I explain how to build ITensor library on Windows. Unfortunately, the current version of ITensor (2.1.0) cannot be built by the lastest version of Microsoft Visual Studio C++ compiler (2017), because the compiler is not fully C++11 compliant (namely, its lack of support for [Expression SFINAE](http://en.cppreference.com/w/cpp/language/sfinae)). We therefore need to use either Cygwin or MinGW compilers on Windows to build ITensor instead. Here I explain how to build ITensor using MinGW-w64 toolchain.


### Install Toolchain

MSYS2 provides a convinent environment for the MinGW-w64 toolchain. Go to [www.msys2.org](http://www.msys2.org) and download x86_64 version. Follow instructions on the MSYS2 webpage. After the updates, restart `MSYS2 MinGW 64-bit` and install the MinGW-w64 toolchain, git, msys-make and vim:
```
$ pacman -S mingw-w64-x86_64-toolchain git make vim
```

### Install OpenBLAS

ITensor uses BLAS and LAPACK routines. OpenBLAS is an open source implementation of the BLAS, bundled with the reference LAPACK implementation from netlib by default. Binary packages of OpenBLAS for Windows can normally be downloaded from [www.openblas.net](http://www.openblas.net). However, the binary package for the most recent version `v0.2.20` is not available for some reason. Binary package of a previous version `v0.2.19` is available. But it uses an older version of gfortran, so I recommend building OpenBLAS yourself.

To clone the OpenBLAS repository:
```
$ git clone https://github.com/xianyi/OpenBLAS.git ~/.local/src/OpenBLAS
```
This creates a directory named `OpenBLAS` under `~/.local/src`. After cloning, you can list all the previous releases by the following commands: 
```
$ cd ~/.local/src/OpenBLAS
$ git tag
```
Checkout the latest release:
```
$ git checkout v0.2.20
```
To build the library, you simply need to `make`:
```
$ make -j4
```
After the compilation, you can install the files to `~/.local/pkg/OpenBLAS-v0.2.20-Win64-int32` (or any directory of your choice) by the following command:
```
$ make PREFIX=~/.local/pkg/OpenBLAS-v0.2.20-Win64-int32 install
```
This creates directory `~/.local/pkg/OpenBLAS-v0.2.20-Win64-int32` with subdirectories `include`, `lib`, and `bin`, respectively containing the header files, `.a` files, and a `.dll` file.

### Install ITensor

Now we can build ITensor. To clone the ITensor repository:
```
$ git clone https://github.com/ITensor/ITensor.git ~/.local/src/ITensor
```
ITensor uses `options.mk` for configuration. A sample configuration file `options.mk` is included in the ITensor repository. Copy it to `options.mk` by
```
$ cd ~/.local/src/ITensor
$ cp options.mk.sample options.mk
```
Edit `options.mk` using vim (or your favorite text editor)
```
$ vim options.mk
```
Make sure you have the following lines uncommented and correct:
```Makefile
CCCOM=g++ -std=c++11 -Wa,-mbig-obj -O2 -fPIC
    ⋮
PLATFORM=openblas
BLAS_LAPACK_LIBFLAGS=\
	-L$(HOME)/.local/pkg/OpenBLAS-v0.2.20-Win64-int32/lib \
	-lpthread \
	-lopenblas
BLAS_LAPACK_INCLUDEFLAGS=\
	-I$(HOME)/.local/pkg/OpenBLAS-v0.2.20-Win64-int32/include \
       -fpermissive \
       -DHAVE_LAPACK_CONFIG_H \
       -DLAPACK_COMPLEX_STRUCTURE
    ⋮
ITENSOR_MAKE_DYLIB=1
```
The compiler flags `-Wa,-mbig-obj -O2` is a workaround for Windows's limitation on binary files. Because of the `-O2` flag, even the debug build will be optimized. This will unfortunately interfere with a debugger if you are using one, making debugging difficult. `ITENSOR_MAKE_DYLIB=1` is for building dynamic library files `libitensor.dll` and `libitensor-g.dll`. Currently, the sample configuration included in ITensor does not support building of `.dll` files. You need to add the following lines to `options.mk`:
```Makefile
DYLIB_EXT=dll
DYLIB_FLAGS=-shared \
            -Wl,--export-all-symbols \
            -Wl,--enable-auto-import \
            -Wl,--out-implib,$@.a
```

After you've changed `options.mk`, you can build ITensor by
```
$ make -j4
```
Once the build is finished, you can install the files to `~/.local/pkg/ITensor/` by
```
$ mkdir -p ~/.local/pkg/ITensor/include
$ mkdir -p ~/.local/pkg/ITensor/lib
$ mkdir -p ~/.local/pkg/ITensor/bin
$ cp -r itensor ~/.local/pkg/ITensor/include/
$ cp -r lib/*.a ~/.local/pkg/ITensor/lib/
$ cp -r lib/*.dll ~/.local/pkg/ITensor/bin/
```
We're done.


### Example Program

Let me now show you how to use the library that you just built in your application. Let's say that you have a `ex.cc` file with a main function which uses ITensor library.

ex.cc
```c++
#include <iostream>
#include <itensor/all.h>

int main(int argc, char** argv)
{
    using namespace itensor;
    Index i("index i", 2);
    Index j("index j", 2);
    ITensor T(i,j), U(i), S, V;
    T.fill(1.0);
    svd(T, U, S, V);
    PrintData(T);
    PrintData(U);
    PrintData(S);
    PrintData(V);
    return 0;
}
```

To compile `ex.cc`, you use the following Makefile:

Makefile
```make
OPENBLAS_ROOT = $(HOME)/.local/pkg/OpenBLAS-v0.2.20-Win64-int32
ITENSOR_ROOT = $(HOME)/.local/pkg/ITensor
LIBS = \
	-static-libgcc -static-libstdc++ \
	-Wl,-Bstatic \
		-litensor \
		-lopenblas \
		-lpthread \
		-lgfortran \
		-lquadmath
CXXFLAGS = \
	-std=c++11 -O3 -Wall \
	-I$(ITENSOR_ROOT)/include \
	-I$(OPENBLAS_ROOT)/include
LDFLAGS = \
	-L$(ITENSOR_ROOT)/lib \
	-L$(OPENBLAS_ROOT)/lib

ex: ex.cc
	$(CXX) $< -o $@ $(CXXFLAGS) $(LDFLAGS) $(LIBS)
```

Typing `make` will create `ex.exe`.

![Example-Static](http://kyungminlee.org/doc/howto/itensor_msys2/example_static_build_run.png)

The resulting binary `ex.exe`, however, has all the dependent libraries (`itensor`, `openblas`, `pthread`, ...) statically linked to it. This will increase size of the binary file and also the build time. To dynamically link the libraries instead, you can remove the flags `-static-libgcc', '-static-libstdc++', and '-Wl,-Bstatic':
```
LIBS = \
	-litensor \
	-lopenblas \
	-lpthread \
	-lgfortran \
	-lquadmath
```
The `ex.exe` built by this Makefile is smaller then before. When you try to run it, however, it will complain about the missing dll files:

![Example-Dynamic-Build](http://kyungminlee.org/doc/howto/itensor_msys2/example_dynamic_build.png)

The simplest way to solve this problem is to copy all the dependent DLL files to the same directory as the executable. The DLL files for ITensor and OpenBLAS can be found in the `bin` subdirectory of each project. Other DLL files can be found in `/mingw64/bin`:

```
$ cp /mingw64/bin/libgcc_s_seh-1.dll /mingw64/bin/libstdc++-6.dll /mingw64/bin/libgfortran-4.dll /mingw64/bin/libwinpthread-1.dll /mingw64/bin/libquadmath-0.dll ~/.local/pkg/ITensor/bin/libitensor*.dll ~/.local/pkg/OpenBLAS-v0.2.20-Win64-int32/bin/libopenblas.dll .
```

Now `ex.exe` is happy.

![Example-Dynamic-Run](http://kyungminlee.org/doc/howto/itensor_msys2/example_dynamic_run.png)

Wait...

You might be thinking, "Why isn't it printing the data? The norms seem correct..." This is because of how static variables defined in header are handled differently in Linux (and Mac OS X) and in Windows; the current version of ITensor is not compatible with how Windows handles it. I won't go into detail, but hopefully this will be fixed soon.
