# Firefox Codebase Learning Path

A roadmap for new systems programmers forking Firefox.

## What You've Got

I've created **3 comprehensive guides** to get you started:

### 1. **CODEBASE_GUIDE.md** (Start Here!)
The complete reference guide covering:
- Firefox architecture (layers, process model)
- Directory structure and purpose of each component
- Core components explained
- Technologies used (C++, Rust, JavaScript)
- Build system details
- Development workflow
- Common tasks and solutions
- Performance and security considerations

**Read this first:** 15-30 minutes for overview, reference for deep dives

### 2. **FORK_GUIDE.md** (Quick Reference)
The pragmatic developer's guide:
- Quick start commands (clone, build, run)
- Directory quick map
- Common development commands
- How to modify core components
- Configuration files
- Maintaining your fork against upstream
- Debugging common issues
- Testing strategies
- Release checklist

**Use this:** Daily reference while developing

### 3. **SYSTEMS_PROGRAMMING_GUIDE.md** (Advanced)
Technical deep dive for systems programmers:
- Multi-process architecture details
- Memory layout and profiling
- Threading model
- Garbage collection tuning
- Platform abstraction (HAL)
- Networking internals
- JavaScript engine (SpiderMonkey) architecture
- Rendering engine details
- Security sandboxing
- Performance optimization techniques
- Debugging with GDB/LLDB
- Creating components and modules

**Use this:** When modifying core systems

---

## Learning Path

### Phase 1: Understanding (Days 1-3)
1. Read [README.md](README.md) - Official project overview
2. Skim [CODEBASE_GUIDE.md](CODEBASE_GUIDE.md) - Architecture sections
3. Set up locally: `./mach bootstrap && ./mach build`
4. Run Firefox: `./mach run`
5. Explore directories: Walk through browser/, dom/, netwerk/

**Goal**: Understand how Firefox is organized and builds

### Phase 2: Building Skills (Weeks 1-2)
1. Read [FORK_GUIDE.md](FORK_GUIDE.md) carefully
2. Make small changes:
   - Add a preference in `browser/app/profile/prefs.js`
   - Modify a string in browser UI (`browser/base/content/`)
   - Change default homepage
3. Rebuild: `./mach build browser`
4. Test changes: `./mach run`
5. Review code in your area of interest

**Goal**: Become comfortable building and testing

### Phase 3: Deeper Understanding (Weeks 2-4)
1. Focus on one component:
   - **If interested in UI**: Study browser/, toolkit/
   - **If interested in networking**: Study netwerk/
   - **If interested in rendering**: Study layout/, gfx/
   - **If interested in JavaScript**: Study js/
   - **If interested in security**: Study security/, caps/

2. Read relevant sections from [SYSTEMS_PROGRAMMING_GUIDE.md](SYSTEMS_PROGRAMMING_GUIDE.md)
3. Make targeted changes to your component
4. Run appropriate tests: `./mach test [componentname]`
5. Read existing code comments and patterns

**Goal**: Specialize in one area, understand patterns

### Phase 4: Advanced Modifications (Weeks 4+)
1. Plan your custom features/changes
2. Design architecture around existing systems
3. Modify multiple components if needed
4. Write comprehensive tests
5. Profile for performance
6. Document your changes

**Goal**: Execute meaningful features in your fork

---

## What to Read Next

### Immediate Focus (This week)
- [ ] Read CODEBASE_GUIDE.md (Quick Overview section)
- [ ] Read FORK_GUIDE.md (Quick Start section)
- [ ] Run `./mach bootstrap`
- [ ] Build Firefox: `./mach build`
- [ ] Run Firefox: `./mach run`

### Next Week
- [ ] Read full CODEBASE_GUIDE.md
- [ ] Choose a component area (browser/, netwerk/, etc.)
- [ ] Make small UI change in browser/
- [ ] Read architecture docs for your component
- [ ] Review existing code in that directory

### After 2 Weeks
- [ ] Read SYSTEMS_PROGRAMMING_GUIDE.md sections relevant to your changes
- [ ] Make more significant modification
- [ ] Write tests for your changes
- [ ] Setup your fork on GitHub
- [ ] Plan your feature/changes list

---

## Component-Specific Learning Paths

### For Browser UI Hackers
```
Start: browser/, toolkit/
Read: CODEBASE_GUIDE.md → Core Components → Browser UI
Study: browser/base/content/, browser/components/
Commands: ./mach build browser
Tests: ./mach test mochitest
```

### For Network Systems Programmers
```
Start: netwerk/
Read: CODEBASE_GUIDE.md → Core Components → Network Stack
Advanced: SYSTEMS_PROGRAMMING_GUIDE.md → Networking Systems
Study: netwerk/protocol/http/, netwerk/socket/
Commands: ./mach build netwerk
Tests: ./mach test netwerk
```

### For Graphics/Rendering Engineers
```
Start: layout/, gfx/
Read: CODEBASE_GUIDE.md → Core Components → Rendering Engine
Advanced: SYSTEMS_PROGRAMMING_GUIDE.md → Rendering Engine Details
Study: layout/generic/, gfx/
Commands: ./mach build layout gfx
Tests: ./mach test layout
```

### For JavaScript Engine Hackers
```
Start: js/
Read: CODEBASE_GUIDE.md → Core Components → JavaScript Engine
Advanced: SYSTEMS_PROGRAMMING_GUIDE.md → JavaScript Engine Internals
Study: js/src/vm/, js/src/jit/, js/src/builtin/
Commands: ./mach build js
Tests: ./mach test jit_test
```

