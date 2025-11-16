# qtVlm Dependencies

> A comprehensive guide to all third-party libraries used in qtVlm. Understanding dependencies helps with building, troubleshooting, and maintaining the project.

---

## Table of Contents

1. [Overview](#overview)
2. [Core Dependencies](#core-dependencies)
3. [Bundled Libraries](#bundled-libraries)
4. [Optional Dependencies](#optional-dependencies)
5. [Platform-Specific Dependencies](#platform-specific-dependencies)
6. [Updating Dependencies](#updating-dependencies)
7. [Licensing](#licensing)

---

## Overview

qtVlm uses a mix of:
- **System libraries** (Qt, system libs)
- **Bundled libraries** (in `src/libs/`)
- **Platform-specific libraries** (OS-dependent)

### Dependency Philosophy

We bundle specific versions of critical libraries to:
- Ensure consistent behavior across platforms
- Avoid dependency hell
- Control updates and security patches

---

## Core Dependencies

### Qt Framework

**Version**: 4.8+ or 5.x  
**License**: LGPL v3 / GPL v3  
**Purpose**: Application framework and GUI

**Required Qt Modules:**
```
Qt Core      - Core non-GUI functionality
Qt GUI       - GUI components
Qt Widgets   - Widget set (Qt 5+)
Qt Network   - Network communication
Qt XML       - XML parsing
Qt XMLPatterns - XPath and XSLT
Qt Multimedia - Audio/video (optional)
Qt Concurrent - Multi-threading helpers (optional)
Qt SerialPort - Serial communication (Qt 5+)
```

**Installation:**

```bash
# Ubuntu/Debian
sudo apt-get install qt5-default qtbase5-dev \
    qtmultimedia5-dev libqt5xmlpatterns5-dev \
    libqt5serialport5-dev

# Fedora/RHEL
sudo dnf install qt5-qtbase-devel qt5-qtmultimedia-devel \
    qt5-qtxmlpatterns-devel qt5-qtserialport-devel

# macOS (via Homebrew)
brew install qt5

# Windows
# Download from https://www.qt.io/download
# Install Qt Creator and desired Qt version
```

**Why Qt?**
- Cross-platform (Windows, Linux, macOS)
- Mature and stable
- Excellent documentation
- Large community
- Built-in networking, XML, graphics

---

## Bundled Libraries

These libraries are included in `src/libs/` and built as part of qtVlm.

### 1. bzip2 (Compression)

**Location**: `src/libs/bzip2/`  
**Version**: Custom build  
**License**: BSD-style  
**Purpose**: Data compression/decompression

**Used For:**
- Compressed GRIB files (.bz2)
- Archive handling

**Links:**
- Official: http://www.bzip.org/
- No external installation needed (bundled)

---

### 2. zlib (Compression)

**Location**: `src/libs/zlib-1.2.7/`  
**Version**: 1.2.7  
**License**: zlib License  
**Purpose**: Compression library

**Used For:**
- GRIB file compression
- General data compression
- PNG support

**⚠️ Security Note**: Version 1.2.7 is from 2012. Current version is 1.3.1.

**Recommendation**: Consider updating to latest version for security patches.

**Links:**
- Official: https://www.zlib.net/
- Vulnerabilities: Check CVE database

---

### 3. jasper (JPEG 2000)

**Location**: `src/libs/jasper/`  
**Version**: Custom build  
**License**: JasPer License 2.0  
**Purpose**: JPEG 2000 codec

**Used For:**
- Decoding JPEG 2000 images in GRIB2 files
- Satellite imagery

**Components:**
```
jasper/base/   - Core functionality
jasper/jp2/    - JPEG 2000 format support
jasper/jpc/    - JPEG 2000 codestream
```

**Links:**
- Official: https://www.ece.uvic.ca/~frodo/jasper/
- GitHub: https://github.com/jasper-software/jasper

---

### 4. g2clib (GRIB2 Decoder)

**Location**: `src/libs/g2clib-1.4.0/`  
**Version**: 1.4.0  
**License**: Public Domain  
**Purpose**: GRIB Edition 2 file decoding

**Used For:**
- Parsing GRIB2 weather files
- Extracting meteorological data

**GRIB2 Features:**
- Multiple data representations
- Complex packing methods
- JPEG 2000 compression (via jasper)
- PNG compression

**Links:**
- Official: https://www.nco.ncep.noaa.gov/pmb/codes/GRIB2/

---

### 5. nmealib (NMEA Parser)

**Location**: `src/libs/nmealib/`  
**Version**: Custom  
**License**: LGPL  
**Purpose**: NMEA 0183 protocol parsing

**Used For:**
- GPS data from serial ports
- Real boat tracking
- Parsing GPS sentences (GGA, RMC, etc.)

**Supported Sentences:**
```
$GPGGA - Global Positioning System Fix Data
$GPGLL - Geographic Position
$GPGSA - GPS DOP and active satellites
$GPGSV - GPS Satellites in view
$GPRMC - Recommended minimum specific GPS/Transit data
$GPVTG - Track made good and ground speed
```

**Links:**
- NMEA 0183 Standard: https://www.nmea.org/

---

### 6. libbsb (BSB/KAP Charts)

**Location**: `src/libs/libbsb/`  
**License**: Custom  
**Purpose**: BSB nautical chart format

**Used For:**
- Reading raster nautical charts
- BSB/KAP format support

**BSB Format:**
- Raster nautical charts
- Used by MapTech, NV Charts, etc.
- Geo-referenced images

---

### 7. miniunz (ZIP Extraction)

**Location**: `src/libs/miniunz/`  
**License**: zlib License  
**Purpose**: Minimal ZIP file extraction

**Used For:**
- Extracting compressed archives
- GRIB file archives

**Note**: Depends on zlib

---

### 8. qjson (JSON Parser)

**Location**: `src/libs/qjson/`  
**License**: LGPL v2.1  
**Purpose**: JSON parsing and generation

**Used For:**
- API communication
- Configuration files
- Data exchange with VLM server

**⚠️ Note**: Qt 5+ has native JSON support (`QJsonDocument`).

**Migration Path**:
```cpp
// Old (qjson)
QJson::Parser parser;
QVariant result = parser.parse(jsonData);

// New (Qt 5+)
QJsonDocument doc = QJsonDocument::fromJson(jsonData);
QJsonObject obj = doc.object();
```

**Recommendation**: Migrate to Qt's native JSON for Qt 5+ builds.

---

### 9. qextserialport (Serial Port)

**Location**: `src/libs/qextserialport12/`  
**Version**: 1.2  
**License**: MIT  
**Purpose**: Cross-platform serial port access

**Used For:**
- GPS device communication (NMEA)
- Real boat tracking
- Serial instrument data

**⚠️ Note**: Qt 5.1+ has native serial port support (`QSerialPort`).

**Migration Path**:
```cpp
// Old (qextserialport)
QextSerialPort *port = new QextSerialPort("/dev/ttyUSB0");
port->open(QIODevice::ReadWrite);

// New (Qt 5.1+)
QSerialPort *port = new QSerialPort("/dev/ttyUSB0");
port->open(QIODevice::ReadWrite);
```

**Recommendation**: Migrate to Qt's `QSerialPort` for Qt 5.1+ builds.

---

### 10. sha1 (Hash Function)

**Location**: `src/libs/sha1/`  
**License**: Public Domain  
**Purpose**: SHA-1 hash calculation

**Used For:**
- Data integrity checks
- Password hashing (legacy)

**⚠️ Security Note**: SHA-1 is cryptographically broken. Use for non-security purposes only.

**Files:**
```
sha1.h         - Header
sha1.cpp       - Implementation
testsha1.cpp   - Test program
```

---

## Optional Dependencies

### libgps (GPS Daemon)

**Location**: `src/libs/libgps/` (currently disabled)  
**Status**: Not built by default  
**Purpose**: gpsd client library

**Would Enable:**
- GPS daemon integration on Linux
- Shared GPS access across applications

**To Enable:**
Uncomment in `qtVlm.pro`:
```
INCLUDEPATH += libs/libgps
LIBS += -lgps
```

---

## Platform-Specific Dependencies

### Linux

**System Libraries:**
```bash
# Standard C++ runtime
libstdc++

# X11 (for Qt GUI)
libX11, libXext, libXrender

# OpenGL (for graphics acceleration)
libGL, libGLU

# Audio (for multimedia)
libasound2 (ALSA) or libpulse (PulseAudio)
```

**Optional:**
```bash
# For GPS daemon
gpsd, libgps-dev
```

---

### Windows

**Runtime Libraries:**
```
MSVC Runtime (included with Qt)
or
MinGW Runtime (if using MinGW)
```

**System Components:**
- DirectX (for graphics)
- Windows Media Foundation (for multimedia)

---

### macOS

**Frameworks:**
```
Cocoa.framework       - macOS native UI
CoreFoundation.framework
CoreGraphics.framework
OpenGL.framework      - Graphics
IOKit.framework       - Hardware access
```

---

## Updating Dependencies

### When to Update

**Update if:**
- Security vulnerability discovered
- Critical bug fix available
- New features needed
- Better performance available

**Be Cautious:**
- API changes may break code
- Behavior changes may affect users
- Test thoroughly after updates

### How to Update a Bundled Library

**Example: Updating zlib**

1. **Backup current version:**
   ```bash
   cd src/libs
   mv zlib-1.2.7 zlib-1.2.7.backup
   ```

2. **Download new version:**
   ```bash
   wget https://www.zlib.net/zlib-1.3.1.tar.gz
   tar xzf zlib-1.3.1.tar.gz
   ```

3. **Update build files:**
   ```bash
   # Copy .pro file from old version
   cp zlib-1.2.7.backup/zlib.pro zlib-1.3.1/
   
   # Edit if needed
   nano zlib-1.3.1/zlib.pro
   ```

4. **Update main Makefile:**
   ```bash
   # Edit Makefile to reference new path
   nano ../../Makefile
   # Change zlib-1.2.7 to zlib-1.3.1
   ```

5. **Test build:**
   ```bash
   cd ../..
   make clean
   make
   ```

6. **Test functionality:**
   - Open GRIB files
   - Verify no crashes
   - Check for regressions

7. **Update documentation:**
   - Update this file (DEPENDENCIES.md)
   - Note changes in CHANGELOG
   - Update version in README

---

## Licensing

### License Compatibility

qtVlm is licensed under **GPL v3**. All dependencies must have GPL-compatible licenses.

**Compatible Licenses:**
- ✅ MIT
- ✅ BSD (2-clause, 3-clause)
- ✅ LGPL (v2.1, v3)
- ✅ Public Domain
- ✅ Apache 2.0 (with GPL v3)
- ✅ zlib License

**Incompatible Licenses:**
- ❌ GPL v2 only (without "or later")
- ❌ Proprietary
- ❌ Some Creative Commons (non-code)

### License Summary

| Library | License | GPL Compatible |
|---------|---------|----------------|
| Qt | LGPL v3 / GPL v3 | ✅ |
| bzip2 | BSD-style | ✅ |
| zlib | zlib License | ✅ |
| jasper | JasPer 2.0 | ✅ |
| g2clib | Public Domain | ✅ |
| nmealib | LGPL | ✅ |
| libbsb | Custom | ⚠️ Verify |
| miniunz | zlib License | ✅ |
| qjson | LGPL v2.1 | ✅ |
| qextserialport | MIT | ✅ |
| sha1 | Public Domain | ✅ |

### Attribution Requirements

When distributing qtVlm, include:

1. **qtVlm License** (GPL v3)
2. **Qt License** (LGPL v3 notice)
3. **Third-Party Licenses** (in `src/libs/*/README` or `LICENSE` files)

**Example Attribution:**
```
qtVlm uses the following third-party libraries:

- Qt Framework (https://www.qt.io/) - LGPL v3
- bzip2 (http://www.bzip.org/) - BSD-style
- zlib (https://www.zlib.net/) - zlib License
- jasper (https://github.com/jasper-software/jasper) - JasPer 2.0
- g2clib (NOAA/NCEP) - Public Domain
- ... (list all)

See individual library directories for full license texts.
```

---

## Security Considerations

### Known Issues

1. **zlib 1.2.7** (2012)
   - Multiple CVEs fixed in later versions
   - **Action**: Update to 1.3.1

2. **SHA-1 usage**
   - Cryptographically broken
   - **Action**: Use only for non-security purposes

3. **Third-party libraries**
   - May have undiscovered vulnerabilities
   - **Action**: Regular security audits

### Best Practices

1. **Monitor CVE Databases**
   - Check https://cve.mitre.org/
   - Subscribe to library mailing lists

2. **Regular Updates**
   - Review dependencies quarterly
   - Update critical security fixes immediately

3. **Minimize Attack Surface**
   - Don't enable unnecessary features
   - Validate all external input
   - Use latest stable versions

---

## Troubleshooting

### Common Build Issues

**"Library not found"**
```bash
# Check that library is built
ls src/libs/build/

# Rebuild specific library
cd src/libs/zlib-1.2.7
qmake
make
```

**"Undefined reference"**
```bash
# Check link order in qtVlm.pro
# Libraries should be listed in dependency order
LIBS += -lminiunz -lbz2 -lz  # miniunz needs bz2 and z
```

**Platform-specific issues**
```bash
# Linux: missing -fPIC
# Add to library .pro file:
QMAKE_CXXFLAGS += -fPIC

# Windows: MSVC vs MinGW
# Ensure all libs use same compiler

# macOS: architecture mismatch
# Rebuild all libs for current architecture
```

---

## Future Considerations

### Modernization Opportunities

1. **Replace qjson**
   - Use Qt 5's native JSON
   - Simpler code, fewer dependencies

2. **Replace qextserialport**
   - Use Qt 5's QSerialPort
   - Better maintained, more features

3. **Update zlib**
   - Security patches
   - Performance improvements

4. **Consider using system libraries**
   - For common libs (zlib, bzip2)
   - Pros: Automatic security updates
   - Cons: Version compatibility issues

5. **CMake build system**
   - Better dependency management
   - ExternalProject for downloading libs
   - More flexible than qmake

---

## Resources

### Documentation

- **Qt Documentation**: https://doc.qt.io/
- **GRIB Format**: https://www.nco.ncep.noaa.gov/pmb/docs/grib2/
- **NMEA 0183**: https://www.nmea.org/

### Tools

- **Dependency Analysis**: `ldd` (Linux), `otool -L` (macOS), `dumpbin` (Windows)
- **License Compliance**: FOSSA, Black Duck, FOSSology
- **Vulnerability Scanning**: Snyk, OWASP Dependency-Check

---

## Contributing

Found a dependency issue? See [CONTRIBUTING.md](CONTRIBUTING.md) for how to help!

---

*Last Updated: November 2025*
