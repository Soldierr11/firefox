# Firefox Architecture Visual Reference

A visual guide to Firefox's architecture and component relationships.

## System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          FIREFOX BROWSER                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────────┐   │
│  │                   MAIN PROCESS                                   │   │
│  │  (Browser UI, Tab Management, Network Coordination)             │   │
│  │                                                                  │   │
│  │  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐       │   │
│  │  │   Browser    │    │  Preference  │    │  Extensions  │       │   │
│  │  │   Components │    │   System     │    │  Manager     │       │   │
│  │  └──────────────┘    └──────────────┘    └──────────────┘       │   │
│  │                                                                  │   │
│  │  ┌──────────────────────────────────────────────────────┐       │   │
│  │  │         IPC Actor System (Messaging)                │       │   │
│  │  └──────────────────────────────────────────────────────┘       │   │
│  └──────────────────────────────────────────────────────────────────┘   │
│                                 │                                       │
│        ┌────────────────────────┼────────────────────────┐             │
│        │                        │                        │             │
│  ┌─────▼──────┐      ┌──────────▼────────┐    ┌─────────▼─────┐      │
│  │  CONTENT   │      │      GPU          │    │    NETWORK    │      │
│  │ PROCESS(S) │      │     PROCESS       │    │   PROCESS     │      │
│  │            │      │                   │    │               │      │
│  │ ┌────────┐ │      │ ┌──────────────┐  │    │ ┌───────────┐ │      │
│  │ │  DOM   │ │      │ │  WebRender   │  │    │ │   HTTP    │ │      │
│  │ │ Layers │ │      │ │  Compositor  │  │    │ │  Stack    │ │      │
│  │ └────────┘ │      │ │  Graphics    │  │    │ │   DNS     │ │      │
│  │            │      │ └──────────────┘  │    │ │ Caching   │ │      │
│  │ ┌────────┐ │      │                   │    │ └───────────┘ │      │
│  │ │ Parser │ │      └─────────────────────    │               │      │
│  │ │ Layout │ │                               │               │      │
│  │ └────────┘ │                               └───────────────┘      │
│  │            │                                    ↓                  │
│  │ ┌────────┐ │                            ┌─────────────────┐       │
│  │ │  Style │ │                            │  Socket Layer   │       │
│  │ │  JS VM │ │                            │  TLS/SSL        │       │
│  │ └────────┘ │                            │  TCP/UDP        │       │
│  │            │                            └─────────────────┘       │
│  └────────────┘                                                      │
│                                                                      │
│  ┌────────────────────────────────────────────────────────────┐    │
│  │            SHARED LIBRARIES & COMPONENTS                   │    │
│  │  ┌──────────┬──────────┬──────────┬──────────┬──────────┐  │    │
│  │  │ Security │ Storage  │ Intl     │  XP COM  │ Platform │  │    │
│  │  │ Manager  │ (Cache,  │(i18n)    │  (Old)   │Abstract. │  │    │
│  │  │(Sandbox) │IndexedDB)│          │  System  │(HAL,    │  │    │
│  │  │ CSP, SOP │ Cookies  │ Text     │          │Widget)  │  │    │
│  │  └──────────┴──────────┴──────────┴──────────┴──────────┘  │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
                                  │
                    ┌─────────────┴──────────────┐
                    │                            │
           ┌────────▼────────┐        ┌──────────▼────────┐
           │   Operating     │        │   Graphics Card   │
           │   System        │        │   (GPU)           │
           │ (Linux/Win/Mac) │        │   Hardware        │
           └─────────────────┘        └───────────────────┘
```

## Component Interaction Flow

### Loading a Web Page
```
User enters URL
    ↓
[browser/components/urlbar/] - URLbar validation
    ↓
[browser/base/BrowserTab.js] - Create new tab
    ↓
[docshell/] - Document loader
    ↓
[netwerk/] - Network request
    ↓
[security/manager/] - TLS, certificate validation
    ↓
HTTP data received
    ↓