### For Security Specialists
```
Start: security/, caps/
Read: CODEBASE_GUIDE.md → Security Considerations
Advanced: SYSTEMS_PROGRAMMING_GUIDE.md → Security Model
Study: security/manager/ssl/, caps/, dom/ipc/
Commands: ./mach build security
Tests: ./mach test security
```

---

## Key Commands Cheat Sheet

```bash
# Build
./mach bootstrap           # Install dependencies (once)
./mach build              # Full build
./mach build dom/         # Specific component
./mach clobber            # Clean build

# Run & Debug
./mach run                # Run Firefox
./mach run --args="-console"  # With console
./mach debug              # Debug with GDB/LLDB

# Test
./mach test mochitest     # Web tests
./mach test gtest         # C++ tests
./mach test dom/          # Component tests

# Code Quality
./mach lint               # Find issues
./mach lint --fix         # Auto-fix
./mach format             # Format code

# Profiling
./mach perf               # Performance tests
./mach run --args="--profiler"  # Profile app
```

---

## Your Fork Plan Template

Create a file in your fork documenting:

```markdown
# My Firefox Fork

## Goals
- [ ] Goal 1 (e.g., custom privacy features)
- [ ] Goal 2 (e.g., performance optimizations)
- [ ] Goal 3 (e.g., security enhancements)

## Modified Components
- [ ] browser/ - UI changes
- [ ] netwerk/ - Network modifications
- [ ] layout/ - Rendering changes
- [ ] [Other components]

## Key Files Modified
- [ ] List of files you've changed
- [ ] Percentage complete for each

## Testing Status
- [ ] Unit tests passing
- [ ] Integration tests passing
- [ ] Manual testing done
- [ ] Performance baseline established

## Build Instructions
```bash
./mach bootstrap
./mach build
./mach run
```

## Known Issues
- Issue 1: [Description]
- Issue 2: [Description]

## Timeline/Roadmap
- Week 1: [Milestones]
- Week 2: [Milestones]
```

---

## Resources You Need

### Official Documentation
- [Firefox Source Docs](https://firefox-source-docs.mozilla.org/) - Complete reference
- [Contributing Guide](https://firefox-source-docs.mozilla.org/contributing/contribution_quickref.html)
- [Architecture](https://firefox-source-docs.mozilla.org/contributing/architecture/)

### Your New Documentation
- [CODEBASE_GUIDE.md](CODEBASE_GUIDE.md) - This repository
- [FORK_GUIDE.md](FORK_GUIDE.md) - This repository
- [SYSTEMS_PROGRAMMING_GUIDE.md](SYSTEMS_PROGRAMMING_GUIDE.md) - This repository

### External Resources
- [SpiderMonkey Documentation](https://spidermonkey.dev/)
- [Mozilla Developer Network (MDN)](https://developer.mozilla.org/)
- [Bugzilla](https://bugzilla.mozilla.org/) - Issue tracker

### Community
- [Mozilla Matrix Chat](https://chat.mozilla.org/#/room/#introduction:mozilla.org)
- GitHub Issues (for your fork)
- Reddit: r/firefox (for user community discussions)

---

## Tips for Success

### As a New Systems Programmer

1. **Start small**: Make tiny changes first (colors, strings, defaults)
2. **Test frequently**: Build and run after each change
3. **Read code**: Understand patterns before modifying
4. **Ask questions**: No shame in asking on Matrix chat
5. **Use version control**: Commit frequently with good messages
6. **Profile before optimizing**: Don't guess about performance
7. **Document your changes**: Future you will thank you
8. **Maintain tests**: Don't break existing tests
9. **Keep security in mind**: Don't introduce vulnerabilities
10. **Stay in sync**: Regularly merge upstream updates

### When You Get Stuck

1. **Check existing tests**: See how components are tested
2. **Search code**: Use grep, linters, or IDE
3. **Read documentation**: Check Firefox Source Docs
4. **Look at Git history**: See how related code evolved
5. **Run with logging**: Enable logging for your component
6. **Use debugger**: Step through code execution
7. **Ask on Matrix**: Real Firefox developers are there to help

### Managing Your Fork

1. **Choose a branch naming strategy**: `dusk-staging`, `dusk-release`
2. **Keep upstream remote**: `git remote add upstream ...`
3. **Regular syncing**: `git fetch upstream && git rebase`
4. **Document deviations**: What's different from Mozilla?
5. **Plan releases**: Regular, scheduled releases
6. **Maintain compatibility**: Support for extensions

---

## Success Metrics

### Week 1
- Can build Firefox successfully
- Understand directory structure
- Made 1 small change, tested it

### Week 2-3
- Modified a component
- Written tests for changes
- Understand build system deeply

### Week 4+
- Implemented meaningful features
- Profiled and optimized
- Fork builds and runs reliably
- Have a release plan

---

## What's Next After Reading?

1. **Immediate** (Today): `./mach bootstrap && ./mach build`
2. **This Week**: Read CODEBASE_GUIDE.md, make first change
3. **This Month**: Complete Phase 1-2 of learning path
4. **This Quarter**: Have working fork with custom features

---

## Questions?

Refer to the specific guide:
- **"How is Firefox organized?"** → CODEBASE_GUIDE.md
- **"How do I build and test?"** → FORK_GUIDE.md
- **"How do I modify X?"** → Directory section in CODEBASE_GUIDE.md or SYSTEMS_PROGRAMMING_GUIDE.md
- **"What's the architecture?"** → SYSTEMS_PROGRAMMING_GUIDE.md
- **"I need a command"** → FORK_GUIDE.md (Common Commands section)

---

**You've got this! Happy hacking, and welcome to systems programming! 🚀**

*Last Updated: March 2026*
