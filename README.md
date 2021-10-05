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
if test "$HAVE_CXX17" = "1"; then
  AX_INTERFERENCE_SIZES([INTERFERENCE_SIZES])
  if test "$INTERFERENCE_SIZES" = "1"; then
    CXXFLAGS="$CXXFLAGS -DINTERFERENCE_SIZES"
  fi
fi
```
where the variable HAVE_CXX17 is set earlier by the following code:

```sh
dnl If the user did not specify a C++ version.
user_cxx=`echo "$PRESET_CXXFLAGS" | $EGREP -o -E "\-std="`
if test "x$user_cxx" = "x"; then
  dnl Set highest possible C++ support
  AX_CXX_COMPILE_STDCXX(17, [noext], [optional])
  if test "$HAVE_CXX17" = "0"; then
    AX_CXX_COMPILE_STDCXX(14, [noext], [optional])
  elif test "$HAVE_CXX14" = "0"; then
    AX_CXX_COMPILE_STDCXX(11, [noext], [mandatory])
  fi
fi
```

ACTION-IF-FOUND sets the variable INTERFERENCE_SIZES, which is uses for conditional compilation when determining the size of the cache line (with fallback branch):

```c
#ifndef CACHE_LINE_SIZE
#	if __cpp_lib_hardware_interference_size >= 201703 && defined INTERFERENCE_SIZES
#		define CACHE_LINE_SIZE std::hardware_destructive_interference_size
#	elif __cplusplus >= 201402L
#		define CACHE_LINE_SIZE (2 * sizeof(std::max_align_t) & (2 * sizeof(std::max_align_t) - 1)) == 0 ? \
						2 * sizeof(std::max_align_t) : nearestPowerOf2(2 * sizeof(std::max_align_t))
#	else
#		define CACHE_LINE_SIZE 64
#	endif
#endif
```