[parser/] - HTML parser
    ↓
[dom/] - DOM tree construction
    ↓
[layout/] - CSS parsing, layout calculation
    ↓
[gfx/] - Rendering, rasterization
    ↓
[js/src/] - JavaScript execution
    ↓
Rendered page displayed
```

## Memory Hierarchy

```
                    ┌─────────────────┐
                    │   Disk Cache    │ (100MB+)
                    │   (netwerk/)    │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  RAM (Physical) │ (800MB-3GB typical)
                    │                 │
                    │  ┌───────────┐  │
                    │  │   Heap    │  │ (Allocated memory)
                    │  │           │  │ - DOM trees
                    │  │  Layout   │  │ - Images
                    │  │  Objects  │  │ - Caches
                    │  └─────┬─────┘  │
                    │        │        │
                    │  ┌─────▼─────┐  │
                    │  │   Stack   │  │ (Local variables)
                    │  │   Space   │  │ (10-20MB)
                    │  └───────────┘  │
                    │                 │
                    └────────┬────────┘
                             │
                    ┌────────▼────────┐
                    │  CPU Cache L3   │ (16MB)
                    │  CPU Cache L2   │ (256KB per core)
                    │  CPU Cache L1   │ (32KB data)
                    └─────────────────┘
```

## Threading Model

```
Main Thread (UI/DOM)
├─ Input handling
├─ DOM manipulation
├─ JavaScript execution
├─ Style recalculation
├─ Layout calculation
└─ Paint operations

Layout Thread
├─ CSS selector matching
├─ Box model calculations
└─ Frame tree building

Network Thread (Network Process)
├─ DNS resolution
├─ HTTP protocol
├─ Socket I/O
└─ Cache operations

GPU Thread
├─ Texture management
├─ Shader compilation
└─ Rendering commands

JS GC Thread
├─ Mark phase
├─ Sweep phase
└─ Compaction

Worker Threads
├─ Service Workers
├─ Web Workers
└─ Async tasks
```

## Code Organization by Responsibility

```
browser/
├─ UI layer
├─ User interaction
├─ Tab management
└─ Extension integration

dom/
├─ DOM tree
├─ Event handling
├─ Web APIs
└─ JavaScript integration

layout/
├─ CSS parsing
├─ Box calculations
├─ Text layout
└─ Positioning

gfx/
├─ Graphics primitives
├─ Text rendering
├─ Image decoding
└─ Compositor

netwerk/
├─ HTTP protocol
├─ DNS resolution
├─ Caching
└─ Cookie handling

js/
├─ JavaScript engine
├─ JIT compilation
├─ Garbage collection
└─ Built-in objects

security/
├─ TLS/SSL
├─ Certificate validation
├─ Encryption
└─ Permissions

toolkit/
├─ Shared utilities
├─ Preferences
├─ Storage
└─ Platform abstraction
```

## Data Flow: User Input → Rendered Pixels

```
┌─────────────────────────────────────────────────────────────┐
│                    USER INPUT                               │
│  (Mouse, Keyboard, Touch, Scroll)                          │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
         ┌──────────────────────┐
         │   Platform Widget    │ (widget/) - Native window
         │   Event System       │ receives OS events
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │   Event Dispatcher   │ (dom/) routes events
         │   Target Selection   │
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │   JavaScript Event   │ Execute event handlers
         │   Handlers (JS, C++) │ May modify DOM
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │  DOM Modification    │ Change document tree
         │  (if applicable)     │
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │  Style Calculation   │(layout/) Cascade, compute styles
         │  (CSSOM + Cascade)   │
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │  Layout (Reflow)     │ (layout/) Calculate positions
         │  - Block layout      │ and sizes
         │  - Inline layout     │
         │  - Flexbox/Grid      │
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │  Paint (Rendering)   │ (gfx/) Draw pixels
         │  - Display list      │ to buffer
         │  - Rasterization     │
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │  Composite           │ (gfx/) Combine layers
         │  (GPU acceleration)  │ apply effects
         └──────────┬───────────┘
                    │
                    ▼
         ┌──────────────────────┐
         │  Screen Refresh      │ Vsync → monitor update
         │  (Display output)    │
         └──────────────────────┘
