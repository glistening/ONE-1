# Backend API

Backend API is defined by One Runtime. It is about actual computation of operations and memory management for operands. In order to allow different kinds of computation units or libraries, Backend API is exposed to support user defined operation kernels and memory manager. It contains several C++ interface classes which are **subject to change**.

## How backends are loaded

When a backend ID is given to a session, the compiler module tries to load `libbackend_{BACKEND_ID}.so`. If it is successful, the runtime looks up for C API functions in it, and make use of those.

## C and C++ API

### C API

We have 2 C API functions which are used as the entrypoint and the exitpoint. Here are the definitions of those.

```c
onert::backend::Backend *onert_backend_create();
void onert_backend_destroy(onert::backend::Backend *backend);
```

What they do is creating a C++ object and destroying it, respectively. These two functions are the only ones that are dynamically resolved at runtime.

### C++ API

> **NOTE** C++ API is subject to change so it may change in every release

C API above is just an entrypoint and it delegates core stuff to C++ API.

Here are major classes are described below. One must implement these classes(and some more classes) to create a backend.

- `Backend` : Responsible to create a backend context which is a set of backend components
- `IConfig` : Configurations and miscellaneous stuff
- `ITensorRegistry` : A set of tensor(`ITensor`) objects that are used by the current backend
- `ITensorBuilder` : Make tensor object and register it to `ITensorRegistry` and static tensors
- `IKernelGenerator` : Generates operation kernels

Please refer to each class document for details. You may refer to [Bundle Backends](#bundle-backends) for actual implementation samples.

## Provided Backend Implementations

We provide some backends along with the runtime. There is the special backend `builtin` which is part of runtime core, and some bundle backends which are baseline backends and samples of backend implementation.

## `builtin` Backend

`builtin` is a special backend that is always loaded(statically linked, part of runtime core). It is implemented just like other backends, but there are some things that it does exclusively.

- Has kernels for If, While and Permute operations (Kernels from other backends are never be used)
- The runtime core directly creates `builtin`'s tensor objects to accept user-given input and output buffers
- The runtime core gives the executor context to `builtin` backend which allows control flow ops can change execution flow properly

## Bundle Backends

Without actual implmentation of backends, we cannot run any models. So we provide 3 bundle backends which support dozens of operations.

### cpu

This backend is written in C++ and all the computation is done with CPU only.

### acl_neon

`acl_neon` is a backend that is an adaptation layer of [ARM ComputeLibrary](https://github.com/ARM-software/ComputeLibrary) NE(NEON) part. So it basically only uses CPU too, but worksonly on ARM.

### acl_cl

`acl_cl` is a backend that is an adaptation layer of [ARM ComputeLibrary](https://github.com/ARM-software/ComputeLibrary) CL(OpenCL) part. OpenCL support(`libOpenCL.so`) is also necessary in the running environment to be able to use this backend. Also, it works only on ARM.
