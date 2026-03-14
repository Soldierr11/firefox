# Systems Programming with Firefox: Deep Dive

A technical guide for systems programmers making low-level modifications to Firefox internals.

## Architecture Deep Dive

### Process Model

Firefox uses a multi-process architecture for security and stability:

```
Main Process (Browser UI, Tab Manager)
    ├─ Content Process 1 (www.example.com)
    ├─ Content Process 2 (www.google.com)
    ├─ GPU Process (Graphics acceleration)
    ├─ Network Process (HTTP requests)
    ├─ VR Process (VR functionality)
    └─ Various utility processes
```

**Key Files**:
- `dom/base/nsGlobalWindowInner.cpp` - Main window object
- `dom/ipc/` - IPC for DOM
- `browser/app/nsBrowserApp.cpp` - Browser startup
- `ipc/glue/` - IPC framework (Actors)

### Memory Layout

```cpp
// Typical Firefox memory structure:
// 1. Static RO data - Read-only code/strings
// 2. Code sections - Executable machine code
// 3. Heap - Dynamic allocations (most of memory)
// 4. Stack - Local variables, function frames
// 5. Mmap regions - Files, graphics buffers
```

**Memory Profiling**:
```bash
./mach run --args="-dmdump"  # DMD memory profiling
./mach perf memory            # Memory performance tests
```

### Threading Model

Firefox uses multiple threads:

- **Main thread**: UI, DOM manipulation, JavaScript
- **Layout thread**: CSS parsing, layout calculations
- **Network thread**: HTTP, DNS (in Network Process)
- **GPU thread**: Graphics operations
- **Worker threads**: Async tasks, service workers
- **GC thread**: Garbage collection

**Thread-Safe Code**:
```cpp
// Use nsAutoString for thread-local storage
// Use Mutex for shared data
#include "mozilla/Mutex.h"

mozilla::Mutex mMutex("MyLock");
mozilla::MutexAutoLock lock(mMutex);
// Critical section
```

### Garbage Collection

Firefox uses a two-tier GC system:

1. **Cycle Collector**: For JavaScript/XPCOM cycles
   - `xpcom/base/nsCycleCollector.cpp`
   - Runs periodically
   - Collects circular references

2. **SpiderMonkey GC**: For JavaScript objects
   - `js/public/GC.h`
   - Generational and incremental
   - Tuned for performance

**Tuning GC**:
```javascript
// In JavaScript:
Cu.forceCC();      // Force cycle collection
Cu.forceGC();      // Force JS GC
```

## Low-Level System Access

### Platform Abstraction Layer (HAL)

Located in `hal/`:
- Battery API
- Screen brightness
- Network information
- Vibration
- Sensors (accelerometer, etc.)

```cpp
// Example: Getting battery status
#include "mozilla/HalTypes.h"
#include "mozilla/hal.h"

void HandleBatteryChange(const mozilla::hal::BatteryInformation& aInfo) {
    printf("Battery level: %f%%\n", aInfo.level() * 100.0);
}

mozilla::hal::RegisterBatteryObserver(this);
```

### Widget System (OS Integration)

Located in `widget/`:
- Window creation
- Event handling
- Clipboard access
- File dialogs
- Native drawing

```cpp
// Platform-specific implementations:
// widget/windows/ - Windows-specific
// widget/gtk/    - Linux
// widget/cocoa/  - macOS
```

### XPCOM (Component Object Model)

Old but still used component system:

```cpp
// Define interface (MyInterface.idl)
[scriptable, uuid(12345678-1234-1234-1234-123456789012)]
interface nsIMyInterface : nsISupports {
    readonly attribute string name;
    void doSomething();
};

// Implement
class MyComponent : public nsIMyInterface {
public:
    NS_DECL_ISUPPORTS
    NS_DECL_NSIMYINTERFACE
    
private:
    ~MyComponent();
};

NS_IMPL_ISUPPORTS(MyComponent, nsIMyInterface)
```

## Networking Systems

### HTTP Layer Architecture

```
HTTP/3 ← HTTP/2 ← HTTP/1.1 ← DNS
    ↓
nsHttpChannel
    ↓
nsIOService (connection pooling)
    ↓
Socket layer (TLS, TCP/UDP)
```

**Key Files**:
- `netwerk/protocol/http/nsHttpChannel.cpp` - Main HTTP handling
- `netwerk/base/nsIOService.cpp` - Core IO service
- `netwerk/socket/` - Socket implementations
- `security/manager/ssl/` - TLS/SSL handling

### Adding Custom Protocol Handler

```cpp
// Create handler for myprotocol://
#include "nsIProtocolHandler.h"

class MyProtocolHandler : public nsIProtocolHandler {
public:
    NS_DECL_ISUPPORTS
    NS_DECL_NSIPROTOCOLHANDLER
};

// Register in component system
```

### DNS and Connection Management

