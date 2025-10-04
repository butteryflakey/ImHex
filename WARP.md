# WARP.md

This file provides guidance to WARP (warp.dev) when working with code in this repository.

## Project Overview

ImHex is a feature-rich hex editor for reverse engineers, programmers, and security researchers. It's written in C++23 and uses CMake for build configuration. The architecture is heavily plugin-based, with most functionality implemented as plugins that load into a minimal main application shell.

## Development Commands

### Building

**Prerequisites**: C++23-compatible compiler (GCC-14 or Clang), CMake 3.25+, Ninja

```bash
# Clone with submodules (required)
git clone https://github.com/WerWolv/ImHex --recurse-submodules

# Install dependencies (choose appropriate script for your system)
./dist/get_deps_arch.sh        # Arch Linux
./dist/get_deps_debian.sh      # Ubuntu/Debian  
./dist/get_deps_fedora.sh      # Fedora/RHEL
./dist/get_deps_msys2.sh       # Windows (MSYS2)

# Build for development (Debug)
mkdir -p build
cd build
CC=gcc-14 CXX=g++-14 cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Debug ..
ninja

# Build for release
CC=gcc-14 CXX=g++-14 cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="/usr" ..
ninja install
```

### Testing

```bash
# Build and run unit tests
cmake -DIMHEX_ENABLE_UNIT_TESTS=ON ..
ninja
ninja unit_tests

# Build and run plugin tests
cmake -DIMHEX_ENABLE_PLUGIN_TESTS=ON ..
ninja
```

### Development Utilities

```bash
# Language file management
python3 dist/langtool.py create plugins/builtin/romfs/lang <iso_code>  # Create new language
python3 dist/langtool.py translate plugins/builtin/romfs/lang <iso_code>  # Update existing

# Run with debug/development settings
./build/imhex --help  # See available command line options
```

## Architecture Overview

### Core Components

**Main Application (`main/`)**: Minimal shell that creates window, OpenGL context, loads plugins, and renders ImGui interface. Most functionality delegated to plugins.

**LibImHex (`lib/libimhex/`)**: Core library providing dependency inversion layer. Contains APIs for plugins to interact with ImHex and each other. Key APIs in `hex/api/` directory include:
- Content Registry: Register new content types, tools, patterns
- Provider API: Data source abstraction (files, memory, network)
- UI API: Views, popups, widgets, themes
- Event System: Plugin communication

**Plugin System (`plugins/`)**: All major functionality implemented as plugins:
- `builtin/`: Core ImHex functionality (hex editor, tools, views)
- `disassembler/`: Capstone-based disassembly  
- `hashes/`: Cryptographic hash algorithms
- `visualizers/`: Data visualization tools
- `script_loader/`: Pattern Language and scripting
- Plugin Template: https://github.com/WerWolv/ImHex-Plugin-Template

### Key Architectural Patterns

**Plugin-Based Architecture**: Main application knows nothing about plugins at build time. All communication goes through libimhex APIs. Even "builtin" features are implemented as a plugin.

**Provider Pattern**: Data sources (files, memory, network) abstracted behind Provider interface. Supports overlays, undo/redo, caching.

**RomFS Embedding**: Static resources embedded at compile time using libromfs. Plugin resources in `romfs/` directories get embedded into binaries.

**Pattern Language**: Custom C++-like DSL for parsing and highlighting binary data structures. Separate project: https://github.com/WerWolv/PatternLanguage

### Build System Details

**CMake Options**: 30+ configuration options for different build scenarios (see CMakeLists.txt:3-30)
- `IMHEX_PLUGINS_IN_SHARE`: Plugin installation location
- `IMHEX_STATIC_LINK_PLUGINS`: Static vs dynamic plugin linking
- `IMHEX_ENABLE_UNIT_TESTS`: Enable test builds
- `IMHEX_OFFLINE_BUILD`: For airgapped environments

**Dependencies**: Mix of system packages and bundled libraries
- System: OpenGL 3.0+, GLFW, various crypto libraries
- Bundled: ImGui, PatternLanguage, many others in `lib/third_party/`

### Development Workflow

1. **Feature Development**: Most new features should be implemented as plugins or in the builtin plugin
2. **API Changes**: Changes to libimhex APIs may require updates to all dependent plugins
3. **Resource Changes**: After modifying `romfs/` contents, re-run CMake to regenerate embedded resources
4. **Pattern Language**: Changes go to separate PatternLanguage repository, not ImHex directly

### Important File Locations

- Plugin APIs: `lib/libimhex/include/hex/api/`
- Main plugin code: `plugins/builtin/source/`
- Embedded resources: `plugins/*/romfs/` 
- Build configurations: `CMakePresets.json`, platform-specific scripts in `dist/compiling/`
- Language files: `plugins/builtin/romfs/lang/`