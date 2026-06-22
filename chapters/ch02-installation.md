# Chapter 2: Installation Guide

## Core Idea
GROMACS is built from source using CMake with extensive options for compilers, parallelization (MPI/OpenMP), GPU acceleration (CUDA/OpenCL/SYCL/HIP), and optional libraries. The standard build is: `cmake .. -DGMX_GPU=CUDA && make -j && make install`.

## Frameworks Introduced
- **Quick installation**: `tar xfz gromacs-2026.2.tar.gz && cd gromacs-2026.2 && mkdir build && cd build && cmake .. -DGMX_GPU=CUDA -DGMX_FFT_LIBRARY=fftw3 && make -j && make install` — for single workstation
- **Cluster installation**: Same but with `-DGMX_MPI=ON` for multi-node; source `GMXRC` after install

## Key Concepts

### Prerequisites
- **Platform**: Linux, macOS, Windows (via WSL/MSYS2), Cray
- **Compiler**: GCC recommended (newest version). Known issues with GCC 12-14 on POWER9. Clang works but check OpenMP support
- **CMake**: Version 3.18 or newer required
- **FFT library**: FFTW3 recommended. MKL or vkFFT (for GPU) alternatives. Compile FFTW yourself with correct x86 flags
- **MPI**: OpenMPI recommended for multi-node. Thread-MPI included for single-node
- **OpenMP**: Required for threading. Check compiler support

### GPU Backends
| Backend | CMake Flag | Platform |
|---|---|---|
| CUDA | `-DGMX_GPU=CUDA` | NVIDIA GPUs (recommended) |
| OpenCL | `-DGMX_GPU=OpenCL` | Cross-vendor (legacy) |
| SYCL/DPC++ | `-DGMX_GPU=SYCL -DGMX_SYCL=DPCPP` | Intel oneAPI for AMD/NVIDIA |
| SYCL/ACPP | `-DGMX_GPU=SYCL -DGMX_SYCL=ACPP` | AdaptiveCpp (AMD recommended) |
| HIP | `-DGMX_GPU=HIP` | AMD GPUs |

### Key CMake Options
| Option | Purpose | Default |
|---|---|---|
| `-DGMX_GPU` | GPU backend | `OFF` |
| `-DGMX_MPI` | MPI parallelization | `OFF` |
| `-DGMX_OPENMP` | Threading | `ON` |
| `-DGMX_FFT_LIBRARY` | FFT library | `fftw3` |
| `-DGMX_SIMD` | CPU SIMD level | Auto-detected |
| `-DGMX_DOUBLE` | Double precision | `OFF` |
| `-DGMX_BUILD_OWN_FFTW` | Build bundled FFTW | `OFF` |
| `-DGMX_USE_RDTSCP` | Low-level timing | `ON` (x86) |
| `-DBUILD_SHARED_LIBS` | Shared libraries | `ON` |
| `-DGMX_PREFER_STATIC_LIBS` | Static linking | `OFF` |
| `-DREGRESSIONTEST_PATH` | Test path | None |
| `-DGMX_GPU_NB_CLUSTER_SIZE` | GPU cluster size | 4 (CUDA) / 8 (SYCL) |
| `-DGMX_GPU_FFT_LIBRARY` | GPU FFT library | `vkfft` |

### Build Types
- **Release** (default): Optimized, no debug symbols
- **Debug**: For development, very slow
- **RelWithDebInfo**: Optimized with debug symbols — useful for profiling
- **MinSizeRel**: Size-optimized

## Code Examples
```bash
# Typical workstation build (NVIDIA GPU)
cmake .. -DGMX_GPU=CUDA -DGMX_FFT_LIBRARY=fftw3 \
         -DCMAKE_INSTALL_PREFIX=/opt/gromacs
make -j$(nproc)
make install
source /opt/gromacs/bin/GMXRC

# Cluster build (CPU-only with MPI)
cmake .. -DGMX_MPI=ON -DGMX_OPENMP=ON \
         -DGMX_FFT_LIBRARY=fftw3
make -j

# AMD GPU build
cmake .. -DGMX_GPU=HIP -DGMX_FFT_LIBRARY=fftw3

# Test the build
make check
```

## Worked Example
Building GROMACS 2026.2 on a Linux workstation with NVIDIA GPU:
1. Download: `wget https://ftp.gromacs.org/gromacs/gromacs-2026.2.tar.gz`
2. Extract: `tar xfz gromacs-2026.2.tar.gz && cd gromacs-2026.2`
3. Configure: `mkdir build && cd build && cmake .. -DGMX_GPU=CUDA`
4. Build: `make -j16` (adjust core count)
5. Test: `make check` (requires regression tests downloaded and REGRESSIONTEST_PATH set)
6. Install: `make install`
7. Setup: `source /usr/local/gromacs/bin/GMXRC`

## Key Takeaways
1. Always use single precision unless you specifically need double (normal mode analysis)
2. Compile FFTW yourself with `--enable-sse2` on x86 for best performance
3. Use newest compiler and GPU driver versions
4. GCC gives best performance on x86 and POWER
5. AVX2 preferred over AVX512 in GPU or highly parallel MPI runs
6. Check `GMX_SIMD` is correct for your CPU architecture — log file warns if suboptimal

## Connects To
- **Ch 11**: Performance tuning after installation
- **Ch 3**: Known issues with specific compiler/platform combinations
- **Ch 12**: Common build errors
