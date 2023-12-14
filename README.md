# GPU Macros

Defines C++ macros to write GPU code that compiles both on CUDA and ROCm.

## How to use

You must add the files in the `src` directory to your project, and include the `gpu_macros.hpp` (and any other library-specific file)  wherever you want to make calls to the HIP or CUDA runtime.

The `__GPU__` macro is defined when the source code is being compiled with NVCC or HIPCC. This provides a way for you to guard GPU-enabled code, and allow CPU only builds when compiling with a host compiler (e.g. GCC).

Here is an example:

```
#ifndef __JONES_MATRIX_H__
#define __JONES_MATRIX_H__

#include <complex>
#include <iostream>
#include "gpu_macros.hpp"


template <typename T>
class JonesMatrix {
    public:

    [...]

    #ifdef __GPU__
    __host__ __device__
    #endif
    JonesMatrix operator*(const JonesMatrix& other) const {
        [...]
    }
    [...]
```

## Build with CMake

You can handle the CUDA and HIP build with the same `CMakeLists.txt` file.

```
# Define CMake options to choose the backend
option(USE_CUDA "Compile the code with NVIDIA GPU support." OFF)
option(USE_HIP "Compile the code with AMD GPU support." OFF)

# Enable CMake support for target language
# I suspect we can so something similar with HIP, with the latest CMake, bur for now
# what I do is simply specifying the -DCMAKE_CXX_COMPILER=hipcc command line option.
if(USE_CUDA)
enable_language(CUDA CXX)
endif()

# We do not want to use `.cu` files because hipcc won't be able to process them.
# So we tell nvcc to treat `.cpp` files as CUDA code (CUDA is a superset of C++ anyway).

if(USE_CUDA)
set_source_files_properties( ${sources} ${headers} ${apps} ${tests} PROPERTIES LANGUAGE CUDA)
# this may be needed for older CUDA versions
add_compile_options("--expt-relaxed-constexpr")
endif()

```
