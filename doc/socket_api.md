# How to use Berkeley/Posix Socket APIs in WebAssembly

**_Berkeley sockets_** usually means an API for Internet sockets and Unix domain
sockets. A socket is an abstract representation of the local endpoint of a
network communication path.

Currently, WAMR supports a limit set of all well-known functions:
`accept()`, `bind()`, `connect()`, `listen()`, `recv()`, `send()`, `shutdown()`
and `socket()`. Users can call those functions in WebAssembly code directly.
Those WebAssembly socket calls will be dispatched to the imported
functions and eventually will be implemented by host socket APIs.

This document introduces a way to support _Berkeley/Posix Socket APIs_ in
WebAssembly code.

## Patch the native code

The first step is to include a header file of the WAMR socket extension in the
native source code.

```c
#ifdef __wasi__
#include <wasi_socket_ext.h>
#endif
```

`__wasi__` is a Marco defined by WASI. The host compiler will not enable it.

## CMake files

It is recommended that the project should use CMake as its build system. Use
[_wasi-sdk_](https://github.com/WebAssembly/wasi-sdk)
as a toolchain to compile C/C++ to WebAssembly

```bash
$ cmake -DWASI_SDK_PREFIX=${WASI_SDK_DIR}
      -DCMAKE_TOOLCHAIN_FILE=${WASI_TOOLCHAIN_FILE}
      -DCMAKE_SYSROOT=${WASI_SYS_ROOT}
      ..
```

In the *CMakeLists.txt*, include an extension of socket support and link with it.

```cmake
include(${CMAKE_CURRENT_SOURCE_DIR}/../../../core/iwasm/libraries/lib-socket/lib_socket_wasi.cmake)
add_executable(socket_example tcp_server.c)
target_link_libraries(socket_example socket_wasi_ext)
```

Now, the native code with socket APIs is ready for compilation.

## Run with iwasm

If having the _.wasm_, the last step is to run it with _iwasm_.

The _iwasm_ should be compiled with `WAMR_BUILD_LIBC_WASI=1`. By default, it is
enabled.

_iwasm_ accepts address ranges via an option, `--addr-pool`, to implement
the capability control. All IP address the WebAssebmly application may need to `bind()` or `connect()` should be announced first. Every IP address should be in CIRD notation.

```bash
$ iwasm --addr-pool=1.2.3.4/15,2.3.4.6/16 socket_example.wasm
```

Refer to [socket api sample](../samples/socket-api) for more details.
