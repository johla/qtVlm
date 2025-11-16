# Contributing to qtVlm

> **Welcome!** 👋 We're excited that you're interested in contributing to qtVlm. This guide is written to help you get started, whether you're fixing a typo or adding a major feature. Every contribution matters!

---

## Table of Contents

1. [Code of Conduct](#code-of-conduct)
2. [Getting Started](#getting-started)
3. [Development Setup](#development-setup)
4. [How to Contribute](#how-to-contribute)
5. [Coding Standards](#coding-standards)
6. [Commit Guidelines](#commit-guidelines)
7. [Pull Request Process](#pull-request-process)
8. [Testing Guidelines](#testing-guidelines)
9. [Documentation](#documentation)
10. [Getting Help](#getting-help)

---

## Code of Conduct

### Our Pledge

We are committed to making participation in qtVlm a harassment-free experience for everyone, regardless of:
- Age, body size, disability
- Ethnicity, gender identity and expression
- Level of experience
- Nationality, personal appearance
- Race, religion, or sexual identity and orientation

### Our Standards

**Positive behaviors include:**
- Using welcoming and inclusive language
- Being respectful of differing viewpoints and experiences
- Gracefully accepting constructive criticism
- Focusing on what is best for the community
- Showing empathy towards other community members

**Unacceptable behaviors include:**
- Trolling, insulting/derogatory comments, and personal or political attacks
- Public or private harassment
- Publishing others' private information without explicit permission
- Other conduct which could reasonably be considered inappropriate

---

## Getting Started

### Prerequisites

Before you begin, ensure you have:

1. **Qt Development Tools**
   - Qt 5.x or later (Qt Creator recommended)
   - Qt development libraries
   - qmake build tool

2. **C++ Compiler**
   - GCC 4.8+ (Linux)
   - MSVC 2015+ (Windows)
   - Clang 3.4+ (macOS)

3. **Version Control**
   - Git installed and configured
   - GitHub account (for pull requests)

4. **Optional Tools**
   - Qt Creator IDE (highly recommended)
   - Debugger (gdb, lldb, or MSVC debugger)
   - Valgrind (for memory leak detection on Linux)

### Knowledge Requirements

**Helpful to know:**
- C++11 or later
- Qt Framework basics (signals/slots, widgets, QML if applicable)
- GRIB file format (for weather-related contributions)
- Basic sailing/navigation concepts (helpful but not required)

**Don't worry if you don't know everything!** We're here to help you learn.

---

## Development Setup

### 1. Fork and Clone

```bash
# Fork the repository on GitHub, then:
git clone https://github.com/YOUR_USERNAME/qtVlm.git
cd qtVlm
git remote add upstream https://github.com/johla/qtVlm.git
```

### 2. Install Dependencies

#### Linux (Ubuntu/Debian)
```bash
sudo apt-get update
sudo apt-get install qt5-default qtmultimedia5-dev \
    libqt5serialport5-dev build-essential
```

#### macOS
```bash
brew install qt5
# Add Qt to PATH
export PATH="/usr/local/opt/qt/bin:$PATH"
```

#### Windows
1. Download and install Qt from https://www.qt.io/download
2. Ensure Qt's bin directory is in your PATH
3. Install Visual Studio or MinGW

### 3. Build qtVlm

```bash
# From the repository root
make clean  # Clean any previous build artifacts
make        # Build all libraries and the main application

# Or using Qt Creator:
# 1. Open qtVlm.pro in Qt Creator
# 2. Configure project with your Qt kit
# 3. Click Build → Build All
```

### 4. Run qtVlm

```bash
# On Linux/macOS
./qtVlm

# On Windows
qtVlm.exe

# Or use Qt Creator's Run button
```

### 5. Verify Setup

- Application should start without errors
- You should see the main map window
- Try opening a GRIB file to ensure weather data works
- Check that menus and toolbar respond correctly

**If you encounter issues**, see the [Getting Help](#getting-help) section.

---

## How to Contribute

### Types of Contributions Welcome

We welcome all kinds of contributions:

- 🐛 **Bug fixes** - Found something broken? Fix it!
- ✨ **New features** - Have an idea? Implement it!
- 📝 **Documentation** - Help others understand the code
- 🌍 **Translations** - Make qtVlm available in your language
- 🎨 **UI improvements** - Make qtVlm more user-friendly
- ⚡ **Performance** - Speed up slow operations
- 🧪 **Tests** - Add test coverage (we need this!)
- 🔧 **Code refactoring** - Improve code quality

### Finding Something to Work On

1. **Check Issues**:
   - Look for issues labeled `good first issue` or `help wanted`
   - Browse open bugs that interest you

2. **Ask First**:
   - Before starting major work, open an issue to discuss
   - This prevents duplicate effort and ensures your approach fits

3. **Start Small**:
   - Your first contribution doesn't have to be huge
   - Even fixing typos or improving comments is valuable

---

## Coding Standards

### Code Style

We aim for consistency. While we don't have a strict style guide yet, follow these principles:

#### Naming Conventions

```cpp
// Classes: PascalCase
class MainWindow { };
class GribReader { };

// Methods/Functions: camelCase
void calculateRoute();
bool isVisible();

// Variables: camelCase
double windSpeed;
QString userName;

// Constants: ALL_CAPS or camelCase for const
#define MAX_BOATS 100
const int maxIterations = 1000;

// Private members: consider using m_ prefix or trailing underscore
class MyClass {
private:
    int m_counter;     // or
    QString name_;
};
```

#### Indentation & Formatting

```cpp
// Use 4 spaces for indentation (not tabs)
class Example {
public:
    void method() {
        if (condition) {
            // Code here
        }
    }
};

// Braces on same line for control structures
if (condition) {
    doSomething();
} else {
    doSomethingElse();
}

// Class/function braces can be on new line (Qt style)
class MyClass
{
public:
    MyClass()
    {
        // Constructor
    }
};
```

#### Comments

```cpp
// Single-line comments for brief explanations
int count = 0;  // Number of boats

/**
 * Multi-line comments for complex explanations.
 * Use Doxygen style for documentation.
 * 
 * @param lat Latitude in degrees
 * @param lon Longitude in degrees
 * @return Distance in nautical miles
 */
double calculateDistance(double lat, double lon);

// Explain WHY, not WHAT (code shows what)
// BAD: Increment counter
count++;

// GOOD: Track boats for resource cleanup
count++;  // Ensures all boats are released on exit
```

### Qt-Specific Guidelines

#### Use Qt Types

```cpp
// Prefer Qt types over STL when working with Qt
QString text;          // Not std::string
QList<int> numbers;    // Not std::vector<int>
QMap<QString, int> m;  // Not std::map<std::string, int>
```

#### Signal/Slot Connections

```cpp
// Use new (Qt 5) connection syntax when possible
connect(button, &QPushButton::clicked,
        this, &MyClass::onButtonClicked);

// Old syntax still works but is less type-safe
// connect(button, SIGNAL(clicked()), this, SLOT(onButtonClicked()));
```

#### Memory Management

```cpp
// Objects with parents are auto-deleted
MyWidget *widget = new MyWidget(parent);
// No need to delete manually

// For objects without parents, use smart pointers or deleteLater()
QTimer *timer = new QTimer();
timer->deleteLater();  // Safe deletion in event loop
```

### Architecture Guidelines

1. **Separation of Concerns**
   - Keep UI code separate from business logic
   - Use signals/slots for component communication
   - Avoid tight coupling between classes

2. **Single Responsibility**
   - Each class should have one clear purpose
   - If a class does too much, consider splitting it

3. **Prefer Composition Over Inheritance**
   - Use Qt's parent-child system
   - Delegate to contained objects rather than inheriting

4. **Error Handling**
   ```cpp
   // Check return values
   if (!file.open(QIODevice::ReadOnly)) {
       qWarning() << "Failed to open file:" << file.fileName();
       return false;
   }
   
   // Provide context in errors
   if (!grib->loadFile(fileName)) {
       QMessageBox::warning(this, tr("Error"), 
           tr("Could not load GRIB file: %1").arg(fileName));
       return;
   }
   ```

---

## Commit Guidelines

### Commit Messages

Write clear, descriptive commit messages:

```
Short summary (50 chars or less)

More detailed explanation if needed. Wrap at 72 characters.
Explain the problem that this commit is solving and why the
change is being made.

- Bullet points are okay
- Use present tense ("Add feature" not "Added feature")
- Reference issues: "Fixes #123" or "Related to #456"
```

**Examples:**

```
✅ Good:
Add support for wave height visualization

Implements GRIB wave height parameter display with color-coded
overlays. Users can toggle wave display from View menu.

Fixes #234

✅ Good:
Fix crash when loading corrupted GRIB files

Added validation for GRIB header before parsing. Shows error
dialog instead of crashing.

❌ Bad:
Fixed stuff

❌ Bad:
WIP

❌ Bad:
Updated code
```

### Commit Frequency

- Commit early and often
- Each commit should be a logical unit
- Don't commit broken code to main branch
- Squash "fix typo" commits before PR

---

## Pull Request Process

### Before Submitting

1. **Update your branch**
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

2. **Test thoroughly**
   - Build without warnings
   - Test your changes manually
   - Run existing tests (if any)
   - Check for regressions

3. **Review your changes**
   ```bash
   git diff upstream/main
   ```
   - Remove debug code
   - Remove commented-out code
   - Check for unintended changes

### Submitting a Pull Request

1. **Push to your fork**
   ```bash
   git push origin your-branch-name
   ```

2. **Open PR on GitHub**
   - Click "New Pull Request"
   - Choose base: `johla/qtVlm:main`
   - Choose compare: `your-username/qtVlm:your-branch-name`

3. **Fill out PR template**
   ```markdown
   ## Description
   Brief description of what this PR does
   
   ## Type of Change
   - [ ] Bug fix
   - [ ] New feature
   - [ ] Documentation
   - [ ] Code refactoring
   
   ## Testing Done
   Describe how you tested this
   
   ## Screenshots (if applicable)
   Show UI changes
   
   ## Related Issues
   Fixes #123
   ```

4. **Respond to feedback**
   - Be open to suggestions
   - Make requested changes
   - Push updates to same branch

### PR Review Process

**What reviewers look for:**
- Code correctness
- Adherence to coding standards
- Proper error handling
- Performance considerations
- Documentation updates
- Test coverage

**Timeline:**
- Initial review: Usually within 1 week
- Follow-up: Depends on discussion
- Be patient! Reviewers are volunteers

---

## Testing Guidelines

### Manual Testing

**Before submitting, test:**

1. **Basic functionality**
   - Application starts without errors
   - No crashes during normal use
   - UI is responsive

2. **Your specific changes**
   - Test all code paths
   - Try edge cases
   - Test error conditions

3. **Cross-platform (if possible)**
   - Windows, Linux, macOS
   - Different Qt versions

### Automated Testing (Future)

We're working on adding unit tests. For now:

```cpp
// If adding testable code, write it in a testable way:

// ❌ Hard to test (tightly coupled)
void MyClass::doSomething() {
    double data = DataManager::getInstance()->getData();
    // use data...
}

// ✅ Easy to test (dependency injection)
void MyClass::doSomething(DataManager *manager) {
    double data = manager->getData();
    // use data...
}
```

### Test Checklist

Before submitting:
- [ ] Builds without errors or warnings
- [ ] Runs without crashes
- [ ] Feature works as intended
- [ ] No regressions in existing features
- [ ] Tested with sample data (GRIB files, routes, etc.)
- [ ] Memory leaks checked (valgrind on Linux)
- [ ] No performance degradation

---

## Documentation

### Code Documentation

#### Header Files (.h)

```cpp
/**
 * @class MyClass
 * @brief One-line description of the class
 * 
 * Detailed description of what this class does,
 * its responsibilities, and how to use it.
 */
class MyClass : public QObject
{
    Q_OBJECT
    
public:
    /**
     * @brief Constructor
     * @param parent Parent QObject for memory management
     */
    MyClass(QObject *parent = nullptr);
    
    /**
     * @brief Calculate something important
     * @param input Input value in range [0, 100]
     * @return Calculated result in nautical miles
     * @throws std::invalid_argument if input out of range
     */
    double calculate(double input);
};
```

#### Implementation Files (.cpp)

```cpp
// Explain complex algorithms
double MyClass::calculate(double input)
{
    // Haversine formula for great circle distance
    // https://en.wikipedia.org/wiki/Haversine_formula
    
    double a = sin(dlat/2) * sin(dlat/2) +
               cos(lat1) * cos(lat2) *
               sin(dlon/2) * sin(dlon/2);
    
    // ... rest of calculation
}
```

### User Documentation

If your change affects users:

1. **Update README** (for installation/usage changes)
2. **Add release notes** (describe new features)
3. **Create screenshots** (for UI changes)
4. **Update help text** (in dialogs/tooltips)

### Translation

If adding user-visible text:

```cpp
// Always use tr() for translatable strings
QString message = tr("Wind speed: %1 knots").arg(speed);

// Not this:
QString message = QString("Wind speed: %1 knots").arg(speed);
```

---

## Getting Help

### Where to Ask Questions

1. **GitHub Issues** - For bug reports and feature requests
2. **Discussions** - For general questions and ideas
3. **Code Comments** - For specific implementation questions in PRs

### Debugging Tips

#### Application Crashes

```bash
# Run with debugger
gdb ./qtVlm
(gdb) run
# ... crash occurs ...
(gdb) backtrace

# Or in Qt Creator: Debug → Start Debugging (F5)
```

#### Memory Issues

```bash
# Linux: Check for leaks with valgrind
valgrind --leak-check=full ./qtVlm

# macOS: Use Instruments (Leaks template)

# Windows: Use Visual Studio diagnostic tools
```

#### Qt Issues

```cpp
// Enable Qt debug output
qDebug() << "Variable value:" << myVar;
qWarning() << "Something unexpected happened";

// Or set environment variable
export QT_DEBUG_PLUGINS=1
```

### Common Problems

**Build Errors:**
- **"Qt not found"**: Ensure Qt is in PATH
- **"Unknown module"**: Install missing Qt component
- **Link errors**: Check that all libraries are built

**Runtime Errors:**
- **Segfault on startup**: Check plugin paths
- **Cannot load GRIB**: Verify file format and libraries
- **Blank window**: Check graphics drivers

---

## Recognition

Contributors will be:
- Listed in release notes
- Credited in the application (About dialog)
- Appreciated in the community! 🎉

---

## License

By contributing to qtVlm, you agree that your contributions will be licensed under the GNU General Public License v3.0.

---

## Final Words

**Thank you for contributing to qtVlm!** 

Every line of code, every bug report, every documentation improvement makes this project better. Whether this is your first open-source contribution or your hundredth, we appreciate your time and effort.

**Remember:**
- Ask questions if something is unclear
- Don't be discouraged by feedback - it's how we all improve
- Small contributions are just as valuable as large ones
- Have fun! This is a hobby project for most of us

**Happy sailing and happy coding!** ⛵💻

---

*Questions about this guide? Open an issue and we'll improve it!*
