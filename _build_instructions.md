# GSL Build Instructions

This document provides step-by-step instructions for building the GNU Scientific Library (GSL) on Windows using CMake and Visual Studio.

## Prerequisites

- **CMake** (version 3.4 or later) - Available at `c:\dev\cmake` or install from <https://cmake.org/>
- **Visual Studio 2022** with C++ development tools
- **Git** (optional, for cloning the repository)

## Build Steps

### 1. Setup Environment

First, ensure CMake is in your system PATH:

```powershell
# Add CMake to current session PATH
$env:PATH = "c:\dev\cmake\bin;" + $env:PATH

# Or add permanently to user environment
[Environment]::SetEnvironmentVariable("Path", "c:\dev\cmake\bin;" + [Environment]::GetEnvironmentVariable("Path", "User"), "User")
```

### 2. Configure the Build

Navigate to the GSL source directory and create a build directory:

```powershell
cd c:\DataAnnotations\Other\c\gsl
mkdir build
cd build
```

Configure the project with CMake:

```powershell
cmake .. -G"Visual Studio 17 2022" -DCMAKE_INSTALL_PREFIX="C:\dev\gsl" -DGSL_INSTALL_MULTI_CONFIG=ON -DGSL_DISABLE_TESTS=1 -DNO_AMPL_BINDINGS=1
```

**Configuration Options:**

- `-G"Visual Studio 17 2022"` - Use Visual Studio 2022 generator
- `-DCMAKE_INSTALL_PREFIX="C:\dev\gsl"` - Install location (no admin rights required)
- `-DGSL_INSTALL_MULTI_CONFIG=ON` - Support multiple configurations (Debug/Release)
- `-DGSL_DISABLE_TESTS=1` - Skip building tests (faster build)
- `-DNO_AMPL_BINDINGS=1` - Disable AMPL bindings (not needed for most users)

### 3. Build the Library

Build the Release configuration:

```powershell
cmake --build . --config Release -j 8
```

The `-j 8` flag enables parallel compilation using 8 CPU cores (adjust as needed).

### 4. Install the Library

Install GSL to the specified directory:

```powershell
cmake --build . --config Release --target install
```

This will install:

- **Libraries**: `C:\dev\gsl\lib\Release\gsl.lib` and `C:\dev\gsl\lib\Release\gslcblas.lib`
- **Headers**: `C:\dev\gsl\include\gsl\*.h`
- **CMake config**: `C:\dev\gsl\lib\cmake\gsl-2.7\`
- **pkg-config**: `C:\dev\gsl\lib\pkgconfig\gsl.pc`

## Running Tests (Optional)

GSL includes a comprehensive test suite to verify the library functions correctly. If you want to run tests, you need to build with tests enabled.

### Building with Tests

To enable tests, configure without the `-DGSL_DISABLE_TESTS=1` flag:

```powershell
# Clean build directory
cd c:\DataAnnotations\Other\c\gsl
Remove-Item -Recurse -Force build -ErrorAction SilentlyContinue
mkdir build
cd build

# Configure with tests enabled
cmake .. -G"Visual Studio 17 2022" -DCMAKE_INSTALL_PREFIX="C:\dev\gsl" -DGSL_INSTALL_MULTI_CONFIG=ON -DNO_AMPL_BINDINGS=1

# Build the library and tests
cmake --build . --config Release -j 8
```

### Running All Tests

Once built with tests enabled, run the complete test suite:

```powershell
# Run all tests in parallel (recommended)
ctest -C Release -j 8

# Run tests with verbose output (for debugging)
ctest -C Release -j 8 -V

# Run tests with very verbose output (shows all test output)
ctest -C Release -j 8 -VV
```

### Running Specific Tests

You can run individual test modules:

```powershell
# Run only linear algebra tests
ctest -C Release -R "linalg_test"

# Run only special function tests
ctest -C Release -R "specfunc_test"

# Run only vector tests
ctest -C Release -R "vector_test"

# List all available tests
ctest -C Release -N
```

### Understanding Test Results

A typical test run will show results like:

```text
Test project C:/DataAnnotations/Other/c/gsl/build
      Start  1: sys_test
      Start  2: err_test
      Start  3: bst_test
 1/57 Test  #1: sys_test .........................   Passed    0.39 sec
 2/57 Test  #2: err_test .........................   Passed    0.34 sec
 3/57 Test  #3: bst_test .........................   Passed    0.29 sec
