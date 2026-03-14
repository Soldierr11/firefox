# Firefox Codebase Guide 

A comprehensive guide to understanding the Firefox source code structure, architecture, and development workflow for new systems programmers who want to fork and modify Firefox.

## Table of Contents
1. [Quick Overview](#quick-overview)
2. [Architecture](#architecture)
3. [Directory Structure](#directory-structure)
4. [Core Components](#core-components)
5. [Technologies Used](#technologies-used)
6. [Build System](#build-system)
7. [Development Workflow](#development-workflow)
8. [Getting Started](#getting-started)
9. [Common Tasks](#common-tasks)
10. [Resources](#resources)

---

## Quick Overview

Firefox is a complex, modern web browser written primarily in C++, JavaScript, and Rust. The codebase includes:

- **~20 million lines of code** (approximately)
- **Multi-platform support** (Windows, macOS, Linux, Android, iOS)
- **Gecko rendering engine** (custom HTML/CSS/DOM renderer)
- **SpiderMonkey JavaScript engine**
- **Multiple security and privacy features**

The browser is designed with layers of abstraction, separating concerns like networking, rendering, DOM manipulation, and UI.

---

## Architecture

### High-Level Layers

```
┌─────────────────────────────────────────────────────────────┐
│                    Browser UI Layer                          │
│  (User Interface, Tabs, Menus, Extensions, Preferences)     │
│                  [browser/, toolkit/]                        │
├─────────────────────────────────────────────────────────────┤
│                  Core Browser Services                       │
│  (Security, Networking, Storage, IPC, Updates)              │
│      [netwerk/, security/, storage/, ipc/, services/]       │
├─────────────────────────────────────────────────────────────┤
│                  Web Platform Layer                          │
│  (DOM, JavaScript, CSS, HTML Parser, Web APIs)              │
│          [dom/, js/, layout/, parser/, media/]              │
├─────────────────────────────────────────────────────────────┤
│                 Rendering Engine (Gecko)                    │
│  (Graphics, Layout, Text Rendering, Image Decoding)         │
│          [gfx/, layout/, image/, media/]                    │
├─────────────────────────────────────────────────────────────┤
│                   Platform Abstraction                       │
│  (OS-specific code, Hardware Access, Windowing)             │
│        [widget/, hal/, mozglue/, nsprpub/]                  │
├─────────────────────────────────────────────────────────────┤
│                   Operating System                           │
│              (Windows, macOS, Linux, Android)                │
└─────────────────────────────────────────────────────────────┘
```

### Key Architectural Concepts

1. **Process Model**: Firefox uses multiple processes for security and stability
   - Main process: Browser UI and coordination
   - Content processes: Web page rendering (sandboxed)
   - GPU process: Graphics acceleration
   - Network process: Network operations

2. **IPC (Inter-Process Communication)**: Actors system for safe process communication

3. **Component System**: Modular architecture using XPCOM and newer component systems

4. **Sandboxing**: Security model that isolates untrusted content

---

## Directory Structure

### Top-Level Directories

| Directory | Purpose |
|-----------|---------|
| **browser/** | Firefox desktop browser UI, menus, tabs, extensions integration |
| **dom/** | DOM implementation, Web APIs, event handling |
| **gfx/** | Graphics, rendering, text layout, image handling |
| **js/** | SpiderMonkey JavaScript engine |
| **layout/** | CSS and HTML layout engine |
| **netwerk/** | Network stack, HTTP, WebSocket, DNS |
| **toolkit/** | Shared code for multiple Mozilla applications |
| **media/** | Audio/video codecs, playback, streaming |
| **security/** | Certificate handling, encryption, CSP, sandbox |
| **storage/** | Database, cache, IndexedDB, localStorage |
| **testing/** | Test frameworks, automation, CI configuration |
| **devtools/** | Developer tools (Inspector, Debugger, Console) |
| **extensions/** | WebExtensions API implementation |
| **mobile/** | Android-specific code (GeckoView) |
| **build/** | Build system scripts and tools |
| **python/** | Python build infrastructure |
| **third_party/** | Vendored dependencies |
| **xpcom/** | Component object model, low-level utilities |
| **intl/** | Internationalization, text handling |
| **xpfe/** | Ancient UI framework (mostly deprecated) |
| **accessible/** | Accessibility (screen readers, a11y) |
| **parser/** | HTML and XML parsers |
| **editor/** | Text editor component |
| **docshell/** | Document loading and navigation |
| **uriloader/** | URI loading and protocol handling |

---

## Core Components

### 1. **Browser UI** (`browser/`)
- Firefox desktop interface
- Tabs, menu bar, toolbars, sidebar
- Extensions and add-ons integration
- Preferences/settings UI
- About pages, error pages

### 2. **Content Process** (`dom/`, `layout/`, `gfx/`)
- Renders web pages
- Executes JavaScript
- Handles user interaction on pages
- Applies CSS styling

### 3. **JavaScript Engine** (`js/`)
- SpiderMonkey JS interpreter
- JIT compilation
- Garbage collection
- Modern ECMAScript support

### 4. **Rendering Engine (Gecko)** (`layout/`, `gfx/`)
- CSS selector matching
- Box model, flexbox, grid layout
- Paint and composite
- Hardware acceleration

### 5. **Network Stack** (`netwerk/`)
- HTTP/HTTPS protocol handling
- DNS resolution
- Caching
- Cookie management
- WebSocket, HTTP/2, HTTP/3 support

### 6. **Security** (`security/`, `caps/`)
- TLS/SSL certificate validation
- Sandbox enforcement
- Content Security Policy (CSP)
- Same-Origin Policy
- Cryptographic operations

### 7. **Developer Tools** (`devtools/`)
- Inspector
- Debugger
- Console
- Performance profiler
- Network monitor

### 8. **Media** (`media/`)
- Audio/video playback
- Codec support
- Media element API
- WebRTC

### 9. **Storage** (`storage/`)
- Cookies
- Session storage
- Local storage
- IndexedDB
- Service Worker cache

### 10. **Accessibility** (`accessible/`)
- Screen reader support
- ARIA implementation
- Keyboard navigation

---

## Technologies Used

### Languages
- **C++**: Core engine, rendering, networking (70%+ of codebase)
- **JavaScript**: Browser UI, pages content, extensions
- **Rust**: New components, cryptography, performance-critical code
- **Python**: Build scripts, testing infrastructure
- **HTML/CSS**: UI markup and styling (XUL, XHTML)

### Key Libraries
- **Gecko**: Custom rendering engine
- **SpiderMonkey**: JavaScript engine
- **OpenSSL/NSS**: Cryptography
- **zlib**: Compression
- **libVPX**: Video codec
- **Skia**: Graphics library (2D rendering)
- **WebRTC**: Real-time communication

### Programming Paradigms
- **Object-oriented**: Heavy use of classes and inheritance
- **Component-based**: XPCOM interface system
- **Functional**: Some JavaScript functional patterns
- **Async/await**: Modern event-driven architecture

---

## Build System

### Tools
- **mach**: Primary build tool (Python-based)
- **moz.build**: Build configuration files (Python-like syntax)
- **moz.configure**: Configuration system
- **GCC/Clang**: C++ compiler
- **Cargo**: Rust package manager
- **gradle**: Android build (for GeckoView)

### Build Configuration
```bash
# Check if you have dependencies
./mach --help

# Configure build
./mozilla-build/start-shell.bat  # Windows
./mach configure

# Build
./mach build

# Run Firefox
./mach run

# Run tests
./mach test

# Build specific component
./mach build dom/
```

### moz.build Files
- Found in each directory
- Define what to build, source files, flags
- Similar to Makefiles but more readable

---

## Development Workflow

### 1. **Clone and Setup**
```bash
git clone https://github.com/mozilla/firefox.git
cd firefox
./mach bootstrap  # Install dependencies
```

### 2. **Configuration**
```bash
# Edit mozconfig file or:
./mach configure

# Common configuration:
# - Optimization level
# - Debug symbols
# - Compiler flags
```

### 3. **Build**
```bash
./mach build           # Full build
./mach clobber        # Clean build
./mach build dom/     # Specific component
```

### 4. **Testing**
```bash
./mach test mochitest  # Web platform tests
./mach test xpcshell   # JavaScript tests
./mach test gtest      # C++ unit tests
```

### 5. **Running**
```bash
./mach run             # Run Firefox
./mach debug           # Run with debugger
```

### 6. **Code Changes**
- Edit files in relevant directories
- Rebuild affected components
- Test changes
- Create patch or commit

---

## Getting Started with Modifications

### Easy Starting Points

1. **Change Default Settings** (`browser/app/profile/`)
   - Modify default preferences
   - No rebuilding needed (mostly)

2. **Add Browser Extensions**
   - Most features can be added as WebExtensions
   - No core modification needed

3. **Modify UI** (`browser/base/content/`)
   - Change toolbar buttons, menus
   - Relatively contained changes

4. **Add URL Patterns** (`browser/components/urlbar/`)
   - Customize address bar behavior

5. **Change Themes** (`browser/themes/`)
   - Customize colors, fonts, appearance

### Moderate Modifications

1. **Add Network Feature** (`netwerk/`)
   - Custom protocol handler
   - Caching strategy changes
   - HTTP optimization

2. **Modify Rendering** (`gfx/`, `layout/`)
   - CSS support changes
   - Layout algorithm modifications
   - Graphics optimization

3. **Extend JavaScript** (`js/`)
   - Add custom built-in functions
   - Modify object behavior

### Advanced Modifications

1. **Change Security Model** (`security/`, `caps/`)
   - Modify sandbox rules
   - Change policy system
   - Cryptography algorithms

2. **Add New Web API** (`dom/`)
   - New DOM methods
   - New browser APIs
   - Custom event systems

3. **Optimize Performance** (everywhere)
   - Profiling and benchmarking
   - Algorithm improvements
   - Memory optimizations

---

## Common Tasks

### Adding a New Preference
1. Define in `browser/app/profile/prefs.js`
2. Access in C++ via `Services.prefs.getBoolPref()`
3. Access in JS via `Services.prefs.getBoolPref()`

### Adding a New Extension API
1. Define in `browser/components/extensions/`
2. Add permission requirements
3. Implement in C++ backend
4. Test with extension

### Modifying Network Behavior
1. Edit relevant files in `netwerk/`
2. Modify `nsHttpChannel.cpp` or similar
3. Rebuild `netwerk/` component
4. Test with web requests

### Changing UI Elements
1. Edit XUL/HTML in `browser/base/content/`
2. Edit CSS in `browser/themes/`
3. Modify JavaScript event handlers in `browser/base/content/`
4. Rebuild browser component

### Debugging Issues
```bash
./mach debug              # Run with debugger
./mach test --debugger    # Run test with debugger
./mach test --gdb         # Use gdb specifically
```

---

## Code Organization Patterns

### File Naming Conventions
- `.cpp`: C++ implementation
- `.h`: C++ header
- `.rs`: Rust implementation
- `.jsm`: JavaScript module
- `.js`: JavaScript (browser UI)
- `.xul`: XUL (old UI format)
- `.html`: HTML UI

### Header Guards
```cpp
#ifndef mymodule_h__
#define mymodule_h__
// content
#endif // mymodule_h__
```

### Interface Pattern (XPCOM)
```cpp
// .h file
class MyComponent : public nsIComponent {
public:
  NS_DECL_ISUPPORTS
  NS_DECL_MYCOMPONENT
};

// .cpp file
NS_IMPL_ISUPPORTS(MyComponent, nsIComponent)
```

### Tests
- Unit tests in `*_unittest.cpp` in same directory
- Integration tests in `testing/mochitest/`
- Run: `./mach test path/to/test`

---

## Building Your Fork

### Fork Strategy

1. **Create a branch for your modifications**
   ```bash
   git checkout -b my-custom-firefox
   ```

2. **Document your changes**
   - Keep track of modifications
   - Create README for your fork
   - Document build instructions

3. **Maintain against upstream**
   ```bash
   git fetch upstream
   git rebase upstream/main
   ```

4. **Publish your fork**
   ```bash
   git push origin my-custom-firefox
   ```

### Release Checklist
- [ ] All tests passing
- [ ] Documentation updated
- [ ] Build on all platforms tested
- [ ] Extension compatibility verified
- [ ] No security regressions
- [ ] Performance benchmarks run

---

## Performance Considerations

- **Startup time**: Affected by extension loading, network operations
- **Memory usage**: DOM tree size, image caching, service workers
- **Rendering performance**: Layout complexity, image decoding
- **JavaScript execution**: JIT compilation overhead, script parsing
- **Network**: HTTP/3 support, DNS, caching strategies

### Profiling Tools
```bash
./mach test --profiler  # Profile tests
./mach perf             # Performance testing
```

---

## Security Considerations for Forks

1. **Update CA certificates** regularly
2. **Keep cryptography libraries** current
3. **Review security patches** from Mozilla
4. **Sandboxing**: Understand and maintain the security model
5. **Extension permissions**: Be careful with granting permissions
6. **Network security**: TLS version support, certificate pinning

---

## Key Files to Know

| File | Purpose |
|------|---------|
| `./mach` | Main build tool |
| `./mozilla-config.h.in` | Platform configuration template |
| `./moz.configure` | Build configuration |
| `./Cargo.toml` | Rust dependencies |
| `./package.json` | Node.js dependencies |
| `browser/app/profile/prefs.js` | Default preferences |
| `.mozconfig` | User build configuration |

---

## Learning Resources

### Official Documentation
- [Firefox Source Docs](https://firefox-source-docs.mozilla.org/)
- [Contributing Guide](https://firefox-source-docs.mozilla.org/contributing/contribution_quickref.html)
- [Architecture Overview](https://firefox-source-docs.mozilla.org/contributing/architecture/)

### Local Documentation
- `docs/` directory in source
- Comments in source code
- Commit messages (searchable via git)

### External Resources
- Mozilla Developer Network (MDN)
- GitHub issues and discussions
- Matrix chat: chat.mozilla.org

---

## Tips for Success

1. **Start small**: Make single, focused changes
2. **Test frequently**: Build and test after each change
3. **Use version control**: Commit often with meaningful messages
4. **Read code**: Understanding existing patterns is crucial
5. **Profile before optimizing**: Don't guess, measure
6. **Check CI**: Run tests before pushing changes
7. **Document changes**: Future you will appreciate it
8. **Join community**: Ask questions, get feedback
9. **Monitor security**: Keep up with security advisories
10. **Plan releases**: Regular, stable release cycles

---

## Common Issues and Solutions

### Build Fails
- Run `./mach clean` or `./mach clobber`
- Check `mozconfig` settings
- Ensure all dependencies installed via `./mach bootstrap`
- Check available disk space (40GB+ recommended)

### Tests Fail
- Rebuild affected component: `./mach build dom/`
- Run specific test with verbose output
- Check for race conditions in parallel tests
- Run serially: `./mach test --single-threaded`

### Runtime Crashes
- Use debugger: `./mach debug`
- Check browser console for errors
- Look at extension compatibility
- Profile with Performance tab in DevTools

### Performance Issues
- Use profiler in DevTools
- Check for N+1 problems in loops
- Profile startup vs runtime
- Consider using Telemetry for metrics

---

## Next Steps

1. **Clone the repository** and build Firefox
2. **Read the Architecture Overview** in the docs
3. **Explore a component** that interests you
4. **Make a small change** and rebuild
5. **Write tests** for your changes
6. **Submit patches** (if contributing back to Mozilla)
7. **Maintain your fork** with upstream updates

---

**Happy hacking! Questions? Check the Matrix chat or Firefox Source Docs.**
