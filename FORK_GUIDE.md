# Firefox Fork Quick Reference

A condensed guide for systems programmers working with a Firefox fork.

## Quick Start

```bash
# Clone your fork
git clone https://github.com/YOUR_USERNAME/firefox.git
cd firefox

# Install build dependencies
./mach bootstrap

# Configure build
./mach configure

# Build Firefox
./mach build

# Run Firefox
./mach run
```

## Directory Quick Map

```
browser/          → Firefox UI (tabs, menus, extensions)
dom/              → DOM, JavaScript APIs
gfx/              → Graphics, rendering, text
js/               → JavaScript engine (SpiderMonkey)
layout/           → CSS, HTML layout
netwerk/          → Network, HTTP, DNS
toolkit/          → Shared code
security/         → TLS, sandbox, CSP
storage/          → DB, cache, IndexedDB
extensions/       → WebExtensions API
media/            → Audio/video codecs
devtools/         → Developer tools
testing/          → Tests and CI
mobile/           → Android/mobile code
build/            → Build system
```

## Common Commands

```bash
# Building
./mach build                    # Full build
./mach build dom/              # Build specific component
./mach clobber                 # Clean build
./mach build -j4               # Parallel build (4 cores)

# Running & Debugging
./mach run                      # Run Firefox
./mach run --args="-purgecaches"    # Run with arguments
./mach debug                    # Debug with gdb/lldb
gdb --args firefox              # Direct gdb (if built)

# Testing
./mach test mochitest           # Web platform tests
./mach test xpcshell            # XPConnect shell tests
./mach test gtest               # C++ unit tests
./mach test dom/tests/mochitest # Specific test directory
./mach test --single-threaded   # No parallel execution

# Code Quality
./mach lint                     # Lint all code
./mach lint dom/                # Lint specific directory
./mach format                   # Auto-format code

# Profiling
./mach perf                     # Performance tests
./mach pydoc                    # Python documentation
./mach help                     # Full command reference
```

## Modifying Core Components

### Browser UI
**Location**: `browser/base/content/`, `browser/components/`
**Files to edit**: `.xul`, `.html`, `.js`
**Rebuild**: `./mach build browser`
**Test**: `./mach test mochitest`

Example - Add preference:
1. Edit `browser/app/profile/prefs.js`
2. Access via `Services.prefs.getBoolPref("my.pref")`
3. Test and verify

### Network Stack
**Location**: `netwerk/`
**Files to edit**: `.cpp`, `.h`
**Key files**: `nsHttpChannel.cpp`, `nsStreamLoader.cpp`
**Rebuild**: `./mach build netwerk`
**Test**: `./mach test netwerk`

### JavaScript Engine
**Location**: `js/`
**Files to edit**: C++ source in `vm/`, `builtin/`
**Rebuild**: `./mach build js`
**Test**: `./mach test jit_test` (C++ tests)

### Web APIs (DOM)
**Location**: `dom/`
**Structure**: `dom/base/`, `dom/fetch/`, `dom/html/`, etc.
**Rebuild**: `./mach build dom`
**Test**: `./mach test dom/tests/mochitest`

### Rendering Engine
**Location**: `layout/`, `gfx/`
**Key areas**: 
  - `layout/generic/` - Layout algorithms
  - `gfx/` - Graphics, compositing
**Rebuild**: `./mach build layout gfx`

## Configuration Files

### `.mozconfig` (Build Configuration)
Create this in Firefox root for persistent build settings:
```python
# Plain build
ac_add_options --disable-debug
ac_add_options --enable-optimize=2

# Development build (with debug symbols)
ac_add_options --enable-debug
ac_add_options --disable-optimize

# Custom options
ac_add_options --disable-tests
ac_add_options --enable-rust-tests
```

### `browser/app/profile/prefs.js` (Default Preferences)
Add your custom defaults:
```javascript
pref("my.custom.setting", true);
pref("my.custom.number", 42);
```

## Maintaining Your Fork

```bash
# Add upstream as remote
git remote add upstream https://github.com/mozilla-firefox/firefox.git

# Fetch updates
git fetch upstream main

# Rebase your changes on top of latest
git rebase upstream/main

# Or merge if you prefer
git merge upstream/main

# Push to your fork
git push origin your-branch -f
```

## Debugging Common Issues

### Build Errors
```bash
# Clear and rebuild
./mach clobber
./mach bootstrap  # Reinstall dependencies
./mach build

# Check specific error
./mach build 2>&1 | tail -50  # Last 50 lines
```

### Runtime Crashes
```bash
# Debug mode
./mach debug
(gdb) run
(gdb) bt          # Backtrace
(gdb) frame 0     # Go to frame
(gdb) p variable  # Print variable
(gdb) l           # List source

# Or use logs
./mach run --args="-console"  # Open console at startup
```

