gRPC C++ - Building from source
===========================

This document has detailed instructions on how to build gRPC C++ from source. Note that it only covers the build of gRPC itself and is mostly meant for gRPC C++ contributors and/or power users.
Other should follow the user instructions. See the [How to use](https://github.com/grpc/grpc/tree/master/src/cpp#to-start-using-grpc-c) instructions for guidance on how to add gRPC as a dependency to a C++ application (there are several ways and system wide installation is often not the best choice).

# Pre-requisites

## Linux

```sh
 $ [sudo] apt-get install build-essential autoconf libtool pkg-config
```

If you plan to build from source and run tests, install the following as well:
```sh
 $ [sudo] apt-get install libgflags-dev libgtest-dev
 $ [sudo] apt-get install clang-5.0 libc++-dev
```
Before building, you need to clone the gRPC github repository and download submodules containing source code
for gRPC's dependencies (that's done by the `submodule` command or `--recursive` flag). The following commands will clone the gRPC
repository at the latest stable version.

## Unix

```sh
 $ git clone -b $(curl -L https://grpc.io/release) https://github.com/grpc/grpc
 $ cd grpc
 $ git submodule update --init
 ```

# Build from source

In the C++ world, there's no "standard" build system that would work for in all supported use cases and on all supported platforms.
Therefore, gRPC supports several major build systems, which should satisfy most users. Depending on your needs
we recommend building using `bazel` or `cmake`.

## Building with bazel (recommended)

Bazel is the primary build system for gRPC C++ and if you're comfortable with using bazel, we can certainly recommend it.
Using bazel will give you the best developer experience as well as faster and cleaner builds.

