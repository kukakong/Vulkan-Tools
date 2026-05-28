# Vulkan-Tools Code Wiki

> **Branch**: `main` | **Version**: 1.4.352 | **License**: Apache 2.0

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Repository Structure](#repository-structure)
3. [Build System](#build-system)
4. [Module: cube (VkCube Demo)](#module-cube-vkcube-demo)
5. [Module: vulkaninfo](#module-vulkaninfo)
6. [Module: icd (Mock ICD)](#module-icd-mock-icd)
7. [Module: scripts (Code Generation)](#module-scripts-code-generation)
8. [Module: tests](#module-tests)
9. [Dependencies](#dependencies)
10. [Cross-Compilation Guide](#cross-compilation-guide)
11. [Key Data Structures](#key-data-structures)
12. [WSI Platform Support](#wsi-platform-support)
13. [Code Generation Pipeline](#code-generation-pipeline)

---

## Project Overview

Vulkan-Tools is the official Khronos Group repository providing essential Vulkan tools and utilities that assist developers in verifying their applications' correct use of the Vulkan API. The project is primarily developed by LunarG, Inc. with contributions from Valve Corporation and oversight by Khronos.

### Core Components

| Component | Description | Language |
|-----------|-------------|----------|
| **vkcube** | Vulkan cube demo application (C API) | C99 |
| **vkcubepp** | Vulkan cube demo application (C++ API) | C++17 |
| **vulkaninfo** | GPU/device capability reporting tool | C++17 |
| **VkICD_mock_icd** | Mock Installable Client Driver for testing | C++17 |

---

## Repository Structure

```
Vulkan-Tools/
├── CMakeLists.txt          # Top-level CMake configuration (v1.4.352)
├── BUILD.md                # Build instructions
├── README.md               # Project overview
├── BUILD.gn                # GN build configuration (Chromium)
├── cube/                   # VkCube demo application
│   ├── CMakeLists.txt      # Cube build configuration
│   ├── cube.c              # C version of the demo
│   ├── cube.cpp            # C++ version of the demo
│   ├── cube_functions.h    # Vulkan function pointer declarations
│   ├── cube.vert           # Vertex shader (GLSL)
│   ├── cube.frag           # Fragment shader (GLSL)
│   ├── cube.vert.inc       # Pre-compiled vertex shader (SPIR-V)
│   ├── cube.frag.inc       # Pre-compiled fragment shader (SPIR-V)
│   ├── linmath.h           # Linear math utilities (matrix/vector)
│   ├── gettime.h           # Time measurement utilities
│   ├── object_type_string_helper.h  # VkObjectType string helpers
│   ├── android/            # Android-specific build files
│   │   ├── CMakeLists.txt  # Android CMake configuration
│   │   ├── AndroidManifest.xml
│   │   ├── android_util.cpp
│   │   └── android_util.h
│   ├── macOS/              # macOS app bundle configurations
│   │   ├── cube/           # C version macOS app
│   │   └── cubepp/         # C++ version macOS app
│   ├── xcb_loader.h        # XCB WSI dynamic loader
│   ├── xlib_loader.h       # Xlib WSI dynamic loader
│   ├── wayland_loader.h    # Wayland WSI dynamic loader
│   └── fuchsia/            # Fuchsia platform support
├── vulkaninfo/             # Vulkan info tool
│   ├── CMakeLists.txt      # Vulkaninfo build configuration
│   ├── vulkaninfo.cpp      # Main source file
│   ├── vulkaninfo.h        # Header with data structures
│   ├── vulkaninfo_functions.h  # Function implementations
│   ├── outputprinter.h     # Multi-format output (text/html/json)
│   ├── generated/          # Generated header (vulkaninfo.hpp)
│   │   └── vulkaninfo.hpp  # Auto-generated from vk.xml
│   ├── macOS/              # macOS Metal view support
│   └── iOS/                # iOS support
├── icd/                    # Mock ICD
│   ├── CMakeLists.txt      # ICD build configuration
│   ├── mock_icd.cpp        # Mock ICD implementation
│   ├── mock_icd.h          # Mock ICD data structures
│   ├── VkICD_mock_icd.json.in  # ICD manifest template
│   ├── VkICD_mock_icd.def  # Windows export definitions
│   └── generated/          # Generated headers
│       ├── function_declarations.h  # Auto-generated function declarations
│       ├── function_definitions.h   # Auto-generated function definitions
│       └── vk_typemap_helper.h      # Auto-generated type map helpers
├── scripts/                # Code generation and build scripts
│   ├── CMakeLists.txt      # Scripts build configuration
│   ├── generate_source.py  # Main code generation entry point
│   ├── kvt_genvk.py        # Vulkan tools code generation wrapper
│   ├── common_codegen.py   # Shared code generation utilities
│   ├── android.py          # Android build automation script
│   ├── update_deps.py      # Dependency fetcher/updater
│   ├── known_good.json     # Pinned dependency versions
│   └── generators/         # Code generator implementations
│       ├── mock_icd_generator.py       # Mock ICD code generator
│       ├── vulkaninfo_generator.py     # VulkanInfo code generator
│       └── vulkan_tools_helper_file_generator.py  # Helper file generator
└── tests/                  # Test suite
    ├── CMakeLists.txt      # Test build configuration
    ├── main.cpp            # Test runner
    ├── test_common.h       # Test utilities
    └── icd/
        └── mock_icd_tests.cpp  # Mock ICD unit tests
```

---

## Build System

### CMake Configuration

The project uses **CMake 3.22.1+** as its build system with the following key options:

| Option | Default | Description |
|--------|---------|-------------|
| `BUILD_CUBE` | ON | Build vkcube and vkcubepp |
| `BUILD_VULKANINFO` | ON | Build vulkaninfo |
| `BUILD_ICD` | ON | Build Mock ICD |
| `BUILD_TESTS` | OFF | Build test suite |
| `BUILD_WERROR` | OFF | Treat warnings as errors |
| `TOOLS_CODEGEN` | OFF | Enable code generation target |
| `UPDATE_DEPS` | OFF | Auto-fetch dependencies |
| `INSTALL_ICD` | OFF | Install Mock ICD to system |
| `ENABLE_ADDRESS_SANITIZER` | OFF | Enable ASan |

### Compiler Requirements

- **C Standard**: C99
- **C++ Standard**: C++17 (required)
- **Visibility**: Hidden by default (`CMAKE_CXX_VISIBILITY_PRESET`, `CMAKE_C_VISIBILITY_PRESET`)
- **PIC**: Position Independent Code enabled (`CMAKE_POSITION_INDEPENDENT_CODE ON`)

### Standard Build Commands

```bash
# Quick build with auto-dependencies
cmake -S . -B build -D UPDATE_DEPS=ON -D CMAKE_BUILD_TYPE=Release
cmake --build build

# Development build with tests and warnings-as-errors
cmake -S . -B build -D UPDATE_DEPS=ON -D BUILD_WERROR=ON -D BUILD_TESTS=ON -D CMAKE_BUILD_TYPE=Debug
cmake --build build
```

---

## Module: cube (VkCube Demo)

### Purpose

The `cube` module provides a spinning 3D cube demo that validates Vulkan rendering functionality. It exists in two versions:
- **vkcube** (C API, `cube.c`) — Direct C-style Vulkan calls
- **vkcubepp** (C++ API, `cube.cpp`) — C++ wrapper Vulkan calls

### Key Files

| File | Description |
|------|-------------|
| `cube.c` | Main C demo (~2000+ lines), contains full Vulkan pipeline setup |
| `cube.cpp` | C++ version of the demo |
| `cube_functions.h` | Vulkan function pointer declarations and dynamic loading |
| `cube.vert` / `cube.frag` | GLSL vertex/fragment shaders |
| `cube.vert.inc` / `cube.frag.inc` | Pre-compiled SPIR-V shader bytecode (C arrays) |
| `linmath.h` | Lightweight linear algebra (mat4x4, vec4 operations) |
| `gettime.h` | Platform-independent time measurement |
| `xcb_loader.h` | Dynamic XCB library loader |
| `xlib_loader.h` | Dynamic Xlib library loader |
| `wayland_loader.h` | Dynamic Wayland library loader |

### Architecture

The cube demo follows this initialization flow:

1. **Vulkan Library Loading** — Dynamically loads `libvulkan.so` / `vulkan-1.dll`
2. **Instance Creation** — Creates VkInstance with required extensions
3. **Device Selection** — Enumerates physical devices, selects first suitable GPU
4. **Surface Creation** — Platform-specific surface (XCB/Xlib/Wayland/Win32/Android/Metal)
5. **Swapchain Setup** — Creates swapchain with appropriate format and present mode
6. **Render Pipeline** — Creates descriptor sets, pipeline, command buffers
7. **Render Loop** — Spins the cube using model-view-projection matrices

### Dynamic WSI Loading

The cube demo dynamically loads WSI (Window System Integration) libraries at runtime rather than linking them directly. This is handled by platform-specific loader headers:

```c
// xcb_loader.h pattern
static inline xcb_connection_t* (*my_xcb_connect)(const char*, int*);
// Loaded via dlsym/dlopen at runtime
```

### Android Build

On Android, `vkcube` is built as a **shared library** (`MODULE`) rather than an executable, since it runs within an Android Activity. The Android-specific build is in `cube/android/CMakeLists.txt`.

---

## Module: vulkaninfo

### Purpose

`vulkaninfo` reports detailed information about the Vulkan capabilities of the system's GPU(s). It outputs in multiple formats:
- **Text** (default console output)
- **HTML** (browser-viewable report)
- **JSON** (machine-readable, for Vulkan Configurator)

### Key Files

| File | Description |
|------|-------------|
| `vulkaninfo.cpp` | Main entry point and application logic |
| `vulkaninfo.h` | Core data structures (AppInstance, AppGpu, etc.) |
| `vulkaninfo_functions.h` | Vulkan function implementations |
| `outputprinter.h` | Multi-format output printer (text/html/json) |
| `generated/vulkaninfo.hpp` | Auto-generated Vulkan info output code |

### Key Data Structures

- **`AppInstance`** — Wraps VkInstance, manages instance-level queries
- **`AppGpu`** — Wraps VkPhysicalDevice, manages device-level queries
- **`AppDisplay`** / **`AppDisplayMode`** / **`AppDisplayPlane`** — Display information
- **`AppVideoProfile`** — Video encode/decode profile information
- **`Printer`** — Polymorphic output formatter (text/html/json/vkconfig_output)
- **`PrinterCreateDetails`** — Configuration for output format and destination

### Output Architecture

The `Printer` class in `outputprinter.h` implements a stack-based output system:

```
OutputType::text  → Plain text with indentation
OutputType::html  → HTML tables and formatting
OutputType::json  → JSON structured output
OutputType::vkconfig_output → Vulkan Configurator compatible JSON
```

### Error Handling

Two custom exception types:
- **`FileLineException`** — General errors with file:line information
- **`VulkanException`** — Vulkan API errors with VkResult codes

### Helper Templates

- **`GetVector<T>`** / **`GetVectorInit<T>`** — Robust two-call pattern implementation for Vulkan enumeration functions

---

## Module: icd (Mock ICD)

### Purpose

The Mock ICD (`VkICD_mock_icd`) is a test driver that implements the Vulkan API without actual hardware. It returns valid-looking data for all queries, enabling testing of the Vulkan loader and layers without a real GPU.

### Key Files

| File | Description |
|------|-------------|
| `mock_icd.cpp` | Full Mock ICD implementation |
| `mock_icd.h` | Data structures and state management |
| `VkICD_mock_icd.json.in` | ICD manifest template (CMake configured) |
| `VkICD_mock_icd.def` | Windows DLL export definitions |
| `generated/function_declarations.h` | Auto-generated Vulkan function declarations |
| `generated/function_definitions.h` | Auto-generated Vulkan function stubs |
| `generated/vk_typemap_helper.h` | Auto-generated VkType → VkObjectType mapping |

### Architecture

The Mock ICD operates in the `vkmock` namespace and maintains:

- **Handle Management** — Uses `global_unique_handle` counter to generate unique handles
- **Physical Device Map** — Maps VkInstance → VkPhysicalDevice arrays
- **Memory Tracking** — Maps VkDeviceMemory → mapped allocations and sizes
- **Queue Map** — Maps VkDevice → queue family → queue index → VkQueue
- **Buffer State** — Tracks buffer size and device address
- **Image State** — Tracks image memory sizes
- **Command Pool Tracking** — Maps command pools to their command buffers
- **Swapchain Images** — Pre-allocated swapchain image arrays

### Physical Device Limits

`SetLimits()` in `mock_icd.h` configures realistic device limits matching a mid-range GPU profile (4096 max image dimensions, 4 max bound descriptor sets, etc.).

### ICD Manifest

The JSON manifest (`VkICD_mock_icd.json.in`) is configured by CMake with the correct library path:
- Linux: `./libVkICD_mock_icd.so`
- macOS: `./libVkICD_mock_icd.dylib`
- Windows: `.\VkICD_mock_icd.dll`

---

## Module: scripts (Code Generation)

### Purpose

The scripts module handles:
1. **Code Generation** — Generating C++ headers from the Vulkan XML registry (`vk.xml`)
2. **Dependency Management** — Fetching and building known-good dependencies
3. **Android Build Automation** — Streamlining Android APK creation

### Key Files

| File | Description |
|------|-------------|
| `generate_source.py` | Main code generation entry point |
| `kvt_genvk.py` | Vulkan tools generation wrapper |
| `common_codegen.py` | Shared utilities (path helpers, shell commands) |
| `android.py` | Android build automation |
| `update_deps.py` | Dependency fetcher and builder |
| `known_good.json` | Pinned dependency versions |
| `generators/mock_icd_generator.py` | Mock ICD code generator |
| `generators/vulkaninfo_generator.py` | VulkanInfo code generator |
| `generators/vulkan_tools_helper_file_generator.py` | Helper file generator (vk_typemap_helper.h) |

### Code Generation Targets

| Target | Output Directory | Generator |
|--------|-----------------|-----------|
| `vk_typemap_helper.h` | `icd/generated/` | HelperFileOutputGenerator |
| `function_declarations.h` | `icd/generated/` | MockICDOutputGenerator |
| `function_definitions.h` | `icd/generated/` | MockICDOutputGenerator |
| `vulkaninfo.hpp` | `vulkaninfo/generated/` | VulkanInfoGenerator |

### Generation Flow

```
vk.xml (Vulkan Registry)
    ↓
generate_source.py
    ↓
Registry (from Vulkan-Headers/reg.py)
    ↓
┌─────────────────────────┬──────────────────────────┬──────────────────────┐
│ MockICDOutputGenerator  │ HelperFileOutputGenerator │ VulkanInfoGenerator  │
│  ├─ function_declarations.h │  └─ vk_typemap_helper.h  │  └─ vulkaninfo.hpp │
│  └─ function_definitions.h  │                           │                    │
└─────────────────────────┴──────────────────────────┴──────────────────────┘
    ↓ (optional)
clang-format (if available)
```

---

## Module: tests

### Purpose

Unit tests for the Mock ICD, using Google Test (googletest).

### Key Files

| File | Description |
|------|-------------|
| `main.cpp` | Test runner entry point |
| `test_common.h` | Test utility macros and helpers |
| `icd/mock_icd_tests.cpp` | Mock ICD specific tests |

---

## Dependencies

Dependencies are defined in `scripts/known_good.json`:

| Dependency | Version | Purpose | Required |
|-----------|---------|---------|----------|
| **Vulkan-Headers** | v1.4.352 | Vulkan API headers and registry | Yes |
| **MoltenVK** | v1.4.1 | macOS/iOS Vulkan implementation | macOS only |
| **googletest** | v1.14.0 | Test framework | Tests only |
| **Vulkan-Loader** | v1.4.352 | Vulkan runtime loader | macOS/Tests |

### Dependency Resolution

Use `-D UPDATE_DEPS=ON` to auto-fetch dependencies, or manually run:
```bash
python3 scripts/update_deps.py
```

---

## Cross-Compilation Guide

### Linux 64-bit (Native)

```bash
cmake -S . -B build-x86_64 -D UPDATE_DEPS=ON -D CMAKE_BUILD_TYPE=Release \
    -D BUILD_CUBE=ON -D BUILD_VULKANINFO=ON -D BUILD_ICD=ON
cmake --build build-x86_64
cmake --install build-x86_64 --prefix dist/linux-x86_64
```

### Linux 32-bit (Cross-compile on 64-bit host)

```bash
# Install 32-bit toolchain
sudo apt-get install gcc-multilib g++-multilib libx11-dev:i386

# Build with 32-bit flags
export ASFLAGS=--32
export CFLAGS=-m32
export CXXFLAGS=-m32
export PKG_CONFIG_LIBDIR=/usr/lib/i386-linux-gnu

cmake -S . -B build-x86 -D UPDATE_DEPS=ON -D CMAKE_BUILD_TYPE=Release \
    -D CMAKE_C_FLAGS=-m32 -D CMAKE_CXX_FLAGS=-m32 \
    -D BUILD_WSI_XCB_SUPPORT=OFF -D BUILD_WSI_XLIB_SUPPORT=OFF \
    -D BUILD_WSI_WAYLAND_SUPPORT=OFF \
    -D BUILD_CUBE=ON -D BUILD_VULKANINFO=ON -D BUILD_ICD=ON
cmake --build build-x86
cmake --install build-x86 --prefix dist/linux-x86
```

### Android arm64-v8a

```bash
export ANDROID_NDK_HOME=$HOME/Android/Sdk/ndk/27.0.12077973

cmake -S . -B build-android-arm64 \
    -D CMAKE_TOOLCHAIN_FILE=$ANDROID_NDK_HOME/build/cmake/android.toolchain.cmake \
    -D ANDROID_PLATFORM=26 \
    -D CMAKE_ANDROID_ARCH_ABI=arm64-v8a \
    -D CMAKE_ANDROID_STL_TYPE=c++_static \
    -D ANDROID_USE_LEGACY_TOOLCHAIN_FILE=NO \
    -D CMAKE_BUILD_TYPE=Release \
    -D UPDATE_DEPS=ON \
    -G Ninja
cmake --build build-android-arm64
cmake --install build-android-arm64 --prefix dist/android-arm64
```

### Android armeabi-v7a

```bash
cmake -S . -B build-android-arm32 \
    -D CMAKE_TOOLCHAIN_FILE=$ANDROID_NDK_HOME/build/cmake/android.toolchain.cmake \
    -D ANDROID_PLATFORM=26 \
    -D CMAKE_ANDROID_ARCH_ABI=armeabi-v7a \
    -D CMAKE_ANDROID_STL_TYPE=c++_static \
    -D ANDROID_USE_LEGACY_TOOLCHAIN_FILE=NO \
    -D CMAKE_BUILD_TYPE=Release \
    -D UPDATE_DEPS=ON \
    -G Ninja
cmake --build build-android-arm32
cmake --install build-android-arm32 --prefix dist/android-arm32
```

---

## Key Data Structures

### cube_functions.h — Vulkan Function Pointers

```c
// Global Vulkan function pointers (dynamically loaded)
PFN_vkGetInstanceProcAddr vkGetInstanceProcAddr;
PFN_vkEnumerateInstanceExtensionProperties vkEnumerateInstanceExtensionProperties;
PFN_vkCreateInstance vkCreateInstance;

// Platform-specific surface creation
PFN_vkCreateXcbSurfaceKHR vkCreateXcbSurfaceKHR;        // XCB
PFN_vkCreateXlibSurfaceKHR vkCreateXlibSurfaceKHR;      // Xlib
PFN_vkCreateWaylandSurfaceKHR vkCreateWaylandSurfaceKHR; // Wayland
PFN_vkCreateAndroidSurfaceKHR vkCreateAndroidSurfaceKHR; // Android
PFN_vkCreateMetalSurfaceEXT vkCreateMetalSurfaceEXT;     // macOS/iOS
```

### mock_icd.h — Mock ICD State

```cpp
namespace vkmock {
    static mutex_t global_lock;
    static uint64_t global_unique_handle = 1;
    static constexpr uint32_t icd_physical_device_count = 1;
    static constexpr uint32_t icd_swapchain_image_count = 1;

    // Handle tracking maps
    static std::unordered_map<VkInstance, std::array<VkPhysicalDevice, 1>> physical_device_map;
    static std::unordered_map<VkDeviceMemory, std::vector<void*>> mapped_memory_map;
    static std::unordered_map<VkDeviceMemory, VkDeviceSize> allocated_memory_size_map;
    static std::unordered_map<VkDevice, std::unordered_map<uint32_t, std::unordered_map<uint32_t, VkQueue>>> queue_map;

    struct BufferState {
        VkDeviceSize size;
        VkDeviceAddress address;
    };
    static std::unordered_map<VkDevice, std::unordered_map<VkBuffer, BufferState>> buffer_map;
}
```

### vulkaninfo.h — Info Structures

```cpp
struct FileLineException : std::runtime_error { ... };
struct VulkanException : std::runtime_error { ... };

// Forward declarations
struct AppInstance;
struct AppGpu;
struct AppVideoProfile;
struct AppDisplay;
struct AppDisplayMode;
struct AppDisplayPlane;

// pNext chain structures
struct phys_device_props2_chain;
struct phys_device_mem_props2_chain;
struct phys_device_features2_chain;
struct surface_capabilities2_chain;
struct format_properties2_chain;
struct queue_properties2_chain;
```

### outputprinter.h — Output System

```cpp
enum class OutputType { text, html, json, vkconfig_output };

struct PrinterCreateDetails {
    OutputType output_type = OutputType::text;
    bool print_to_file = false;
    std::string file_name = APP_SHORT_NAME ".txt";
    std::string start_string = "";
};

class Printer {
    // Stack-based output system supporting text, HTML, JSON formats
};
```

---

## WSI Platform Support

The cube and vulkaninfo modules support multiple Window System Integration (WSI) platforms:

| Platform | Compile Definition | Build Option | Default |
|----------|-------------------|--------------|---------|
| XCB | `VK_USE_PLATFORM_XCB_KHR` | `BUILD_WSI_XCB_SUPPORT` | ON (Linux) |
| Xlib | `VK_USE_PLATFORM_XLIB_KHR` | `BUILD_WSI_XLIB_SUPPORT` | ON (Linux) |
| Wayland | `VK_USE_PLATFORM_WAYLAND_KHR` | `BUILD_WSI_WAYLAND_SUPPORT` | ON (Linux) |
| DirectFB | `VK_USE_PLATFORM_DIRECTFB_EXT` | `BUILD_WSI_DIRECTFB_SUPPORT` | OFF |
| Display KHR | `VK_USE_PLATFORM_DISPLAY_KHR` | `BUILD_WSI_DISPLAY_SUPPORT` | ON (non-Apple/Android) |
| Win32 | `VK_USE_PLATFORM_WIN32_KHR` | (auto on Windows) | — |
| Android | `VK_USE_PLATFORM_ANDROID_KHR` | (auto on Android) | — |
| Metal | `VK_USE_PLATFORM_METAL_EXT` | (auto on Apple) | — |

### Dynamic Loading Pattern

On Linux, the cube demo dynamically loads WSI libraries using `dlopen`/`dlsym` rather than linking them directly. This allows the binary to run on systems without all WSI libraries installed:

```c
// Pattern from xcb_loader.h
void *libXcb = dlopen("libxcb.so.1", RTLD_LOCAL | RTLD_LAZY);
my_xcb_connect = (xcb_connection_t*(*)(const char*, int*))dlsym(libXcb, "xcb_connect");
```

---

## Code Generation Pipeline

### Overview

The code generation system transforms the Vulkan XML registry (`vk.xml`) into C++ source files that implement Mock ICD functions and VulkanInfo output.

### Entry Point

```bash
# Generate all code
python3 scripts/generate_source.py <path-to-vulkan-headers-registry>

# Incremental update (only changed files)
python3 scripts/generate_source.py <registry> --incremental --generated-version 1.4.352

# Verify generated code matches repo
python3 scripts/generate_source.py <registry> --verify

# CMake integration
cmake -S . -B build -D TOOLS_CODEGEN=ON
cmake --build build --target tools_codegen
```

### Generators

1. **MockICDOutputGenerator** — Produces:
   - `function_declarations.h` — Declares all Vulkan function stubs
   - `function_definitions.h` — Implements all Vulkan function stubs

2. **HelperFileOutputGenerator** — Produces:
   - `vk_typemap_helper.h` — Maps Vulkan types to their ObjectType enums

3. **VulkanInfoGenerator** — Produces:
   - `vulkaninfo.hpp` — Implements VulkanInfo output for all Vulkan structures

### Version Management

When `--generated-version` is passed:
1. Updates `icd/VkICD_mock_icd.json.in` with new API version
2. Updates `CMakeLists.txt` project version
3. Generates code matching the specified Vulkan API version

---

*Generated on 2026-05-28 from branch `main` at commit `e3d18f9`*
