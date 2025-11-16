# qtVlm Build Guide

> **Welcome!** This guide will help you build qtVlm from source, whether you're on Linux, Windows, or macOS. We've tried to make this as straightforward as possible, but if you get stuck, don't hesitate to ask for help!

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Prerequisites](#prerequisites)
3. [Building on Linux](#building-on-linux)
4. [Building on Windows](#building-on-windows)
5. [Building on macOS](#building-on-macos)
6. [Build Options](#build-options)
7. [Troubleshooting](#troubleshooting)
8. [IDE Setup](#ide-setup)

---

## Quick Start

**For the impatient:**

```bash
# Clone the repository
git clone https://github.com/johla/qtVlm.git
cd qtVlm

# Build everything
make

# Run
./qtVlm  # Linux/macOS
# or
qtVlm.exe  # Windows
```

**That's it!** If this works for you, great! If not, read on for detailed instructions.

---

## Prerequisites

### Required Software

| Component | Minimum Version | Recommended | Purpose |
|-----------|----------------|-------------|---------|
| **Qt** | 4.8 | Qt 5.12+ | GUI framework |
| **C++ Compiler** | C++11 | C++17 | Code compilation |
| **qmake** | 4.8 | Qt 5.x | Build system |
| **make** | Any | GNU Make | Build orchestration |
| **Git** | Any | Latest | Version control |

### Compiler Requirements

**Linux:**
- GCC 4.8+ or Clang 3.4+
- `sudo apt-get install build-essential`

**Windows:**
- MSVC 2015+ (Visual Studio) or
- MinGW-w64 5.3+

**macOS:**
- Xcode Command Line Tools
- `xcode-select --install`

---

## Building on Linux

### Ubuntu/Debian

#### 1. Install Dependencies

```bash
# Update package list
sudo apt-get update

# Install Qt 5 and development tools
sudo apt-get install -y \
    qt5-default \
    qtbase5-dev \
    qtmultimedia5-dev \
    libqt5xmlpatterns5-dev \
    libqt5serialport5-dev \
    build-essential \
    git

# Optional: Qt Creator IDE
sudo apt-get install qtcreator
```

#### 2. Clone Repository

```bash
git clone https://github.com/johla/qtVlm.git
cd qtVlm
```

#### 3. Build

```bash
# Clean build (recommended for first time)
make clean

# Build all libraries and main application
make

# This will:
# 1. Build all bundled libraries (bzip2, zlib, jasper, etc.)
# 2. Build the main qtVlm application
# 3. Create the qtVlm executable in the current directory
```

#### 4. Run

```bash
./qtVlm
```

#### 5. Install (Optional)

```bash
# Edit Makefile to set install location
nano Makefile
# Change: prefix = ~/qtVlm
# To your desired path

# Install
make install

# This copies:
# - qtVlm executable
# - Resource files (icons, images, polars)
# - Translation files
```

---

### Fedora/RHEL/CentOS

```bash
# Install dependencies
sudo dnf install -y \
    qt5-qtbase-devel \
    qt5-qtmultimedia-devel \
    qt5-qtxmlpatterns-devel \
    qt5-qtserialport-devel \
    gcc-c++ \
    make \
    git

# Build (same as Ubuntu)
git clone https://github.com/johla/qtVlm.git
cd qtVlm
make
```

---

### Arch Linux

```bash
# Install dependencies
sudo pacman -S --needed \
    qt5-base \
    qt5-multimedia \
    qt5-xmlpatterns \
    qt5-serialport \
    base-devel \
    git

# Build
git clone https://github.com/johla/qtVlm.git
cd qtVlm
make
```

---

## Building on Windows

### Method 1: Qt Creator (Recommended)

#### 1. Install Qt

1. Download Qt installer from https://www.qt.io/download
2. Run installer and select:
   - Qt 5.15.x (or latest)
   - Qt Creator
   - MinGW 64-bit compiler (or use your own MSVC)

#### 2. Clone Repository

```powershell
# Using Git Bash or PowerShell
git clone https://github.com/johla/qtVlm.git
cd qtVlm
```

#### 3. Open in Qt Creator

1. Launch Qt Creator
2. File → Open File or Project
3. Navigate to `qtVlm/src/qtVlm.pro`
4. Select your Qt kit (Desktop Qt 5.15.x MinGW/MSVC)
5. Click "Configure Project"

#### 4. Build

1. Click the hammer icon (Build) or press Ctrl+B
2. Wait for compilation (may take several minutes)
3. Check "Compile Output" tab for any errors

#### 5. Run

1. Click the play icon (Run) or press Ctrl+R
2. qtVlm should launch

---

### Method 2: Command Line (Advanced)

#### Prerequisites

1. Install Qt (as above)
2. Add Qt to PATH:
   ```powershell
   # Example for Qt 5.15.2 with MinGW
   $env:PATH += ";C:\Qt\5.15.2\mingw81_64\bin"
   $env:PATH += ";C:\Qt\Tools\mingw810_64\bin"
   ```

#### Build

```powershell
# Clone
git clone https://github.com/johla/qtVlm.git
cd qtVlm

# Build libraries
cd src\libs\bzip2
qmake
mingw32-make
cd ..\..\..

# (Repeat for each library or use Makefile)

# Build main application
cd src
qmake CONFIG+=release
mingw32-make
cd ..
```

#### Run

```powershell
.\qtVlm.exe
```

#### Create Installer

1. Copy qtVlm.exe to installation folder
2. Copy required DLLs:
   ```powershell
   windeployqt qtVlm.exe
   ```
3. Copy resource folders:
   - base_dir/icon → install_dir/icon
   - base_dir/img → install_dir/img
   - base_dir/polar → install_dir/polar
   - base_dir/dataColors.dat → install_dir/
   - tr/ → install_dir/tr

---

## Building on macOS

### 1. Install Xcode

```bash
# Install Xcode Command Line Tools
xcode-select --install
```

### 2. Install Qt

**Option A: Homebrew (Recommended)**

```bash
# Install Homebrew if not already installed
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Qt
brew install qt@5

# Add to PATH
echo 'export PATH="/usr/local/opt/qt@5/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

**Option B: Official Qt Installer**

1. Download from https://www.qt.io/download
2. Install Qt 5.15.x for macOS

### 3. Build

```bash
# Clone
git clone https://github.com/johla/qtVlm.git
cd qtVlm

# Build
make

# This creates qtVlm.app bundle
```

### 4. Run

```bash
# From command line
./qtVlm.app/Contents/MacOS/qtVlm

# Or double-click qtVlm.app in Finder
```

### 5. Create DMG (Optional)

```bash
# Use macdeployqt to bundle dependencies
macdeployqt qtVlm.app -dmg

# This creates qtVlm.dmg for distribution
```

---

## Build Options

### Configuration Options

Edit `src/qtVlm.pro` before building:

#### Qt Version

```qmake
# Detect Qt version automatically
contains(QT_VERSION, "^5.*") {
    DEFINES += QT_V5
}
```

#### Debug vs Release

```bash
# Debug build (larger, with debug symbols)
cd src
qmake CONFIG+=debug
make

# Release build (optimized, smaller)
qmake CONFIG+=release
make
```

#### Address Sanitizer (for developers)

```bash
# Enable memory error detection
qmake CONFIG+=asan
make
```

This enables:
- Buffer overflow detection
- Use-after-free detection
- Memory leak detection

**Note**: Significantly slower, use for debugging only.

#### Platform-Specific Defines

Automatically set based on platform:

```qmake
# Linux
unix:!macx: DEFINES += __UNIX_QTVLM

# Windows
win32: DEFINES += __WIN_QTVLM

# macOS
macx: DEFINES += __MAC_QTVLM
```

---

## Troubleshooting

### Common Issues

#### "Qt not found"

**Symptoms:**
```
Project ERROR: Unknown module(s) in QT: widgets
```

**Solution:**
```bash
# Linux: Install Qt development packages
sudo apt-get install qt5-default qtbase5-dev

# Ensure qmake is in PATH
which qmake  # Should show path to Qt's qmake

# If using custom Qt installation, set QTDIR
export QTDIR=/path/to/qt
export PATH=$QTDIR/bin:$PATH
```

---

#### Build errors in bundled libraries

**Symptoms:**
```
error: unknown type name 'uchar'
```

**Solution:**
```bash
# Clean and rebuild
make clean
cd src/libs/problematic-lib
qmake
make
cd ../../..
make
```

If persistent, check:
- Compiler version (need C++11)
- Platform-specific code (#ifdef blocks)

---

#### "Cannot find -lqjson" or similar link errors

**Symptoms:**
```
/usr/bin/ld: cannot find -lqjson
```

**Solution:**
```bash
# Ensure libraries are built first
make libs

# Check that libraries exist
ls src/libs/build/

# Should see: libbz2.a, libqjson.a, libz.a, etc.

# If missing, rebuild specific library:
cd src/libs/qjson
qmake
make
```

---

#### Segmentation fault on startup

**Possible causes:**

1. **Missing resources**
   ```bash
   # Ensure you're running from the qtVlm directory
   # Or set up proper installation with make install
   ```

2. **Qt plugin issues**
   ```bash
   # Set debug flag
   export QT_DEBUG_PLUGINS=1
   ./qtVlm
   # Check output for missing plugins
   ```

3. **Library incompatibilities**
   ```bash
   # Check linked libraries
   ldd qtVlm  # Linux
   otool -L qtVlm  # macOS
   
   # Ensure all Qt libraries are from same version
   ```

---

#### Black or blank window

**Cause**: Graphics driver issues

**Solution:**
```bash
# Try software rendering
export QT_XCB_GL_INTEGRATION=xcb_egl  # Linux
./qtVlm

# Or disable OpenGL
export QT_OPENGL=software
./qtVlm
```

---

#### Build takes too long

**Speed up:**
```bash
# Use parallel compilation
make -j$(nproc)  # Linux: use all CPU cores
make -j8         # Use 8 cores

# In Qt Creator:
# Tools → Options → Build & Run → General
# Increase "Parallel build jobs"
```

---

### Platform-Specific Issues

#### Linux: Missing libraries

```bash
# If application won't start due to missing .so files
# Check dependencies
ldd qtVlm | grep "not found"

# Install missing packages
# Example for missing libQt5Multimedia
sudo apt-get install libqt5multimedia5
```

---

#### Windows: MSVC vs MinGW

**Issue**: Mixing MSVC and MinGW builds

**Solution**: 
- Rebuild everything with one compiler
- Clean build directory first
- Use matching Qt build (MSVC Qt with MSVC, MinGW Qt with MinGW)

---

#### macOS: Architecture mismatch

**Issue**: Building for wrong architecture (Intel vs Apple Silicon)

**Solution:**
```bash
# Check Qt architecture
file /usr/local/opt/qt/bin/qmake

# Match your system:
# Intel: x86_64
# Apple Silicon: arm64

# If mismatch, install correct Qt version
arch -x86_64 brew install qt@5  # For Intel
arch -arm64 brew install qt@5   # For Apple Silicon
```

---

## IDE Setup

### Qt Creator (All Platforms)

#### Setup

1. **Open Project**: File → Open File or Project → `src/qtVlm.pro`

2. **Configure Kit**:
   - Select Qt version
   - Select compiler
   - Click "Configure Project"

3. **Build Settings**:
   - Projects (left sidebar)
   - Build Settings tab
   - Choose Debug or Release

4. **Run Settings**:
   - Projects → Run
   - Working directory: Set to qtVlm root (where base_dir/ is)

#### Debugging

1. Set breakpoints: Click line number margin
2. Start debugging: F5 or Debug → Start Debugging
3. Use debug views: Variables, Call Stack, Breakpoints

#### Keyboard Shortcuts

| Action | Shortcut |
|--------|----------|
| Build | Ctrl+B |
| Run | Ctrl+R |
| Debug | F5 |
| Step Over | F10 |
| Step Into | F11 |
| Toggle Breakpoint | F9 |
| Switch header/source | F4 |
| Find in Files | Ctrl+Shift+F |

---

### Visual Studio (Windows Only)

#### Setup

1. Install Qt Visual Studio Tools extension
2. Qt VS Tools → Qt Options → Add Qt version
3. File → Open → Project → `src/qtVlm.pro`
4. Qt VS Tools → Convert to Qt VS project

#### Building

- Build → Build Solution (Ctrl+Shift+B)
- Debug → Start Debugging (F5)

---

### VS Code (All Platforms)

#### Extensions

Install these extensions:
- C/C++ (Microsoft)
- Qt tools
- CMake Tools (if using CMake)

#### Configuration

Create `.vscode/c_cpp_properties.json`:
```json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/src/**",
                "/usr/include/x86_64-linux-gnu/qt5/**"
            ],
            "compilerPath": "/usr/bin/g++",
            "cStandard": "c11",
            "cppStandard": "c++17"
        }
    ]
}
```

#### Tasks

Create `.vscode/tasks.json`:
```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "build",
            "type": "shell",
            "command": "make",
            "group": {
                "kind": "build",
                "isDefault": true
            }
        }
    ]
}
```

Build: Ctrl+Shift+B

---

## Performance Tips

### Optimization Flags

For maximum performance:

```qmake
# In qtVlm.pro
release {
    QMAKE_CXXFLAGS += -O3 -march=native
    DEFINES += QT_NO_DEBUG_OUTPUT
}
```

### Profile-Guided Optimization (PGO)

```bash
# 1. Build with profiling
qmake CONFIG+=release QMAKE_CXXFLAGS+="-fprofile-generate"
make

# 2. Run typical workload
./qtVlm  # Use application normally, then quit

# 3. Rebuild with profile data
make clean
qmake CONFIG+=release QMAKE_CXXFLAGS+="-fprofile-use"
make
```

---

## Packaging for Distribution

### Linux

#### AppImage

```bash
# Use linuxdeployqt
wget https://github.com/probonopd/linuxdeployqt/releases/download/continuous/linuxdeployqt-continuous-x86_64.AppImage
chmod +x linuxdeployqt-continuous-x86_64.AppImage

# Create AppImage
./linuxdeployqt-continuous-x86_64.AppImage qtVlm -appimage
```

#### Debian Package

```bash
# Use checkinstall
sudo checkinstall --pkgname=qtvlm --pkgversion=3.4.3 make install
```

---

### Windows

```powershell
# Use windeployqt
windeployqt --release qtVlm.exe

# Creates folder with all dependencies
# Package with NSIS or Inno Setup
```

---

### macOS

```bash
# Create app bundle with dependencies
macdeployqt qtVlm.app -dmg

# Result: qtVlm.dmg ready for distribution
```

---

## Continuous Integration

### GitHub Actions Example

```yaml
name: Build

on: [push, pull_request]

jobs:
  build-linux:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Install Qt
        run: sudo apt-get install qt5-default qtbase5-dev
      - name: Build
        run: make
      - name: Test
        run: ./qtVlm --version
```

---

## Getting Help

**Build issues?**

1. **Check this guide** - Most issues are covered here
2. **Search issues** - Someone may have had the same problem
3. **Ask in discussions** - Community can help
4. **File an issue** - If it's a genuine bug

**Include when asking for help:**
- Operating system and version
- Qt version (`qmake --version`)
- Compiler version (`g++ --version` or `cl.exe`)
- Full error message
- Steps to reproduce

---

## Next Steps

After successfully building:

1. **Read ARCHITECTURE.md** - Understand the codebase
2. **See CONTRIBUTING.md** - Start contributing
3. **Check CODE_ASSESSMENT.md** - Learn about code quality
4. **Explore the code!** - Best way to learn

---

**Happy Building!** 🛠️

If you successfully built qtVlm, congratulations! You're now ready to contribute or customize it to your needs. ⛵

---

*Questions about this guide? Open an issue and we'll improve it!*