```cpp
// DNS: netwerk/dns/nsIDNSService.idl
// Connection pooling: netwerk/base/nsConnectionMgr.cpp

// Access DNS service:
nsCOMPtr<nsIDNSService> dns = do_GetService("@mozilla.org/network/dns-service;1");
dns->asyncResolve(name, flags, listener, aEventTarget, result);
```

## JavaScript Engine Internals

### SpiderMonkey Architecture

```
JavaScript Source Code
    ↓
Lexer (tokenization)
    ↓
Parser (AST generation)
    ↓
Bytecode Compiler
    ↓
JIT Compiler (Baseline → IonMonkey)
    ↓
Native Machine Code
```

**Key Directories**:
- `js/src/vm/` - Core VM implementation
- `js/src/parser/` - Parser
- `js/src/jit/` - JIT compiler
- `js/src/gc/` - Garbage collector
- `js/src/builtin/` - Built-in functions

### Adding Built-in Functions

```cpp
// js/src/builtin/MyBuiltins.cpp
static bool MyFunction(JSContext* cx, unsigned argc, JS::Value* vp) {
    JS::CallArgs args = JS::CallArgsFromVp(argc, vp);
    
    if (args.length() < 1) {
        JS_ReportErrorNumberASCII(cx, GetErrorMessage, nullptr, JSMSG_MISSING_FUN_ARG, "param");
        return false;
    }
    
    args.rval().setString(JS_NewStringCopyZ(cx, "result"));
    return true;
}

// Register function
if (!JS_DefineFunction(cx, global, "myFunction", MyFunction, 1, JSFUN_FAST_NATIVE)) {
    return false;
}
```

## Rendering Engine Details

### Layout System

```
HTML Document
    ↓
Content Tree (nsIContent hierarchy)
    ↓
Frame Tree Construction (PresShell)
    ↓
Box Model Calculation
    ↓
Line and Inline Layout
    ↓
Block Layout (normal flow, floats, positioning)
    ↓
Painting (render to graphics backend)
```

**Key Files**:
- `layout/generic/nsBlockFrame.cpp` - Block layout
- `layout/generic/nsInlineFrame.cpp` - Inline layout
- `layout/generic/nsFlexContainerFrame.cpp` - Flexbox
- `layout/generic/nsGridContainerFrame.cpp` - CSS Grid

### CSS Implementation

```cpp
// CSS parsing and cascade
// dom/base/Element.cpp - Element styling
// layout/style/StyleComputation.cpp - Cascade and resolution

// Access computed styles:
nsCOMPtr<nsIDOMCSSStyleDeclaration> styles;
element->GetComputedStyle(styles);
```

### Graphics Backend

```
Web Content
    ↓
Painting/Recording (DisplayList)
    ↓
WebRender or D2D/Skia
    ↓
GPU acceleration (if available)
    ↓
Screen
```

**Tuning graphics**:
```javascript
// In about:config
gfx.webrender.enabled = true;  // Enable WebRender
gfx.webrender.compositor = auto;
layers.acceleration.force-enabled = true;
```

## Security Model

### Sandboxing

Each content process runs in a sandbox:

```
Main Process (Full privileges)
    ↓ IPC
Content Process (Restricted permissions)
    - No file system access (except approved paths)
    - No hardware access
    - No network access (goes through main process)
    - Can't execute arbitrary code
```

**Sandbox Policy**:
- `security/sandbox/` - Sandbox implementation
- `dom/ipc/` - IPC security checks

### Security Policies

```cpp
// Content Security Policy (CSP)
// security/csp/ - CSP implementation

// Same-Origin Policy
// caps/nsScriptSecurityManager.cpp

// Permissions system
// dom/base/PermissionStatus.cpp
```

## Performance Optimization Techniques

### Profiling

```bash
# CPU profiling
./mach run --args="--profiler"
# Then: Shift+Ctrl+5 to open profiler

# Memory profiling
./mach run --args="-dmdump"

# Frame rate profiling
# In browser console:
// Enable:
Enable in about:config: layers.acceleration.force-enabled
prof = Cu.import("resource://gre/modules/Profiler.jsm");
prof.profiler.startProfiler();
// ... do work ...
prof.profiler.getRawProfile();
```

### Common Optimizations

1. **Reduce allocations**:
   ```cpp
   // Use AutoTArray instead of Vector for small sizes
   AutoTArray<MyType, 16> array;  // Stack allocated
   ```

2. **Lazy initialization**:
   ```cpp
   // Initialize only when needed
   mLazyValue = nullptr;  // Checked at access
   ```

3. **Caching**:
   ```cpp
   // Cache computed results
   mCachedValue = ComputeExpensive();
   mIsCached = true;
   ```

4. **Batch operations**:
   ```cpp
   // Group DOM mutations under single batch
   dom->BeginUpdate(UPDATE_CONTENT_MODEL);
   // ... multiple modifications ...
   dom->EndUpdate(UPDATE_CONTENT_MODEL);
   ```