You'll need `bazel` version `1.0.0` or higher to build gRPC.
See [Installing Bazel](https://docs.bazel.build/versions/master/install.html) for instructions how to install bazel on your system.
We support building with `bazel` on Linux, MacOS and Windows.

From the grpc repository root
```
# Build gRPC C++
$ bazel build :all
```

```
# Run all the C/C++ tests
$ bazel test --config=dbg //test/...
```

NOTE: If you are gRPC maintainer and you have access to our test cluster, you should use the our [gRPC's Remote Execution environment](tools/remote_build/README.md)
to get significant improvement to the build and test speed (and a bunch of other very useful features).

## Building with CMake

### Linux/Unix, Using Make

Run from grpc directory after cloning the repo with --recursive or updating submodules.
```
$ mkdir -p cmake/build
$ cd cmake/build
$ cmake ../..
$ make
```

If you want to build shared libraries (`.so` files), run `cmake` with `-DBUILD_SHARED_LIBS=ON`.

### Dependency management

gRPC's CMake build system provides two modes for handling dependencies.
* module - build dependencies alongside gRPC.
* package - use external copies of dependencies that are already available
on your system.

This behavior is controlled by the `gRPC_<depname>_PROVIDER` CMake variables,
ie `gRPC_CARES_PROVIDER`.

### Install after build

Perform the following steps to install gRPC using CMake.
* Set `gRPC_INSTALL` to `ON`
* Build the `install` target

The install destination is controlled by the
[`CMAKE_INSTALL_PREFIX`](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html) variable.

If you are running CMake v3.13 or newer you can build gRPC's dependencies
in "module" mode and install them alongside gRPC in a single step.
[Example](test/distrib/cpp/run_distrib_test_cmake_module_install.sh)

If you are using an older version of gRPC, you will need to select "package"
mode (rather than "module" mode) for the dependencies.
This means you will need to have external copies of these libraries available
on your system.
```
$ cmake .. -DgRPC_CARES_PROVIDER=package    \
           -DgRPC_PROTOBUF_PROVIDER=package \
           -DgRPC_SSL_PROVIDER=package      \
           -DgRPC_ZLIB_PROVIDER=package
$ make
$ make install
```
[Example](test/distrib/cpp/run_distrib_test_cmake.sh)

### Cross-compiling

You can use CMake to cross-compile gRPC for another architecture. In order to
do so, you will first need to build `protoc` and `grpc_cpp_plugin`
for the host architecture. These tools are used during the build of gRPC, so
we need copies of executables that can be run natively.

You will likely need to install the toolchain for the platform you are
targeting for your cross-compile. Once you have done so, you can write a
toolchain file to tell CMake where to find the compilers and system tools
that will be used for this build.

This toolchain file is specified to CMake by setting the `CMAKE_TOOLCHAIN_FILE`
variable.
```
$ cmake .. -DCMAKE_TOOLCHAIN_FILE=path/to/file
$ make
```

[Cross-compile example](test/distrib/cpp/run_distrib_test_raspberry_pi.sh)

## Building with make on UNIX systems (deprecated)

NOTE: `make` used to be gRPC's default build system, but we're no longer recommending it. You should use `bazel` or `cmake` instead. The `Makefile` is only intended for internal usage and is not meant for public consumption.

From the grpc repository root
```sh
 $ make
```

NOTE: if you get an error on linux such as 'aclocal-1.15: command not found', which can happen if you ran 'make' before installing the pre-reqs, try the following:
```sh
$ git clean -f -d -x && git submodule foreach --recursive git clean -f -d -x
$ [sudo] apt-get install build-essential autoconf libtool pkg-config
$ make
```

### A note on `protoc`

By default gRPC uses [protocol buffers](https://github.com/google/protobuf),
you will need the `protoc` compiler to generate stub server and client code.

If you compile gRPC from source, as described below, the Makefile will
automatically try compiling the `protoc` in third_party if you cloned the
repository recursively and it detects that you do not already have 'protoc' compiler
installed.

```
Install the project...
-- Install configuration: ""
-- Installing: /usr/local/include/grpc/support/alloc.h
-- Installing: /usr/local/include/grpc/support/atm.h
-- Installing: /usr/local/include/grpc/support/atm_gcc_atomic.h
-- Installing: /usr/local/include/grpc/support/atm_gcc_sync.h
-- Installing: /usr/local/include/grpc/support/atm_windows.h
-- Installing: /usr/local/include/grpc/support/cpu.h
-- Installing: /usr/local/include/grpc/support/log.h
-- Installing: /usr/local/include/grpc/support/log_windows.h
-- Installing: /usr/local/include/grpc/support/port_platform.h
-- Installing: /usr/local/include/grpc/support/string_util.h
-- Installing: /usr/local/include/grpc/support/sync.h
-- Installing: /usr/local/include/grpc/support/sync_custom.h
-- Installing: /usr/local/include/grpc/support/sync_generic.h
-- Installing: /usr/local/include/grpc/support/sync_posix.h
-- Installing: /usr/local/include/grpc/support/sync_windows.h
-- Installing: /usr/local/include/grpc/support/thd_id.h
-- Installing: /usr/local/include/grpc/support/time.h
-- Installing: /usr/local/include/grpc/impl/codegen/atm.h
-- Installing: /usr/local/include/grpc/impl/codegen/atm_gcc_atomic.h
-- Installing: /usr/local/include/grpc/impl/codegen/atm_gcc_sync.h
-- Installing: /usr/local/include/grpc/impl/codegen/atm_windows.h
-- Installing: /usr/local/include/grpc/impl/codegen/fork.h
-- Installing: /usr/local/include/grpc/impl/codegen/gpr_slice.h
-- Installing: /usr/local/include/grpc/impl/codegen/gpr_types.h
-- Installing: /usr/local/include/grpc/impl/codegen/log.h
-- Installing: /usr/local/include/grpc/impl/codegen/port_platform.h
-- Installing: /usr/local/include/grpc/impl/codegen/sync.h
-- Installing: /usr/local/include/grpc/impl/codegen/sync_custom.h
-- Installing: /usr/local/include/grpc/impl/codegen/sync_generic.h
-- Installing: /usr/local/include/grpc/impl/codegen/sync_posix.h
-- Installing: /usr/local/include/grpc/impl/codegen/sync_windows.h
-- Installing: /usr/local/include/grpc/impl/codegen/byte_buffer.h
-- Installing: /usr/local/include/grpc/impl/codegen/byte_buffer_reader.h
-- Installing: /usr/local/include/grpc/impl/codegen/compression_types.h
-- Installing: /usr/local/include/grpc/impl/codegen/connectivity_state.h
-- Installing: /usr/local/include/grpc/impl/codegen/grpc_types.h
-- Installing: /usr/local/include/grpc/impl/codegen/propagation_bits.h
-- Installing: /usr/local/include/grpc/impl/codegen/slice.h
-- Installing: /usr/local/include/grpc/impl/codegen/status.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_atomic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_windows.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/fork.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/log.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/port_platform.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_custom.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_generic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_posix.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_windows.h
-- Installing: /usr/local/include/grpc/grpc_security.h
-- Installing: /usr/local/include/grpc/byte_buffer.h
-- Installing: /usr/local/include/grpc/byte_buffer_reader.h
-- Installing: /usr/local/include/grpc/compression.h
-- Installing: /usr/local/include/grpc/fork.h
-- Installing: /usr/local/include/grpc/grpc.h
-- Installing: /usr/local/include/grpc/grpc_posix.h
-- Installing: /usr/local/include/grpc/grpc_security_constants.h
-- Installing: /usr/local/include/grpc/load_reporting.h
-- Installing: /usr/local/include/grpc/slice.h
-- Installing: /usr/local/include/grpc/slice_buffer.h
-- Installing: /usr/local/include/grpc/status.h
-- Installing: /usr/local/include/grpc/support/workaround_list.h
-- Installing: /usr/local/include/grpc/census.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/byte_buffer_reader.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/compression_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/connectivity_state.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/grpc_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/propagation_bits.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/status.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_atomic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_windows.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/fork.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/log.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/port_platform.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_custom.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_generic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_posix.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_windows.h
-- Installing: /usr/local/include/grpc/grpc_cronet.h
-- Up-to-date: /usr/local/include/grpc/grpc_security.h
-- Up-to-date: /usr/local/include/grpc/grpc_security_constants.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/byte_buffer_reader.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/compression_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/connectivity_state.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/grpc_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/propagation_bits.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/status.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_atomic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_windows.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/fork.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/log.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/port_platform.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_custom.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_generic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_posix.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_windows.h
-- Up-to-date: /usr/local/include/grpc/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc/byte_buffer_reader.h
-- Up-to-date: /usr/local/include/grpc/compression.h
-- Up-to-date: /usr/local/include/grpc/fork.h
-- Up-to-date: /usr/local/include/grpc/grpc.h
-- Up-to-date: /usr/local/include/grpc/grpc_posix.h
-- Up-to-date: /usr/local/include/grpc/grpc_security_constants.h
-- Up-to-date: /usr/local/include/grpc/load_reporting.h
-- Up-to-date: /usr/local/include/grpc/slice.h
-- Up-to-date: /usr/local/include/grpc/slice_buffer.h
-- Up-to-date: /usr/local/include/grpc/status.h
-- Up-to-date: /usr/local/include/grpc/support/workaround_list.h
-- Up-to-date: /usr/local/include/grpc/census.h
-- Installing: /usr/local/include/grpc++/alarm.h
-- Installing: /usr/local/include/grpc++/channel.h
-- Installing: /usr/local/include/grpc++/client_context.h
-- Installing: /usr/local/include/grpc++/completion_queue.h
-- Installing: /usr/local/include/grpc++/create_channel.h
-- Installing: /usr/local/include/grpc++/create_channel_posix.h
-- Installing: /usr/local/include/grpc++/ext/health_check_service_server_builder_option.h
-- Installing: /usr/local/include/grpc++/generic/async_generic_service.h
-- Installing: /usr/local/include/grpc++/generic/generic_stub.h
-- Installing: /usr/local/include/grpc++/grpc++.h
-- Installing: /usr/local/include/grpc++/health_check_service_interface.h
-- Installing: /usr/local/include/grpc++/impl/call.h
-- Installing: /usr/local/include/grpc++/impl/channel_argument_option.h
-- Installing: /usr/local/include/grpc++/impl/client_unary_call.h
-- Installing: /usr/local/include/grpc++/impl/codegen/core_codegen.h
-- Installing: /usr/local/include/grpc++/impl/grpc_library.h
-- Installing: /usr/local/include/grpc++/impl/method_handler_impl.h
-- Installing: /usr/local/include/grpc++/impl/rpc_method.h
-- Installing: /usr/local/include/grpc++/impl/rpc_service_method.h
-- Installing: /usr/local/include/grpc++/impl/serialization_traits.h
-- Installing: /usr/local/include/grpc++/impl/server_builder_option.h
-- Installing: /usr/local/include/grpc++/impl/server_builder_plugin.h
-- Installing: /usr/local/include/grpc++/impl/server_initializer.h
-- Installing: /usr/local/include/grpc++/impl/service_type.h
-- Installing: /usr/local/include/grpc++/resource_quota.h
-- Installing: /usr/local/include/grpc++/security/auth_context.h
-- Installing: /usr/local/include/grpc++/security/auth_metadata_processor.h
-- Installing: /usr/local/include/grpc++/security/credentials.h
-- Installing: /usr/local/include/grpc++/security/server_credentials.h
-- Installing: /usr/local/include/grpc++/server.h
-- Installing: /usr/local/include/grpc++/server_builder.h
-- Installing: /usr/local/include/grpc++/server_context.h
-- Installing: /usr/local/include/grpc++/server_posix.h
-- Installing: /usr/local/include/grpc++/support/async_stream.h
-- Installing: /usr/local/include/grpc++/support/async_unary_call.h
-- Installing: /usr/local/include/grpc++/support/byte_buffer.h
-- Installing: /usr/local/include/grpc++/support/channel_arguments.h
-- Installing: /usr/local/include/grpc++/support/config.h
-- Installing: /usr/local/include/grpc++/support/slice.h
-- Installing: /usr/local/include/grpc++/support/status.h
-- Installing: /usr/local/include/grpc++/support/status_code_enum.h
-- Installing: /usr/local/include/grpc++/support/string_ref.h
-- Installing: /usr/local/include/grpc++/support/stub_options.h
-- Installing: /usr/local/include/grpc++/support/sync_stream.h
-- Installing: /usr/local/include/grpc++/support/time.h
-- Installing: /usr/local/include/grpcpp/alarm.h
-- Installing: /usr/local/include/grpcpp/alarm_impl.h
-- Installing: /usr/local/include/grpcpp/channel.h
-- Installing: /usr/local/include/grpcpp/channel_impl.h
-- Installing: /usr/local/include/grpcpp/client_context.h
-- Installing: /usr/local/include/grpcpp/completion_queue.h
-- Installing: /usr/local/include/grpcpp/completion_queue_impl.h
-- Installing: /usr/local/include/grpcpp/create_channel.h
-- Installing: /usr/local/include/grpcpp/create_channel_impl.h
-- Installing: /usr/local/include/grpcpp/create_channel_posix.h
-- Installing: /usr/local/include/grpcpp/create_channel_posix_impl.h
-- Installing: /usr/local/include/grpcpp/ext/health_check_service_server_builder_option.h
-- Installing: /usr/local/include/grpcpp/generic/async_generic_service.h
-- Installing: /usr/local/include/grpcpp/generic/generic_stub.h
-- Installing: /usr/local/include/grpcpp/generic/generic_stub_impl.h
-- Installing: /usr/local/include/grpcpp/grpcpp.h
-- Installing: /usr/local/include/grpcpp/health_check_service_interface.h
-- Installing: /usr/local/include/grpcpp/health_check_service_interface_impl.h
-- Installing: /usr/local/include/grpcpp/impl/call.h
-- Installing: /usr/local/include/grpcpp/impl/channel_argument_option.h
-- Installing: /usr/local/include/grpcpp/impl/client_unary_call.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/core_codegen.h
-- Installing: /usr/local/include/grpcpp/impl/grpc_library.h
-- Installing: /usr/local/include/grpcpp/impl/method_handler_impl.h
-- Installing: /usr/local/include/grpcpp/impl/rpc_method.h
-- Installing: /usr/local/include/grpcpp/impl/rpc_service_method.h
-- Installing: /usr/local/include/grpcpp/impl/serialization_traits.h
-- Installing: /usr/local/include/grpcpp/impl/server_builder_option.h
-- Installing: /usr/local/include/grpcpp/impl/server_builder_option_impl.h
-- Installing: /usr/local/include/grpcpp/impl/server_builder_plugin.h
-- Installing: /usr/local/include/grpcpp/impl/server_initializer.h
-- Installing: /usr/local/include/grpcpp/impl/server_initializer_impl.h
-- Installing: /usr/local/include/grpcpp/impl/service_type.h
-- Installing: /usr/local/include/grpcpp/resource_quota.h
-- Installing: /usr/local/include/grpcpp/resource_quota_impl.h
-- Installing: /usr/local/include/grpcpp/security/auth_context.h
-- Installing: /usr/local/include/grpcpp/security/auth_metadata_processor.h
-- Installing: /usr/local/include/grpcpp/security/auth_metadata_processor_impl.h
-- Installing: /usr/local/include/grpcpp/security/credentials.h
-- Installing: /usr/local/include/grpcpp/security/credentials_impl.h
-- Installing: /usr/local/include/grpcpp/security/server_credentials.h
-- Installing: /usr/local/include/grpcpp/security/server_credentials_impl.h
-- Installing: /usr/local/include/grpcpp/security/tls_credentials_options.h
-- Installing: /usr/local/include/grpcpp/server.h
-- Installing: /usr/local/include/grpcpp/server_builder.h
-- Installing: /usr/local/include/grpcpp/server_builder_impl.h
-- Installing: /usr/local/include/grpcpp/server_context.h
-- Installing: /usr/local/include/grpcpp/server_impl.h
-- Installing: /usr/local/include/grpcpp/server_posix.h
-- Installing: /usr/local/include/grpcpp/server_posix_impl.h
-- Installing: /usr/local/include/grpcpp/support/async_stream.h
-- Installing: /usr/local/include/grpcpp/support/async_stream_impl.h
-- Installing: /usr/local/include/grpcpp/support/async_unary_call.h
-- Installing: /usr/local/include/grpcpp/support/async_unary_call_impl.h
-- Installing: /usr/local/include/grpcpp/support/byte_buffer.h
-- Installing: /usr/local/include/grpcpp/support/channel_arguments.h
-- Installing: /usr/local/include/grpcpp/support/channel_arguments_impl.h
-- Installing: /usr/local/include/grpcpp/support/client_callback.h
-- Installing: /usr/local/include/grpcpp/support/client_callback_impl.h
-- Installing: /usr/local/include/grpcpp/support/client_interceptor.h
-- Installing: /usr/local/include/grpcpp/support/config.h
-- Installing: /usr/local/include/grpcpp/support/interceptor.h
-- Installing: /usr/local/include/grpcpp/support/message_allocator.h
-- Installing: /usr/local/include/grpcpp/support/proto_buffer_reader.h
-- Installing: /usr/local/include/grpcpp/support/proto_buffer_writer.h
-- Installing: /usr/local/include/grpcpp/support/server_callback.h
-- Installing: /usr/local/include/grpcpp/support/server_callback_impl.h
-- Installing: /usr/local/include/grpcpp/support/server_interceptor.h
-- Installing: /usr/local/include/grpcpp/support/slice.h
-- Installing: /usr/local/include/grpcpp/support/status.h
-- Installing: /usr/local/include/grpcpp/support/status_code_enum.h
-- Installing: /usr/local/include/grpcpp/support/string_ref.h
-- Installing: /usr/local/include/grpcpp/support/stub_options.h
-- Installing: /usr/local/include/grpcpp/support/sync_stream.h
-- Installing: /usr/local/include/grpcpp/support/sync_stream_impl.h
-- Installing: /usr/local/include/grpcpp/support/time.h
-- Installing: /usr/local/include/grpcpp/support/validate_service_config.h
-- Up-to-date: /usr/local/include/grpc/support/alloc.h
-- Up-to-date: /usr/local/include/grpc/support/atm.h
-- Up-to-date: /usr/local/include/grpc/support/atm_gcc_atomic.h
-- Up-to-date: /usr/local/include/grpc/support/atm_gcc_sync.h
-- Up-to-date: /usr/local/include/grpc/support/atm_windows.h
-- Up-to-date: /usr/local/include/grpc/support/cpu.h
-- Up-to-date: /usr/local/include/grpc/support/log.h
-- Up-to-date: /usr/local/include/grpc/support/log_windows.h
-- Up-to-date: /usr/local/include/grpc/support/port_platform.h
-- Up-to-date: /usr/local/include/grpc/support/string_util.h
-- Up-to-date: /usr/local/include/grpc/support/sync.h
-- Up-to-date: /usr/local/include/grpc/support/sync_custom.h
-- Up-to-date: /usr/local/include/grpc/support/sync_generic.h
-- Up-to-date: /usr/local/include/grpc/support/sync_posix.h
-- Up-to-date: /usr/local/include/grpc/support/sync_windows.h
-- Up-to-date: /usr/local/include/grpc/support/thd_id.h
-- Up-to-date: /usr/local/include/grpc/support/time.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_atomic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_windows.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/fork.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/log.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/port_platform.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_custom.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_generic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_posix.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_windows.h
-- Up-to-date: /usr/local/include/grpc/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc/byte_buffer_reader.h
-- Up-to-date: /usr/local/include/grpc/compression.h
-- Up-to-date: /usr/local/include/grpc/fork.h
-- Up-to-date: /usr/local/include/grpc/grpc.h
-- Up-to-date: /usr/local/include/grpc/grpc_posix.h
-- Up-to-date: /usr/local/include/grpc/grpc_security_constants.h
-- Up-to-date: /usr/local/include/grpc/load_reporting.h
-- Up-to-date: /usr/local/include/grpc/slice.h
-- Up-to-date: /usr/local/include/grpc/slice_buffer.h
-- Up-to-date: /usr/local/include/grpc/status.h
-- Up-to-date: /usr/local/include/grpc/support/workaround_list.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/byte_buffer_reader.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/compression_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/connectivity_state.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/grpc_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/propagation_bits.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/status.h
-- Installing: /usr/local/include/grpc++/impl/codegen/async_stream.h
-- Installing: /usr/local/include/grpc++/impl/codegen/async_unary_call.h
-- Installing: /usr/local/include/grpc++/impl/codegen/byte_buffer.h
-- Installing: /usr/local/include/grpc++/impl/codegen/call.h
-- Installing: /usr/local/include/grpc++/impl/codegen/call_hook.h
-- Installing: /usr/local/include/grpc++/impl/codegen/channel_interface.h
-- Installing: /usr/local/include/grpc++/impl/codegen/client_context.h
-- Installing: /usr/local/include/grpc++/impl/codegen/client_unary_call.h
-- Installing: /usr/local/include/grpc++/impl/codegen/completion_queue.h
-- Installing: /usr/local/include/grpc++/impl/codegen/completion_queue_tag.h
-- Installing: /usr/local/include/grpc++/impl/codegen/config.h
-- Installing: /usr/local/include/grpc++/impl/codegen/core_codegen_interface.h
-- Installing: /usr/local/include/grpc++/impl/codegen/create_auth_context.h
-- Installing: /usr/local/include/grpc++/impl/codegen/grpc_library.h
-- Installing: /usr/local/include/grpc++/impl/codegen/metadata_map.h
-- Installing: /usr/local/include/grpc++/impl/codegen/method_handler_impl.h
-- Installing: /usr/local/include/grpc++/impl/codegen/rpc_method.h
-- Installing: /usr/local/include/grpc++/impl/codegen/rpc_service_method.h
-- Installing: /usr/local/include/grpc++/impl/codegen/security/auth_context.h
-- Installing: /usr/local/include/grpc++/impl/codegen/serialization_traits.h
-- Installing: /usr/local/include/grpc++/impl/codegen/server_context.h
-- Installing: /usr/local/include/grpc++/impl/codegen/server_interface.h
-- Installing: /usr/local/include/grpc++/impl/codegen/service_type.h
-- Installing: /usr/local/include/grpc++/impl/codegen/slice.h
-- Installing: /usr/local/include/grpc++/impl/codegen/status.h
-- Installing: /usr/local/include/grpc++/impl/codegen/status_code_enum.h
-- Installing: /usr/local/include/grpc++/impl/codegen/string_ref.h
-- Installing: /usr/local/include/grpc++/impl/codegen/stub_options.h
-- Installing: /usr/local/include/grpc++/impl/codegen/sync_stream.h
-- Installing: /usr/local/include/grpc++/impl/codegen/time.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/async_generic_service.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/async_stream.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/async_stream_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/async_unary_call.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/async_unary_call_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/byte_buffer.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/call.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/call_hook.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/call_op_set.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/call_op_set_interface.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/callback_common.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/channel_interface.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/client_callback.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/client_callback_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/client_context.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/client_context_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/client_interceptor.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/client_unary_call.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/completion_queue.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/completion_queue_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/completion_queue_tag.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/config.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/core_codegen_interface.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/create_auth_context.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/delegating_channel.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/grpc_library.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/intercepted_channel.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/interceptor.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/interceptor_common.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/message_allocator.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/metadata_map.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/method_handler.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/method_handler_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/rpc_method.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/rpc_service_method.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/security/auth_context.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/serialization_traits.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/server_callback.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/server_callback_handlers.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/server_callback_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/server_context.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/server_context_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/server_interceptor.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/server_interface.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/service_type.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/slice.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/status.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/status_code_enum.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/string_ref.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/stub_options.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/sync_stream.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/sync_stream_impl.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/time.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/sync.h
-- Installing: /usr/local/include/grpc++/impl/codegen/proto_utils.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/proto_buffer_reader.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/proto_buffer_writer.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/proto_utils.h
-- Installing: /usr/local/include/grpc++/impl/codegen/config_protobuf.h
-- Installing: /usr/local/include/grpcpp/impl/codegen/config_protobuf.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/security/auth_context.h
-- Installing: /usr/local/include/grpcpp/security/alts_context.h
-- Installing: /usr/local/include/grpcpp/security/alts_util.h
-- Installing: /usr/local/include/grpc++/support/error_details.h
-- Installing: /usr/local/include/grpcpp/support/error_details.h
-- Installing: /usr/local/include/grpcpp/support/error_details_impl.h
-- Installing: /usr/local/include/grpc++/ext/proto_server_reflection_plugin.h
-- Installing: /usr/local/include/grpcpp/ext/proto_server_reflection_plugin.h
-- Installing: /usr/local/include/grpcpp/ext/proto_server_reflection_plugin_impl.h
-- Up-to-date: /usr/local/include/grpc++/alarm.h
-- Up-to-date: /usr/local/include/grpc++/channel.h
-- Up-to-date: /usr/local/include/grpc++/client_context.h
-- Up-to-date: /usr/local/include/grpc++/completion_queue.h
-- Up-to-date: /usr/local/include/grpc++/create_channel.h
-- Up-to-date: /usr/local/include/grpc++/create_channel_posix.h
-- Up-to-date: /usr/local/include/grpc++/ext/health_check_service_server_builder_option.h
-- Up-to-date: /usr/local/include/grpc++/generic/async_generic_service.h
-- Up-to-date: /usr/local/include/grpc++/generic/generic_stub.h
-- Up-to-date: /usr/local/include/grpc++/grpc++.h
-- Up-to-date: /usr/local/include/grpc++/health_check_service_interface.h
-- Up-to-date: /usr/local/include/grpc++/impl/call.h
-- Up-to-date: /usr/local/include/grpc++/impl/channel_argument_option.h
-- Up-to-date: /usr/local/include/grpc++/impl/client_unary_call.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/core_codegen.h
-- Up-to-date: /usr/local/include/grpc++/impl/grpc_library.h
-- Up-to-date: /usr/local/include/grpc++/impl/method_handler_impl.h
-- Up-to-date: /usr/local/include/grpc++/impl/rpc_method.h
-- Up-to-date: /usr/local/include/grpc++/impl/rpc_service_method.h
-- Up-to-date: /usr/local/include/grpc++/impl/serialization_traits.h
-- Up-to-date: /usr/local/include/grpc++/impl/server_builder_option.h
-- Up-to-date: /usr/local/include/grpc++/impl/server_builder_plugin.h
-- Up-to-date: /usr/local/include/grpc++/impl/server_initializer.h
-- Up-to-date: /usr/local/include/grpc++/impl/service_type.h
-- Up-to-date: /usr/local/include/grpc++/resource_quota.h
-- Up-to-date: /usr/local/include/grpc++/security/auth_context.h
-- Up-to-date: /usr/local/include/grpc++/security/auth_metadata_processor.h
-- Up-to-date: /usr/local/include/grpc++/security/credentials.h
-- Up-to-date: /usr/local/include/grpc++/security/server_credentials.h
-- Up-to-date: /usr/local/include/grpc++/server.h
-- Up-to-date: /usr/local/include/grpc++/server_builder.h
-- Up-to-date: /usr/local/include/grpc++/server_context.h
-- Up-to-date: /usr/local/include/grpc++/server_posix.h
-- Up-to-date: /usr/local/include/grpc++/support/async_stream.h
-- Up-to-date: /usr/local/include/grpc++/support/async_unary_call.h
-- Up-to-date: /usr/local/include/grpc++/support/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc++/support/channel_arguments.h
-- Up-to-date: /usr/local/include/grpc++/support/config.h
-- Up-to-date: /usr/local/include/grpc++/support/slice.h
-- Up-to-date: /usr/local/include/grpc++/support/status.h
-- Up-to-date: /usr/local/include/grpc++/support/status_code_enum.h
-- Up-to-date: /usr/local/include/grpc++/support/string_ref.h
-- Up-to-date: /usr/local/include/grpc++/support/stub_options.h
-- Up-to-date: /usr/local/include/grpc++/support/sync_stream.h
-- Up-to-date: /usr/local/include/grpc++/support/time.h
-- Up-to-date: /usr/local/include/grpcpp/alarm.h
-- Up-to-date: /usr/local/include/grpcpp/alarm_impl.h
-- Up-to-date: /usr/local/include/grpcpp/channel.h
-- Up-to-date: /usr/local/include/grpcpp/channel_impl.h
-- Up-to-date: /usr/local/include/grpcpp/client_context.h
-- Up-to-date: /usr/local/include/grpcpp/completion_queue.h
-- Up-to-date: /usr/local/include/grpcpp/completion_queue_impl.h
-- Up-to-date: /usr/local/include/grpcpp/create_channel.h
-- Up-to-date: /usr/local/include/grpcpp/create_channel_impl.h
-- Up-to-date: /usr/local/include/grpcpp/create_channel_posix.h
-- Up-to-date: /usr/local/include/grpcpp/create_channel_posix_impl.h
-- Up-to-date: /usr/local/include/grpcpp/ext/health_check_service_server_builder_option.h
-- Up-to-date: /usr/local/include/grpcpp/generic/async_generic_service.h
-- Up-to-date: /usr/local/include/grpcpp/generic/generic_stub.h
-- Up-to-date: /usr/local/include/grpcpp/generic/generic_stub_impl.h
-- Up-to-date: /usr/local/include/grpcpp/grpcpp.h
-- Up-to-date: /usr/local/include/grpcpp/health_check_service_interface.h
-- Up-to-date: /usr/local/include/grpcpp/health_check_service_interface_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/call.h
-- Up-to-date: /usr/local/include/grpcpp/impl/channel_argument_option.h
-- Up-to-date: /usr/local/include/grpcpp/impl/client_unary_call.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/core_codegen.h
-- Up-to-date: /usr/local/include/grpcpp/impl/grpc_library.h
-- Up-to-date: /usr/local/include/grpcpp/impl/method_handler_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/rpc_method.h
-- Up-to-date: /usr/local/include/grpcpp/impl/rpc_service_method.h
-- Up-to-date: /usr/local/include/grpcpp/impl/serialization_traits.h
-- Up-to-date: /usr/local/include/grpcpp/impl/server_builder_option.h
-- Up-to-date: /usr/local/include/grpcpp/impl/server_builder_option_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/server_builder_plugin.h
-- Up-to-date: /usr/local/include/grpcpp/impl/server_initializer.h
-- Up-to-date: /usr/local/include/grpcpp/impl/server_initializer_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/service_type.h
-- Up-to-date: /usr/local/include/grpcpp/resource_quota.h
-- Up-to-date: /usr/local/include/grpcpp/resource_quota_impl.h
-- Up-to-date: /usr/local/include/grpcpp/security/auth_context.h
-- Up-to-date: /usr/local/include/grpcpp/security/auth_metadata_processor.h
-- Up-to-date: /usr/local/include/grpcpp/security/auth_metadata_processor_impl.h
-- Up-to-date: /usr/local/include/grpcpp/security/credentials.h
-- Up-to-date: /usr/local/include/grpcpp/security/credentials_impl.h
-- Up-to-date: /usr/local/include/grpcpp/security/server_credentials.h
-- Up-to-date: /usr/local/include/grpcpp/security/server_credentials_impl.h
-- Up-to-date: /usr/local/include/grpcpp/security/tls_credentials_options.h
-- Up-to-date: /usr/local/include/grpcpp/server.h
-- Up-to-date: /usr/local/include/grpcpp/server_builder.h
-- Up-to-date: /usr/local/include/grpcpp/server_builder_impl.h
-- Up-to-date: /usr/local/include/grpcpp/server_context.h
-- Up-to-date: /usr/local/include/grpcpp/server_impl.h
-- Up-to-date: /usr/local/include/grpcpp/server_posix.h
-- Up-to-date: /usr/local/include/grpcpp/server_posix_impl.h
-- Up-to-date: /usr/local/include/grpcpp/support/async_stream.h
-- Up-to-date: /usr/local/include/grpcpp/support/async_stream_impl.h
-- Up-to-date: /usr/local/include/grpcpp/support/async_unary_call.h
-- Up-to-date: /usr/local/include/grpcpp/support/async_unary_call_impl.h
-- Up-to-date: /usr/local/include/grpcpp/support/byte_buffer.h
-- Up-to-date: /usr/local/include/grpcpp/support/channel_arguments.h
-- Up-to-date: /usr/local/include/grpcpp/support/channel_arguments_impl.h
-- Up-to-date: /usr/local/include/grpcpp/support/client_callback.h
-- Up-to-date: /usr/local/include/grpcpp/support/client_callback_impl.h
-- Up-to-date: /usr/local/include/grpcpp/support/client_interceptor.h
-- Up-to-date: /usr/local/include/grpcpp/support/config.h
-- Up-to-date: /usr/local/include/grpcpp/support/interceptor.h
-- Up-to-date: /usr/local/include/grpcpp/support/message_allocator.h
-- Up-to-date: /usr/local/include/grpcpp/support/proto_buffer_reader.h
-- Up-to-date: /usr/local/include/grpcpp/support/proto_buffer_writer.h
-- Up-to-date: /usr/local/include/grpcpp/support/server_callback.h
-- Up-to-date: /usr/local/include/grpcpp/support/server_callback_impl.h
-- Up-to-date: /usr/local/include/grpcpp/support/server_interceptor.h
-- Up-to-date: /usr/local/include/grpcpp/support/slice.h
-- Up-to-date: /usr/local/include/grpcpp/support/status.h
-- Up-to-date: /usr/local/include/grpcpp/support/status_code_enum.h
-- Up-to-date: /usr/local/include/grpcpp/support/string_ref.h
-- Up-to-date: /usr/local/include/grpcpp/support/stub_options.h
-- Up-to-date: /usr/local/include/grpcpp/support/sync_stream.h
-- Up-to-date: /usr/local/include/grpcpp/support/sync_stream_impl.h
-- Up-to-date: /usr/local/include/grpcpp/support/time.h
-- Up-to-date: /usr/local/include/grpcpp/support/validate_service_config.h
-- Up-to-date: /usr/local/include/grpc/support/alloc.h
-- Up-to-date: /usr/local/include/grpc/support/atm.h
-- Up-to-date: /usr/local/include/grpc/support/atm_gcc_atomic.h
-- Up-to-date: /usr/local/include/grpc/support/atm_gcc_sync.h
-- Up-to-date: /usr/local/include/grpc/support/atm_windows.h
-- Up-to-date: /usr/local/include/grpc/support/cpu.h
-- Up-to-date: /usr/local/include/grpc/support/log.h
-- Up-to-date: /usr/local/include/grpc/support/log_windows.h
-- Up-to-date: /usr/local/include/grpc/support/port_platform.h
-- Up-to-date: /usr/local/include/grpc/support/string_util.h
-- Up-to-date: /usr/local/include/grpc/support/sync.h
-- Up-to-date: /usr/local/include/grpc/support/sync_custom.h
-- Up-to-date: /usr/local/include/grpc/support/sync_generic.h
-- Up-to-date: /usr/local/include/grpc/support/sync_posix.h
-- Up-to-date: /usr/local/include/grpc/support/sync_windows.h
-- Up-to-date: /usr/local/include/grpc/support/thd_id.h
-- Up-to-date: /usr/local/include/grpc/support/time.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_atomic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_gcc_sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/atm_windows.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/fork.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/gpr_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/log.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/port_platform.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_custom.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_generic.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_posix.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/sync_windows.h
-- Up-to-date: /usr/local/include/grpc/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc/byte_buffer_reader.h
-- Up-to-date: /usr/local/include/grpc/compression.h
-- Up-to-date: /usr/local/include/grpc/fork.h
-- Up-to-date: /usr/local/include/grpc/grpc.h
-- Up-to-date: /usr/local/include/grpc/grpc_posix.h
-- Up-to-date: /usr/local/include/grpc/grpc_security_constants.h
-- Up-to-date: /usr/local/include/grpc/load_reporting.h
-- Up-to-date: /usr/local/include/grpc/slice.h
-- Up-to-date: /usr/local/include/grpc/slice_buffer.h
-- Up-to-date: /usr/local/include/grpc/status.h
-- Up-to-date: /usr/local/include/grpc/support/workaround_list.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/byte_buffer_reader.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/compression_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/connectivity_state.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/grpc_types.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/propagation_bits.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/slice.h
-- Up-to-date: /usr/local/include/grpc/impl/codegen/status.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/async_stream.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/async_unary_call.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/byte_buffer.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/call.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/call_hook.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/channel_interface.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/client_context.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/client_unary_call.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/completion_queue.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/completion_queue_tag.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/config.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/core_codegen_interface.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/create_auth_context.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/grpc_library.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/metadata_map.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/method_handler_impl.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/rpc_method.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/rpc_service_method.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/security/auth_context.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/serialization_traits.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/server_context.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/server_interface.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/service_type.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/slice.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/status.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/status_code_enum.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/string_ref.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/stub_options.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/sync_stream.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/time.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/async_generic_service.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/async_stream.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/async_stream_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/async_unary_call.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/async_unary_call_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/byte_buffer.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/call.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/call_hook.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/call_op_set.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/call_op_set_interface.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/callback_common.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/channel_interface.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/client_callback.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/client_callback_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/client_context.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/client_context_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/client_interceptor.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/client_unary_call.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/completion_queue.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/completion_queue_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/completion_queue_tag.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/config.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/core_codegen_interface.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/create_auth_context.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/delegating_channel.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/grpc_library.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/intercepted_channel.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/interceptor.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/interceptor_common.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/message_allocator.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/metadata_map.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/method_handler.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/method_handler_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/rpc_method.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/rpc_service_method.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/security/auth_context.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/serialization_traits.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/server_callback.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/server_callback_handlers.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/server_callback_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/server_context.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/server_context_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/server_interceptor.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/server_interface.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/service_type.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/slice.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/status.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/status_code_enum.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/string_ref.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/stub_options.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/sync_stream.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/sync_stream_impl.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/time.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/sync.h
-- Up-to-date: /usr/local/include/grpc++/impl/codegen/config_protobuf.h
-- Up-to-date: /usr/local/include/grpcpp/impl/codegen/config_protobuf.h
-- Installing: /usr/local/include/grpcpp/ext/channelz_service_plugin.h
-- Installing: /usr/local/include/grpcpp/ext/channelz_service_plugin_impl.h
-- Installing: /usr/local/lib/cmake/grpc/gRPCConfig.cmake
-- Installing: /usr/local/lib/cmake/grpc/gRPCConfigVersion.cmake
-- Installing: /usr/local/lib/cmake/grpc/modules/Findc-ares.cmake
-- Up-to-date: /usr/local/share/grpc/roots.pem
-- Installing: /usr/local/lib/pkgconfig/gpr.pc
-- Installing: /usr/local/lib/pkgconfig/grpc.pc
-- Installing: /usr/local/lib/pkgconfig/grpc_unsecure.pc
-- Installing: /usr/local/lib/pkgconfig/grpc++.pc
-- Installing: /usr/local/lib/pkgconfig/grpc++_unsecure.pc
-- Installing: /usr/local/lib/libcares.a
-- Installing: /usr/local/include/ares.h
-- Installing: /usr/local/include/ares_version.h
-- Installing: /usr/local/include/ares_dns.h
-- Installing: /usr/local/include/ares_build.h
-- Installing: /usr/local/include/ares_rules.h
-- Installing: /usr/local/lib/cmake/c-ares/c-ares-targets.cmake
-- Installing: /usr/local/lib/cmake/c-ares/c-ares-targets-noconfig.cmake
-- Installing: /usr/local/lib/cmake/c-ares/c-ares-config.cmake
-- Installing: /usr/local/lib/pkgconfig/libcares.pc
-- Installing: /usr/local/bin/ahost
-- Installing: /usr/local/bin/adig
-- Installing: /usr/local/bin/acountry
-- Installing: /usr/local/lib/libprotobuf-lite.so.3.8.0.0
-- Installing: /usr/local/lib/libprotobuf-lite.so
-- Set runtime path of "/usr/local/lib/libprotobuf-lite.so.3.8.0.0" to "$ORIGIN"
-- Installing: /usr/local/lib/libprotobuf.so.3.8.0.0
-- Installing: /usr/local/lib/libprotobuf.so
-- Set runtime path of "/usr/local/lib/libprotobuf.so.3.8.0.0" to "$ORIGIN"
-- Installing: /usr/local/lib/libprotoc.so.3.8.0.0
-- Installing: /usr/local/lib/libprotoc.so
-- Set runtime path of "/usr/local/lib/libprotoc.so.3.8.0.0" to "$ORIGIN"
-- Installing: /usr/local/bin/protoc-3.8.0.0
-- Installing: /usr/local/bin/protoc
-- Set runtime path of "/usr/local/bin/protoc-3.8.0.0" to "$ORIGIN/../lib"
-- Installing: /usr/local/lib/pkgconfig/protobuf.pc
-- Installing: /usr/local/lib/pkgconfig/protobuf-lite.pc
-- Installing: /usr/local/include/google/protobuf/any.h
-- Installing: /usr/local/include/google/protobuf/any.pb.h
-- Installing: /usr/local/include/google/protobuf/api.pb.h
-- Installing: /usr/local/include/google/protobuf/arena.h
-- Installing: /usr/local/include/google/protobuf/arena_impl.h
-- Installing: /usr/local/include/google/protobuf/arenastring.h
-- Installing: /usr/local/include/google/protobuf/compiler/code_generator.h
-- Installing: /usr/local/include/google/protobuf/compiler/command_line_interface.h
-- Installing: /usr/local/include/google/protobuf/compiler/cpp/cpp_generator.h
-- Installing: /usr/local/include/google/protobuf/compiler/csharp/csharp_generator.h
-- Installing: /usr/local/include/google/protobuf/compiler/csharp/csharp_names.h
-- Installing: /usr/local/include/google/protobuf/compiler/importer.h
-- Installing: /usr/local/include/google/protobuf/compiler/java/java_generator.h
-- Installing: /usr/local/include/google/protobuf/compiler/java/java_names.h
-- Installing: /usr/local/include/google/protobuf/compiler/js/js_generator.h
-- Installing: /usr/local/include/google/protobuf/compiler/js/well_known_types_embed.h
-- Installing: /usr/local/include/google/protobuf/compiler/objectivec/objectivec_generator.h
-- Installing: /usr/local/include/google/protobuf/compiler/objectivec/objectivec_helpers.h
-- Installing: /usr/local/include/google/protobuf/compiler/parser.h
-- Installing: /usr/local/include/google/protobuf/compiler/php/php_generator.h
-- Installing: /usr/local/include/google/protobuf/compiler/plugin.h
-- Installing: /usr/local/include/google/protobuf/compiler/plugin.pb.h
-- Installing: /usr/local/include/google/protobuf/compiler/python/python_generator.h
-- Installing: /usr/local/include/google/protobuf/compiler/ruby/ruby_generator.h
-- Installing: /usr/local/include/google/protobuf/descriptor.h
-- Installing: /usr/local/include/google/protobuf/descriptor.pb.h
-- Installing: /usr/local/include/google/protobuf/descriptor_database.h
-- Installing: /usr/local/include/google/protobuf/duration.pb.h
-- Installing: /usr/local/include/google/protobuf/dynamic_message.h
-- Installing: /usr/local/include/google/protobuf/empty.pb.h
-- Installing: /usr/local/include/google/protobuf/extension_set.h
-- Installing: /usr/local/include/google/protobuf/extension_set_inl.h
-- Installing: /usr/local/include/google/protobuf/field_mask.pb.h
-- Installing: /usr/local/include/google/protobuf/generated_enum_reflection.h
-- Installing: /usr/local/include/google/protobuf/generated_enum_util.h
-- Installing: /usr/local/include/google/protobuf/generated_message_reflection.h
-- Installing: /usr/local/include/google/protobuf/generated_message_table_driven.h
-- Installing: /usr/local/include/google/protobuf/generated_message_util.h
-- Installing: /usr/local/include/google/protobuf/has_bits.h
-- Installing: /usr/local/include/google/protobuf/implicit_weak_message.h
-- Installing: /usr/local/include/google/protobuf/inlined_string_field.h
-- Installing: /usr/local/include/google/protobuf/io/coded_stream.h
-- Installing: /usr/local/include/google/protobuf/io/gzip_stream.h
-- Installing: /usr/local/include/google/protobuf/io/printer.h
-- Installing: /usr/local/include/google/protobuf/io/strtod.h
-- Installing: /usr/local/include/google/protobuf/io/tokenizer.h
-- Installing: /usr/local/include/google/protobuf/io/zero_copy_stream.h
-- Installing: /usr/local/include/google/protobuf/io/zero_copy_stream_impl.h
-- Installing: /usr/local/include/google/protobuf/io/zero_copy_stream_impl_lite.h
-- Installing: /usr/local/include/google/protobuf/map.h
-- Installing: /usr/local/include/google/protobuf/map_entry.h
-- Installing: /usr/local/include/google/protobuf/map_entry_lite.h
-- Installing: /usr/local/include/google/protobuf/map_field.h
-- Installing: /usr/local/include/google/protobuf/map_field_inl.h
-- Installing: /usr/local/include/google/protobuf/map_field_lite.h
-- Installing: /usr/local/include/google/protobuf/map_type_handler.h
-- Installing: /usr/local/include/google/protobuf/message.h
-- Installing: /usr/local/include/google/protobuf/message_lite.h
-- Installing: /usr/local/include/google/protobuf/metadata.h
-- Installing: /usr/local/include/google/protobuf/metadata_lite.h
-- Installing: /usr/local/include/google/protobuf/parse_context.h
-- Installing: /usr/local/include/google/protobuf/port.h
-- Installing: /usr/local/include/google/protobuf/port_def.inc
-- Installing: /usr/local/include/google/protobuf/port_undef.inc
-- Installing: /usr/local/include/google/protobuf/reflection.h
-- Installing: /usr/local/include/google/protobuf/reflection_ops.h
-- Installing: /usr/local/include/google/protobuf/repeated_field.h
-- Installing: /usr/local/include/google/protobuf/service.h
-- Installing: /usr/local/include/google/protobuf/source_context.pb.h
-- Installing: /usr/local/include/google/protobuf/struct.pb.h
-- Installing: /usr/local/include/google/protobuf/stubs/bytestream.h
-- Installing: /usr/local/include/google/protobuf/stubs/callback.h
-- Installing: /usr/local/include/google/protobuf/stubs/casts.h
-- Installing: /usr/local/include/google/protobuf/stubs/common.h
-- Installing: /usr/local/include/google/protobuf/stubs/fastmem.h
-- Installing: /usr/local/include/google/protobuf/stubs/hash.h
-- Installing: /usr/local/include/google/protobuf/stubs/logging.h
-- Installing: /usr/local/include/google/protobuf/stubs/macros.h
-- Installing: /usr/local/include/google/protobuf/stubs/mutex.h
-- Installing: /usr/local/include/google/protobuf/stubs/once.h
-- Installing: /usr/local/include/google/protobuf/stubs/platform_macros.h
-- Installing: /usr/local/include/google/protobuf/stubs/port.h
-- Installing: /usr/local/include/google/protobuf/stubs/status.h
-- Installing: /usr/local/include/google/protobuf/stubs/stl_util.h
-- Installing: /usr/local/include/google/protobuf/stubs/stringpiece.h
-- Installing: /usr/local/include/google/protobuf/stubs/strutil.h
-- Installing: /usr/local/include/google/protobuf/stubs/template_util.h
-- Installing: /usr/local/include/google/protobuf/text_format.h
-- Installing: /usr/local/include/google/protobuf/timestamp.pb.h
-- Installing: /usr/local/include/google/protobuf/type.pb.h
-- Installing: /usr/local/include/google/protobuf/unknown_field_set.h
-- Installing: /usr/local/include/google/protobuf/util/delimited_message_util.h
-- Installing: /usr/local/include/google/protobuf/util/field_comparator.h
-- Installing: /usr/local/include/google/protobuf/util/field_mask_util.h
-- Installing: /usr/local/include/google/protobuf/util/json_util.h
-- Installing: /usr/local/include/google/protobuf/util/message_differencer.h
-- Installing: /usr/local/include/google/protobuf/util/time_util.h
-- Installing: /usr/local/include/google/protobuf/util/type_resolver.h
-- Installing: /usr/local/include/google/protobuf/util/type_resolver_util.h
-- Installing: /usr/local/include/google/protobuf/wire_format.h
-- Installing: /usr/local/include/google/protobuf/wire_format_lite.h
-- Installing: /usr/local/include/google/protobuf/wrappers.pb.h
-- Installing: /usr/local/include/google/protobuf/any.proto
-- Installing: /usr/local/include/google/protobuf/api.proto
-- Installing: /usr/local/include/google/protobuf/compiler/plugin.proto
-- Installing: /usr/local/include/google/protobuf/descriptor.proto
-- Installing: /usr/local/include/google/protobuf/duration.proto
-- Installing: /usr/local/include/google/protobuf/empty.proto
-- Installing: /usr/local/include/google/protobuf/field_mask.proto
-- Installing: /usr/local/include/google/protobuf/source_context.proto
-- Installing: /usr/local/include/google/protobuf/struct.proto
-- Installing: /usr/local/include/google/protobuf/timestamp.proto
-- Installing: /usr/local/include/google/protobuf/type.proto
-- Installing: /usr/local/include/google/protobuf/wrappers.proto
-- Up-to-date: /usr/local/include/google/protobuf/descriptor.proto
-- Up-to-date: /usr/local/include/google/protobuf/any.proto
-- Up-to-date: /usr/local/include/google/protobuf/api.proto
-- Up-to-date: /usr/local/include/google/protobuf/duration.proto
-- Up-to-date: /usr/local/include/google/protobuf/empty.proto
-- Up-to-date: /usr/local/include/google/protobuf/field_mask.proto
-- Up-to-date: /usr/local/include/google/protobuf/source_context.proto
-- Up-to-date: /usr/local/include/google/protobuf/struct.proto
-- Up-to-date: /usr/local/include/google/protobuf/timestamp.proto
-- Up-to-date: /usr/local/include/google/protobuf/type.proto
-- Up-to-date: /usr/local/include/google/protobuf/wrappers.proto
-- Up-to-date: /usr/local/include/google/protobuf/compiler/plugin.proto
-- Installing: /usr/local/lib/cmake/protobuf/protobuf-targets.cmake
-- Installing: /usr/local/lib/cmake/protobuf/protobuf-targets-noconfig.cmake
-- Up-to-date: /usr/local/lib/cmake/protobuf
-- Installing: /usr/local/lib/cmake/protobuf/protobuf-module.cmake
-- Installing: /usr/local/lib/cmake/protobuf/protobuf-config.cmake
-- Installing: /usr/local/lib/cmake/protobuf/protobuf-config-version.cmake
-- Installing: /usr/local/lib/cmake/protobuf/protobuf-options.cmake
-- Installing: /usr/local/lib/libz.so.1.2.11
-- Installing: /usr/local/lib/libz.so.1
-- Installing: /usr/local/lib/libz.so
-- Installing: /usr/local/lib/libz.a
-- Installing: /usr/local/include/zconf.h
-- Installing: /usr/local/include/zlib.h
-- Up-to-date: /usr/local/share/man/man3/zlib.3
-- Installing: /usr/local/share/pkgconfig/zlib.pc
leo@ubuntu:~/git/grpc/cmake/build$ 

```