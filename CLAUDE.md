# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

This project uses CMake as the build system. The Makefile has been deprecated (see Makefile:6-9 for the deprecation message).

**Basic CPU-only build:**
```bash
cmake -B build
cmake --build build --config Release
```

**Debug build:**
```bash
cmake -B build -DCMAKE_BUILD_TYPE=Debug
cmake --build build
```

**Build with parallel compilation:**
```bash
cmake --build build --config Release -j $(nproc)
```

**Build with specific backends:**
- CUDA: `cmake -B build -DGGML_CUDA=ON`
- Metal (macOS default): `cmake -B build -DGGML_METAL=ON`
- OpenCL: `cmake -B build -DGGML_OPENCL=ON`
- Vulkan: `cmake -B build -DGGML_VULKAN=ON`
- SYCL: `cmake -B build -DGGML_SYCL=ON`
- HIP (AMD): `cmake -B build -DGGML_HIP=ON`

## Testing

**Run all tests:**
```bash
cd build && ctest
```

**Run specific test:**
```bash
./build/bin/test-backend-ops
./build/bin/test-sampling
```

**Run local CI (comprehensive testing):**
```bash
mkdir tmp
bash ./ci/run.sh ./tmp/results ./tmp/mnt
```

With specific backends:
- CUDA: `GG_BUILD_CUDA=1 bash ./ci/run.sh ./tmp/results ./tmp/mnt`
- SYCL: `source /opt/intel/oneapi/setvars.sh && GG_BUILD_SYCL=1 bash ./ci/run.sh ./tmp/results ./tmp/mnt`

## Architecture Overview

**Core Components:**
- `ggml/`: Tensor library for efficient machine learning (core computation engine)
- `src/`: llama.cpp library implementation (higher-level LLM interface)
- `common/`: Shared utilities and helper functions
- `tests/`: Extensive test suite covering various components
- `tools/`: Command-line applications built on top of the library
- `examples/`: Sample code demonstrating library usage

**Key directories:**
- `ggml/src/ggml-cuda/`: CUDA backend implementation
- `ggml/src/ggml-metal/`: Metal backend for Apple Silicon
- `ggml/src/ggml-cpu/`: CPU backend with SIMD optimizations
- `src/`: Contains model loading, KV cache, inference logic

**Backend Architecture:**
The codebase supports multiple acceleration backends that are prioritized at runtime:
1. Metal (Apple Silicon)
2. CUDA (NVIDIA GPUs) 
3. Other GPU backends (Vulkan, OpenCL, etc.)
4. CPU with SIMD optimizations

**Key Design Patterns:**
- Tensors store data in row-major order (dimension 0 = columns, 1 = rows, 2 = matrices)
- Matrix multiplication: `C = ggml_mul_mat(ctx, A, B)` means $C^T = A B^T \Leftrightarrow C = B A^T$
- Snake case naming convention for functions, variables, and types
- Enum values are UPPER_CASE and prefixed with enum name

## Development Guidelines

**Code Style (from CONTRIBUTING.md):**
- Use 4 spaces for indentation
- Avoid modern STL constructs, prefer basic loops
- Vertical alignment for readability
- C++ filenames: lowercase with dashes
- Python filenames: lowercase with underscores

**Testing Requirements:**
- Run `llama-perplexity` and `llama-bench` to verify performance
- Test changes on various backends when modifying ggml operators
- Add tests for new ggml operators in `test-backend-ops`

**Tools Directory:**
- `tools/main/`: Main CLI tool (`llama-cli`)
- `tools/server/`: HTTP server with OpenAI-compatible API
- `tools/quantize/`: Model quantization utilities
- `tools/llama-bench/`: Performance benchmarking
- `tools/perplexity/`: Model quality evaluation