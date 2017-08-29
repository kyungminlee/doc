## Building ITensor using MSYS2 and MinGW-x64 toolchain on Windows

In this guide I explain how to build ITensor library on Windows. Unfortunately, the current version of ITensor (2.1.0) cannot be built by the lastest version of Microsoft Visual Studio C++ compiler (2017), due to the compiler's lack of support for [Expression SFINAE](http://en.cppreference.com/w/cpp/language/sfinae). We therefore need to use either Cygwin or MinGW compilers on Windows to build ITensor instead. Here I explain how to build ITensor using MinGW-w64 toolchain.


### Install Toolchain

MSYS2 provides a convinent environment for the MinGW-w64 toolchain. MSYS2 can be downloaded from http://www.msys2.org/. Install x86_64 version and follow instructions on the msys2 webpage. After the updates, restart `MSYS2 MinGW 64-bit`, and install the MinGW-w64 toolchain:
```
$ pacman -S mingw-w64-x86_64-toolchain git make vim
```

### Install OpenBLAS

ITensor uses BLAS and LAPACK routines. OpenBLAS is an open source implementation of the BLAS, bundled with the reference LAPACK implementation from netlib by default. Binary packages of OpenBLAS for Windows can normally be downloaded from http://www.openblas.net/. However, the binary package for the most recent version `v0.2.20` is not available for some reason. Previous version `v0.2.19` uses an older version of gfortran, so I recommend building OpenBLAS yourself.

To clone the OpenBLAS repository:
```
$ git clone https://github.com/xianyi/OpenBLAS.git
```
This creates a subdirectory named `OpenBLAS`. After cloning, you can list all the previous releases by the following commands: 
```
$ cd OpenBLAS
$ git tag
```
Checkout the latest release:
```
$ git checkout v0.2.20
```
To build the library, you simply need to `make`:
```
make -j4
```
After the compilation, you can install the files to `~/.local/pkg/OpenBLAS-v0.2.20-Win64-int32` (or any directory of your choice) by the following command:
```
make PREFIX=~/.local/pkg/OpenBLAS-v0.2.20-Win64-int32 install
```
This creates directory `~/.local/pkg/OpenBLAS-v0.2.20-Win64-int32` with subdirectories `include`, `lib`, and `bin`, respectively containing the header files, `.a` files, and a `.dll` file.

### Install ITensor

Now we can build ITensor. To clone the ITensor repository:
```
$ git clone https://github.com/ITensor/ITensor.git
```
ITensor uses `options.mk` for configuration. A sample configuration file `options.mk` is included in the ITensor repository. Copy it to `options.mk` by
```
$ cd ITensor
$ cp options.mk.sample options.mk
```

3. Edit `options.mk` using vim (or any of your favorite editors)
```
$ vim options.mk
```
Make sure you have the following lines uncommented and correct:
```Makefile
CCCOM=g++ -std=c++11 -Wa,-mbig-obj -O2 -fPIC
    ⋮
PLATFORM=openblas
BLAS_LAPACK_LIBFLAGS=-lpthread -L$(HOME)/.local/pkg/OpenBLAS-v0.2.20-Win64-int32/lib -lopenblas
BLAS_LAPACK_INCLUDEFLAGS=-I$(HOME)/.local/pkg/OpenBLAS-v0.2.20-Win64-int32/include \
                       -fpermissive \
                       -DHAVE_LAPACK_CONFIG_H \
                       -DLAPACK_COMPLEX_STRUCTURE
    ⋮
ITENSOR_MAKE_DYLIB=1
```
The compiler flags `-Wa,-mbig-obj -O2` is a workaround for Windows's limitation. Because of the `-O2` flag, even the debug build will be optimized. This will unfortunately interfere with a debugger if you are using one, and make it difficult for you to debug. `ITENSOR_MAKE_DYLIB=1` is for building dynamic library files `libitensor.dll` and `libitensor-g.dll`. Currently, the sample configuration included in ITensor does not support building of `.dll` files. You need to add the following lines to `options.mk`:
```Makefile
DYLIB_EXT=dll
DYLIB_FLAGS=-shared \
            -Wl,--export-all-symbols \
            -Wl,--enable-auto-import \
            -Wl,--out-implib,$@.a
```

After you've changed `options.mk`, you can build ITensor by
```
make -j4
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

* ex.cc

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

* Makefile

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

![Example](http://kyungminlee.org/doc/howto/itensor_msys2/run_ex.png)
