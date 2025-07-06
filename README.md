# check-interference-sizes
[![License](https://img.shields.io/badge/License-GPL%20v2-blue.svg)](https://github.com/yvoinov/check-interference-sizes/blob/main/LICENSE)

## M4 macro for autoconf to check C++ interference sizes

Unfortunately, many new features of the C ++ standard have not yet been implemented by compilers.

Moreover, compiler check macros often erroneously report the presence of a feature, but there is no implementation.

According to [this article](https://stackoverflow.com/questions/67999444/no-member-named-hardware-constructive-interference-size-in-namespace-std), in order to check the possibility of using this feature during configuration, before compilation, an M4 macro was implemented that performs this check. 

Macro set specified environment variable to 1 when compilation and run successful (i.e. interference sizes defined and implemented) and 0 otherwise.

To use, add a macro to configure.ac as shown below (for C++ code):

```sh
dnl Check if interference_sizes are defined
AX_INTERFERENCE_SIZES([INTERFERENCE_SIZES])
if test "$INTERFERENCE_SIZES" = "1"; then
  CXXFLAGS="$CXXFLAGS -DINTERFERENCE_SIZES"
fi
```

ACTION-IF-FOUND sets the variable INTERFERENCE_SIZES, which is uses for conditional compilation when determining the size of the cache line:

```c
#ifndef MY_CACHE_LINE_SIZE
#	if __cpp_lib_hardware_interference_size >= 201703 && defined INTERFERENCE_SIZES
#		define MY_CACHE_LINE_SIZE std::hardware_destructive_interference_size
#	else
#		define MY_CACHE_LINE_SIZE (((2 * sizeof(std::max_align_t)) & ((2 * sizeof(std::max_align_t)) - 1)) == 0 ? \
						(2 * sizeof(std::max_align_t)) : \
						nearestPowerof2(2 * sizeof(std::max_align_t)))
#	endif
#endif
```
