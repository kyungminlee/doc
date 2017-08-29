## How to build ITensor using MSYS2 and MinGW-x64 toolchain 

### Install Toolchain

1. Download and install msys2 from http://www.msys2.org/. Install x86_64 version.

2. Start MSYS2 MinGW 64-bit and update the package database and core system packages:
    ```
    pacman -Syu
    ```

3. Restart MSYS2 MinGW 64-bit, and install the toolchain:
    ```
    pacman -S mingw-w64-x86_64-toolchain git make vim
    ```

### Install OpenBLAS

1. Clone OpenBLAS from github:
    ```
    git clone https://github.com/xianyi/OpenBLAS.git
    ```

2. Check what the latest release is:
    ```
    cd OpenBLAS
    git tag
    ```

3. Checkout the latest release and build:
    ```
    git checkout v0.2.20
    make -j4
    make PREFIX=~/.local/pkg/OpenBLAS-v0.2.20 install
    ```

### Install ITensor

1. Clone the ITensor repository.
    ```
    git clone https://github.com/ITensor/ITensor.git
    ```

2. Create the configuration file
    ```
    cp options.mk.sample options.mk
    ```

3. Configure `options.mk`. Pay attention to the following lines:
    ```Makefile
    CCCOM=g++ -std=c++11 -Wa,-mbig-obj -O2 -fPIC
        ⋮
    PLATFORM=openblas
    BLAS_LAPACK_LIBFLAGS=-lpthread -L$(HOME)/.local/pkg/OpenBLAS-v0.2.20/lib -lopenblas
    BLAS_LAPACK_INCLUDEFLAGS=-I$(HOME)/.local/pkg/OpenBLAS-v0.2.20/include \
                           -fpermissive \
                           -DHAVE_LAPACK_CONFIG_H \
                           -DLAPACK_COMPLEX_STRUCTURE
        ⋮
    ITENSOR_MAKE_DYLIB=1
    ```
    Also add
    ```Makefile
    DYLIB_EXT=dll
    DYLIB_FLAGS=-shared \
                -Wl,--export-all-symbols \
                -Wl,--enable-auto-import \
                -Wl,--out-implib,$@.a
    ```

4. Build
    ```
    make -j4
    ```

5. Install
    ```
    mkdir -p ~/.local/pkg/ITensor/include
    mkdir -p ~/.local/pkg/ITensor/lib
    mkdir -p ~/.local/pkg/ITensor/bin
    cp -r itensor ~/.local/pkg/ITensor/include/
    cp -r lib/*.a ~/.local/pkg/ITensor/lib/
    cp -r lib/*.dll ~/.local/pkg/ITensor/bin/
    ```

### Example Program

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
        -I$(OPENBLAS_ROOT)/include \
        -I$(ITENSOR_ROOT)/include
    LDFLAGS = \
        -L$(ITENSOR_ROOT)/lib \
        -L$(OPENBLAS_ROOT)/lib

    ex: ex.cc
    	$(CXX) $< -o $@ $(CXXFLAGS) $(LDFLAGS) $(LIBS)
    ```

* ex.c
    ```c++
    #include <iostream>
    #include <itensor/all.h>

    int main(int argc, char** argv)
    {
        using namespace itensor;
        Index i("index i", 3);
        Index j("index j", 4);
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
* Result
    ![Example](http://kyungminlee.org/doc/howto/itensor_msys/run_ex.png)