### Performance Regression Testing

```bash
# Performance tests
./mach perf

# Benchmark startup
./mach perf startup

# Benchmark power consumption
./mach perf power

# Custom perf tests
./mach test perf/tests/your_test.js
```

## Debugging Techniques

### Using GDB/LLDB

```bash
./mach debug

# In debugger:
(gdb) break nsHttpChannel::OnStartRequest
(gdb) continue
(gdb) frame 0
(gdb) print mURL  # Print member variable
(gdb) print *this # Print entire object
(gdb) info locals # List local variables
(gdb) x/100wx $sp # Examine memory
(gdb) disassemble  # Disassemble current function
```

### Logging

```cpp
// Add logging to your code:
#include "mozilla/Logging.h"

static mozilla::LazyLogModule gMyLog("MyModule");

MOZ_LOG(gMyLog, mozilla::LogLevel::Info, ("Message: %s", var.get()));
MOZ_LOG(gMyLog, mozilla::LogLevel::Error, ("Error!"));
```

### Run with Logging

```bash
MOZ_LOG=MyModule:5 ./mach run
# Level 5 = Debug (0=disabled, 5=debug, 4=warning, 3=error, etc.)
```

## Memory Issues Debugging

### Finding Leaks

```bash
# DMD (Dark Matter Detector)
./mach run --args="-dmdump"

# Valgrind (Linux only)
valgrind --leak-check=full ./firefox
```

### AddressSanitizer

```bash
# Compile with ASan
ac_add_options --enable-address-sanitizer

./mach build
./mach run  # Will report violations
```

## Binary Modification Techniques

### Patching at Runtime

```cpp
// Use LD_PRELOAD or DLL injection
// More commonly: build with modifications

// Or use monkeypatch at JS level:
ChromeUtils.defineModuleGetter(this, "MyModule", "resource://my/module.jsm");
```

### Creating a Statically Linked Build

```bash
# In .mozconfig
ac_add_options --enable-static-build
ac_add_options --disable-shared-js
```

## Module Development

### Creating a New Component

1. **Define IDL interface** (`MyComponent.idl`):
   ```idl
   [scriptable, uuid(00000000-0000-0000-0000-000000000000)]
   interface nsIMyComponent : nsISupports {
       readonly attribute AString name;
       void doWork();
   };
   ```

2. **Implement** (`MyComponent.cpp`):
   ```cpp
   NS_IMPL_ISUPPORTS(MyComponent, nsIMyComponent)
   // Implement methods
   ```

3. **Register in component system** (build manifest)

4. **Use from JavaScript**:
   ```javascript
   const MyComponent = Cc["@mozilla.org/my-component;1"]
       .getService(Ci.nsIMyComponent);
   ```

## Testing Your Modifications

### Unit Testing (C++)

```cpp
// In *_unittest.cpp
#include "gtest/gtest.h"

TEST(MyComponentTest, BasicFunctionality) {
    MyComponent comp;
    ASSERT_EQ(comp.getValue(), expected);
}

// Run:
./mach test gtest::MyComponentTest
```

### Integration Testing

```html
<!-- test_mycomponent.html -->
<script>
  add_task(async function test_mycomponent() {
    let comp = Cc["@mozilla.org/my-component;1"]
        .getService(Ci.nsIMyComponent);
    Assert.equal(comp.name, "test");
  });
</script>

<!-- Run: ./mach test mochitest -->
```

## Common Pitfalls

1. **Thread safety**: Always protect shared data
2. **Circular references**: Use weak references
3. **Memory leaks**: Profile regularly
4. **Blocking main thread**: Use async/await
5. **IPC crossing the process boundary**: Manage carefully
6. **Removing deprecated code**: Check for usages
7. **Configuration changes**: Test all platforms
8. **Security assumptions**: Review permissions

## Tools and Utilities

### Code Analysis

```bash
# Find symbol definitions
grep -r "class MySymbol" .

# Find usages
grep -r "MySymbol" . | grep -v Binary

# Static analysis
./mach lint

# Code coverage
./mach test --coverage
```

### Build System

```bash
# Show what's being built
./mach build --verbose

# Rebuild specific target
./mach build --force dom/base

# Show build dependencies
./mach build --help  # See more options
```

### Instrumentation

```cpp
// Use RAII for profiling
PROFILER_RAII(name);  // Measures scope duration automatically
```

## References

- **Firefox Architecture**: https://firefox-source-docs.mozilla.org/contributing/architecture/
- **XPCOM Guide**: https://firefox-source-docs.mozilla.org/xpcom/
- **SpiderMonkey**: https://spidermonkey.dev/
- **Networking**: https://firefox-source-docs.mozilla.org/networking/
- **Layout Engine**: https://firefox-source-docs.mozilla.org/contributing/architecture/layout.html

---

**Last Updated: March 2026**
