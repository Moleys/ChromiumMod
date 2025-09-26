# ChromiumMod - DevTools Detection Bypass

A modified version of Chromium that bypasses common DevTools detection mechanisms used by websites to prevent debugging and inspection.

## Features

This modification includes three main bypasses:

1. **Console Detection Bypass** - Prevents `console.*` and `throw` statements from being detected by DevTools
2. **Debugger Keyword Modification** - Creates custom debugger keywords to bypass detection
3. **Advanced Debugger Bypass** - Complete replacement of debugger functionality with undetectable alternatives

## Prerequisites

### System Requirements
- **OS**: Ubuntu 22.04 (Linux) or Windows with VS2022
- **RAM**: Minimum 16GB, recommended 32GB+
- **Storage**: 100GB+ free space
- **CPU**: Multi-core processor (compilation is resource-intensive)

### Required Tools
- Git
- Python 3
- depot_tools (Google's build tools)

## Quick Start

### 1. Install depot_tools
```bash
# Clone depot_tools
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git

# Add to PATH (add to ~/.bashrc for persistence)
export PATH="${HOME}/depot_tools:$PATH"
```

### 2. Download Chromium Source
```bash
# Create working directory
mkdir chromium && cd chromium/

# Fetch Chromium source (this will take time and significant bandwidth)
fetch --nohooks --no-history chromium
```

### 3. Install Dependencies
```bash
# Enter source directory
cd src

# Install build dependencies (requires sudo)
sudo ./build/install-build-deps.sh

# Download third-party libraries
gclient runhooks
```

## Applying Modifications

### Console Detection Bypass

**Files to modify:**
- `v8/src/inspector/v8-console-message.h`
- `v8/src/inspector/v8-console-message.cc`

**Implementation:**
1. Add default parameter to `addMessage` function in header:
```cpp
// In v8-console-message.h
void addMessage(std::unique_ptr<V8ConsoleMessage>, bool isOutputToDevTools = false);
```

2. Modify implementation to conditionally output:
```cpp
// In v8-console-message.cc
void V8ConsoleMessageStorage::addMessage(
    std::unique_ptr<V8ConsoleMessage> message, bool isOutputToDevTools) {
  // Default behavior: don't output to DevTools
  if (!isOutputToDevTools) return;

  // ... rest of original implementation
}
```

### Debugger Keyword Modification

**Method 1: Simple Token Replacement**
- Replace `debugger` token with `null` token in `v8/src/parsing/keywords-gen.h`
- Add custom keyword like `u_debugger` that functions as the real debugger
- Update keyword detection in `v8/src/parsing/scanner-inl.h`

**Method 2: Advanced Parser Modification**
- Add new keyword `debugpoint` to `v8/src/parsing/keywords.txt`
- Regenerate `keywords-gen.h` using `v8/tools/gen-keywords-gen-h.py`
- Add token definition in `v8/src/parsing/token.h`
- Modify parser in `v8/src/parsing/parser-base.h` to handle new keyword
- Make original `debugger` return empty statement

## Build Configuration

### Debug Build (Development)
```bash
# Generate build configuration
gn gen out/Debug --export-compile-commands --ide=vs2022

# Edit out/Debug/args.gn:
is_debug = true
symbol_level = 2
is_component_build = true
enable_nacl = false
blink_symbol_level = 2
v8_symbol_level = 2
dcheck_always_on = true
```

### Release Build (Production)
```bash
# Generate build configuration
gn gen out/Release --export-compile-commands

# For PGO optimization, update .gclient to include:
# "checkout_pgo_profiles": True
# Then run: gclient runhooks

# Edit out/Release/args.gn:
is_debug = false
symbol_level = 0
is_component_build = false
is_official_build = true
proprietary_codecs = true
ffmpeg_branding = "Chrome"
target_cpu = "x64"
dcheck_always_on = false
```

## Compilation

```bash
# For Debug build
autoninja -C out/Debug chrome

# For Release build
autoninja -C out/Release chrome

# Control parallel jobs if memory is limited
autoninja -C out/Debug chrome -j 4
```

## Running Modified Chrome

```bash
# Debug version
out/Debug/chrome

# Release version
out/Release/chrome
```

## Creating Installer

```bash
# Build minimal installer (Windows)
autoninja -C out/Release mini_installer
```

## Testing

Test the modifications on sites that use DevTools detection:
- https://www.ldvmp.com/
- Other anti-debugging websites

The modified Chrome should successfully bypass:
- Console API detection (`console.log`, `console.error`, etc.)
- Debugger statement detection
- Exception throwing detection
- DevTools opening detection

## Project Structure

```
魔改/
├── console/           # Console detection bypass documentation
│   ├── 笔记.md       # Implementation notes
│   └── images/       # Screenshots
├── debugger/         # Basic debugger modification
│   ├── 笔记.md       # Implementation guide
│   ├── debugger.patch # Patch file
│   └── Hash.py       # Hash calculation script
└── debugger2.0/      # Advanced debugger bypass
    ├── 笔记.md       # Advanced implementation
    ├── keywords-gen.h # Generated keyword definitions
    └── images/       # Test results
```

## Build Times

- **Initial Build**: 2-6 hours (depending on hardware)
- **Incremental Builds**: 5-30 minutes
- **Disk Usage**: ~50-100GB during build

## Troubleshooting

### Out of Memory Errors
```bash
# Reduce parallel jobs
autoninja -C out/Debug chrome -j 2
```

### Build Failures
```bash
# Clean and rebuild
rm -rf out/Debug
gn gen out/Debug --export-compile-commands
autoninja -C out/Debug chrome
```

### Missing Dependencies
```bash
# Re-sync dependencies
gclient runhooks
```

## Official Documentation

- [Linux Build Instructions](https://chromium.googlesource.com/chromium/src/+/main/docs/linux/build_instructions.md)
- [Windows Build Instructions](https://chromium.googlesource.com/chromium/src/+/master/docs/windows_build_instructions.md)

## Original Project Notes

v8 文件下会放一些分析 v8 源码的文章

devtools-frontend 文件下会放一些分析 devtools 源码的文章

魔改文件夹下存放的是经过我缜密的分析后实操的文章

希望能帮到大家。

## Legal Notice

This project is for educational and research purposes. Users are responsible for complying with applicable laws and website terms of service. The modifications are designed for legitimate debugging and development use cases.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Test your modifications thoroughly
4. Submit a pull request with detailed documentation

## License

This project follows the same license terms as the Chromium project.