### Test Failures
```bash
./mach test mochitest --verbose
./mach test dom --debugger      # Run with debugger
./mach test --single-threaded   # Avoid race conditions
```

## Key Concepts

- **XPCOM**: Old component model (being phased out)
- **MDN/Web APIs**: Modern JavaScript APIs
- **Spidermonkey**: JS engine integrated in browser
- **Gecko**: Rendering engine (layout + graphics)
- **Content process**: Sandboxed page rendering
- **IPC**: Inter-process communication
- **Rust modules**: Performance/safety critical code

## Platform Specific

### Linux
```bash
./mach bootstrap --application=firefox
./mach build
./mach run
```

### Windows
```bash
# Use MozillaBuild tools
start mach.ps1
.\mach bootstrap --application=firefox
.\mach build
.\mach run
```

### macOS
```bash
./mach bootstrap
./mach build
./mach run  # App bundle: obj-*/dist/FirefoxBrowser.app
```

## Essential Tools to Install

```bash
# Install linting tools
./mach bootstrap

# Python packages (optional, included in bootstrap)
pip install cmake
pip install psutil

# For profiling
sudo apt-get install valgrind  # Linux only
```

## Performance Tips

1. **Use ccache**: Speeds up rebuilds
   ```bash
   ./mach bootstrap --ccache
   ```

2. **Incremental builds**: Only rebuild what changed
   ```bash
   ./mach build dom/  # Instead of full build
   ```

3. **Parallel compilation**: Use `-j` flag
   ```bash
   ./mach build -j8   # 8 cores
   ```

4. **Disable debug**: For faster builds
   - In `.mozconfig`: `ac_add_options --disable-debug`

5. **Disable tests**: If you don't need them
   - In `.mozconfig`: `ac_add_options --disable-tests`

## Git Tips for Forking

```bash
# Create feature branch
git checkout -b feature/my-feature

# Work on your changes
git add .
git commit -m "Description of changes"

# Keep in sync with upstream
git fetch upstream
git rebase upstream/main

# Push to your fork
git push origin feature/my-feature

# Create PR on GitHub (if contributing back)
# Otherwise, just maintain your fork branch
```

## Code Style

- **C++**: Follow moz.build style guide
  - `./mach lint` checks automatically
  - `./mach format` auto-fixes many issues

- **JavaScript**: ESLint configuration in `.eslintrc.mjs`
  - Run: `./mach lint --fix`

- **Python**: PEP 8 style
  - Run: `./mach lint path/to/file.py`

## Testing Strategy

1. **Unit tests**: For individual functions
   ```bash
   ./mach test gtest  # C++ tests
   ```

2. **Integration tests**: For component interaction
   ```bash
   ./mach test mochitest  # Web platform tests
   ```

3. **Regression tests**: Before releasing
   ```bash
   ./mach test smoothness     # Rendering performance
   ./mach test startup        # Startup time
   ```

4. **Manual testing**:
   ```bash
   ./mach run
   # Use your modified Firefox
   # Check functionality
   # Check console for errors
   ```

## Division of Labor (Multiple Developers)

For teams working on fork:

1. **Assign components**: Each module to a team member
2. **Use branches**: Feature branches for isolation
3. **PR reviews**: Review changes before merge
4. **Testing**: Dedicated QA for integration
5. **Documentation**: Keep wiki/docs updated
6. **Meetings**: Sync on blocking issues

## Release Checklist

- [ ] All tests passing
- [ ] Code review complete
- [ ] Security review (if security-sensitive)
- [ ] Performance benchmarks acceptable
- [ ] Documentation updated
- [ ] Compatibility verified
- [ ] Build on all platforms
- [ ] Create release notes
- [ ] Tag release: `git tag v1.0.0`
- [ ] Build distribution packages

## Troubleshooting Matrix

| Problem | Solution |
|---------|----------|
| "mach: command not found" | Run from Firefox root, not subdirectory |
| Build timeout | Increase timeout, check disk space |
| Memory exhaustion | Reduce parallel jobs: `./mach build -j2` |
| Linker errors | `./mach clobber`, ensure dependencies installed |
| Test hangs | Kill process, run `--single-threaded` |
| Crash on startup | Check preferences, run with `--console` |
| Git conflicts | See git guide; usually `git rebase --continue` |

## Resources

- Firefox Source Docs: https://firefox-source-docs.mozilla.org/
- Main README: [README.md](README.md)
- Full Codebase Guide: [CODEBASE_GUIDE.md](CODEBASE_GUIDE.md)
- Mozilla Matrix Chat: chat.mozilla.org#introduction:mozilla.org
- Bugzilla: https://bugzilla.mozilla.org/ (Mozilla's issue tracker)

---

**Last Updated: March 2026**
**For updates, check Firefox Source Docs**