```

## File Format Support

```
Document Formats          Handled By
├─ HTML                  parser/ + dom/
├─ XML                   parser/ + dom/
├─ XHTML                 parser/ + dom/
├─ SVG                   dom/ + layout/ + gfx/
└─ PDF                   PDF.js (JavaScript)

Content Formats          Handled By
├─ CSS                   layout/
├─ JavaScript            js/
├─ Images (PNG, JPEG)    image/ + gfx/
├─ WebP                  image/ + gfx/
├─ Video (MP4, WebM)     media/
└─ Audio (MP3, Opus)     media/

Network Protocols        Handled By
├─ HTTP/1.1              netwerk/protocol/http/
├─ HTTP/2                netwerk/protocol/http/
├─ HTTP/3 (QUIC)         netwerk/protocol/http/
├─ HTTPS (TLS)           security/manager/ + netwerk/
├─ WebSocket             dom/websocket/
├─ FTP (deprecated)      netwerk/protocol/ftp/
└─ Data URLs             netwerk/protocol/data/
```

## Module Dependencies (Simplified)

```
browser/          → Depends on: toolkit/, netwerk/, dom/, extensions/
dom/              → Depends on: js/, layout/, netwerk/, storage/
layout/           → Depends on: dom/, gfx/, parser/
gfx/              → Depends on: graphics libraries (Skia, WebRender)
netwerk/          → Depends on: security/, storage/
security/         → Depends on: NSS crypto library
js/               → Mostly standalone (SpiderMonkey)
toolkit/          → Depends on: xpcom/, storage/
storage/          → Depends on: sqlite, IndexedDB impl
media/            → Depends on: ffmpeg, codec libraries
extensions/       → Depends on: dom/, browser/
```

## Common Code Patterns

### XPCOM Interface Declaration
```
.idl file (interface)
    ↓
.h file (C++ header)
    ↓
.cpp file (implementation)
    ↓
Service registration
    ↓
JavaScript access via Services.someService
```

### JavaScript Event Flow
```
addEventListener("click")
    ↓
Event fires
    ↓
DOM event dispatcher
    ↓
Bubble through DOM tree
    ↓
Call handlers
    ↓
Update display if needed
```

### Network Request Lifecycle
```
Create nsIChannel
    ↓
Set request headers
    ↓
Open channel
    ↓
Receive data chunks → Listeners
    ↓
Process response
    ↓
Call completion listener
    ↓
Cache result (if appropriate)
```

## Performance Critical Paths

```
Time-critical operations:
1. Event handling        < 100ms (user feels responsive)
2. Frame rendering      ≤ 16ms (60 FPS)
3. Page load           < 500ms (perception)
4. Startup             < 100ms (user notice)

Memory-critical:
1. DOM nodes           ~500 bytes each
2. Images            2-5MB for typical page
3. Caches            100-500MB typical
4. JS heaps          20-100MB per page
```

## Debugging Hotspots

```
Slow startup?              → browser/, profile loading
Slow page load?            → netwerk/, parser/, dom/
Slow rendering?            → layout/, gfx/
Slow JavaScript?           → js/jit/, dom/ interop
Memory leaks?              → js/ GC, dom/ cycles
Crashes?                   → Platform code (widget/), IPC issues
Extension conflicts?       → extensions/, browser/
Network issues?            → netwerk/, security/
Graphics glitches?         → gfx/, layout/
```

---

## Legend and Symbols

| Symbol | Meaning |
|--------|---------|
| `→` | Data flow direction |
| `├─` | Contains/sub-component |
| `└─` | Last sub-component |
| `[xxx/]` | Directory path |
| Process box | Separate OS process |
| Thread box | Thread within process |

---

This visual guide complements the detailed guides. Use it to visualize how components interact!