...
95% tests passed, 3 tests failed out of 57
Total Test time (real) =   8.44 sec
```

**Test Status Meanings:**

- **Passed**: Test completed successfully
- **Failed**: Test found errors (may indicate build issues)
- **Timeout**: Test took too long to complete
- **Exit code**: Non-zero exit codes indicate errors

### Re-running Failed Tests

If some tests fail, you can re-run only the failed tests:

```powershell
# Re-run failed tests with detailed output
ctest -C Release --rerun-failed --output-on-failure
```

### Test Performance Notes

- **Test time**: Complete test suite takes 5-15 minutes depending on your system
- **Parallel execution**: Use `-j 8` to run tests in parallel (adjust number based on CPU cores)
- **Memory usage**: Some tests may require significant memory for large matrix operations
- **Expected failures**: A few tests may fail on Windows due to floating-point precision differences - this is usually not a concern for typical usage

### Common Test Issues

1. **Vector/Matrix test failures**: Often related to compiler optimizations or floating-point precision
2. **Timeout failures**: Increase timeout with `--timeout 300` (5 minutes)
3. **Memory errors**: Ensure sufficient RAM available during testing
4. **Build configuration**: Some tests may behave differently between Debug and Release builds

## Using GSL in Your Projects

### Method 1: Direct Compilation with MSVC

```powershell
# Set up Visual Studio environment
& "C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvars64.bat"

# Compile your program
cl.exe /I"C:\dev\gsl\include" your_program.c /link "C:\dev\gsl\lib\Release\gsl.lib" "C:\dev\gsl\lib\Release\gslcblas.lib"
```

### Method 2: Using CMake

Create a `CMakeLists.txt` file:

```cmake
cmake_minimum_required(VERSION 3.4)
project(MyGSLProject)

# Find GSL
find_package(GSL REQUIRED PATHS "C:/dev/gsl/lib/cmake/gsl-2.7")

# Create executable
add_executable(my_program your_program.c)

# Link GSL
target_link_libraries(my_program GSL::gsl GSL::gslcblas)
```

### Method 3: Using pkg-config

If you have pkg-config installed:

```bash
export PKG_CONFIG_PATH="C:/dev/gsl/lib/pkgconfig:$PKG_CONFIG_PATH"
gcc `pkg-config --cflags gsl` your_program.c `pkg-config --libs gsl`
```

## Example Program

Here's a simple test program (`test_gsl.c`):

```c
#include <stdio.h>
#include <gsl/gsl_sf_bessel.h>

int main(void)
{
    double x = 15.0;
    double y = gsl_sf_bessel_J0(x);
    printf("J0(%g) = %.18e\n", x, y);
    return 0;
}
```

Expected output:

```text
J0(15) = -7.023729441241979e-02
```

## Troubleshooting

### Common Issues

1. **CMake not found**: Ensure CMake is in your PATH or use the full path to cmake.exe
2. **Visual Studio not found**: Install Visual Studio 2022 with C++ development tools
3. **Permission denied during install**: Use a custom install prefix that doesn't require admin rights
4. **AMPL binding errors**: Use `-DNO_AMPL_BINDINGS=1` to disable AMPL support

### Build Options

- **Enable tests**: Remove `-DGSL_DISABLE_TESTS=1` (increases build time)
- **Debug build**: Use `--config Debug` instead of `--config Release`
- **Shared libraries**: Add `-DBUILD_SHARED_LIBS=ON` to build DLLs instead of static libraries
- **Dynamic runtime**: Add `-DMSVC_RUNTIME_DYNAMIC=ON` for dynamic MSVC runtime linking

### Selective Building

To build only specific GSL modules (when not using AMPL bindings):

```powershell
cmake .. -DBUILDLIBS=ode-initval2,linalg -DNO_AMPL_BINDINGS=1 -DGSL_DISABLE_TESTS=1
```

## Library Contents

GSL provides comprehensive numerical routines including:

- **Linear Algebra**: Matrices, vectors, BLAS operations
- **Special Functions**: Bessel functions, gamma functions, etc.
- **Random Numbers**: Multiple RNG algorithms and distributions
- **Numerical Integration**: Adaptive quadrature, Monte Carlo
- **Optimization**: Multi-dimensional minimization
- **Differential Equations**: ODE solvers
- **Statistics**: Basic and advanced statistical functions
- **Signal Processing**: FFT, filtering, wavelets
- **And much more...**

## Version Information

- **GSL Version**: 2.7
- **CMake Version**: 3.30.1 (minimum 3.4 required)
- **Visual Studio**: 2022 Community (MSVC 19.44)
- **Target Platform**: Windows x64

For more information, visit the [GSL official website](http://www.gnu.org/software/gsl/).
