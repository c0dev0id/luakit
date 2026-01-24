# GitHub Copilot Instructions for Luakit Development

This document provides instructions for AI assistants working on the Luakit codebase.

## Workflow Preferences

### Patch File Generation (REQUIRED)

**ALWAYS create downloadable patch files instead of opening pull requests.**

When making changes:
1. Generate a unified diff patch file with `.patch` extension
2. Present the patch in a downloadable code block
3. Provide wget command to download the patch to `/home/sdk/Downloads`
4. Provide ONLY the git apply commands - assume patch file is already downloaded
5. Use complete absolute paths in all commands
6. Use the EXACT filename shown in the code block (including any Version suffixes from download)

**User Environment:**
- Download directory: `/home/sdk/Downloads` (patches automatically saved here)
- Repository location: `/home/sdk/luakit`
- Patch application method: `git apply`
- Command format: No explanations, no patch creation commands, complete paths only
- Filename format: Browser may append `_Version2`, `_Version3`, etc. to duplicate downloads

## Public API Protection

### Critical Rule: API Stability

**The public Lua API surface MUST NOT be violated.** This is a fundamental requirement for maintaining compatibility with user configurations.

Any removal, modification, or behavioral change to the public API **MUST be**:
1. **Explicitly confirmed** with the maintainer before implementation
2. **Documented** in `MIGRATION_NEXT.md` with complete migration instructions
3. **Tested** to ensure existing user configurations continue to work where possible

### What Constitutes the Public API?

The public API consists of all interfaces exposed to users through their configuration files (`rc.lua`, `userconf.lua`, `theme.lua`). This includes:

#### 1. C-Backed Core APIs (Built-in)
Located in `clib/*.c` (UI process) and `extension/clib/*.c` (web process):
- `luakit.*` - Core engine functions and properties
- `widget` - GTK widget system and all widget types
- `soup.*` - Network/HTTP layer
- `msg.*` - Logging system
- `timer` - Timer API
- `regex` - Regular expression API
- `xdg.*` - XDG directory paths
- `stylesheet` - CSS stylesheet objects
- `download` - Download management

#### 2. Lua Library Modules (lib/)
High-level modules wrapping C APIs or providing pure Lua functionality:
- `window` - Window management
- `webview` - Webview wrapper module
- `modes` - Input mode system
- `binds` - Key binding system
- `settings` - Settings management
- `chrome` - Chrome page framework
- `lousy.*` - Utility libraries (theme, widgets, signals, utilities)

#### 3. Optional Feature Modules (lib/)
User-loadable modules:
- `adblock`, `downloads`, `bookmarks`, `history` (with `_chrome` variants)
- `formfiller`, `noscript`, `proxy`, `follow`, `select`
- `session`, `quickmarks`, `undoclose`, `tabhistory`
- `userscripts`, `editor`, `webinspector`, `styles`, `error_page`
- And many more...

#### 4. Web Process APIs (Extension)
DOM and page interaction APIs:
- `page.*` - Web page API
- `dom_document.*` - DOM document API
- `dom_element.*` - DOM element API
- `require_web_module()` / `ipc_channel()` - IPC communication

#### 5. Signal System
All documented signals on any API object

#### 6. Settings Registry
All settings registered via `settings.register_settings()`

#### 7. Theme Properties
All documented theme properties in `lousy.theme`

### API Change Documentation Requirements

When a public API change is necessary, document it in `MIGRATION_NEXT.md` with:
- What changed (old syntax to new syntax)
- One-sentence reason for the change
- Migration instructions with code examples for rc.lua/userconf.lua/theme.lua

## Development Guidelines

### 1. Before Making API Changes
- Always check if the change affects the public API
- Always get explicit confirmation from maintainers
- Always consider backward compatibility options
- Never assume an API is "internal" without verification

### 2. Adding New APIs
- Document in appropriate `doc/luadoc/*.lua` file
- Include usage examples
- Follow existing naming conventions
- Add tests where possible
- Consider forward compatibility

### 3. Deprecating APIs
- Keep old API working with deprecation warning
- Document in `MIGRATION_NEXT.md`
- Provide clear migration path
- Use `msg.warn()` to inform users at runtime
- Plan for removal in future major version

### 4. Testing API Changes
- Test with actual user configuration patterns
- Verify documentation examples still work
- Check common use cases from `config/rc.lua`
- Run full test suite: `make run-tests`

### 5. Documentation Updates
API changes require updates in:
- `doc/luadoc/*.lua` files (for C APIs)
- Module source files (luadoc-style comments)
- `MIGRATION_NEXT.md` (for breaking changes)
- `CHANGELOG.md` (for all changes)

## Output Format for Changes

When proposing changes, ALWAYS use this format:

1. Show the patch in a downloadable code block
2. Provide wget command to download (when applicable)
3. Provide git commands with complete paths

**Do NOT:**
- Create pull requests
- Include cat or patch creation commands
- Use relative paths
- Add explanatory text between command blocks
- Guess at version suffixes

## Summary

**Remember:** Luakit is a user-configurable browser. Users rely on the API remaining stable across versions. Treat API stability as a top priority, and always document changes clearly to help users migrate their configurations.

**When in doubt:**
- Ask before changing
- Document thoroughly
- Test with real user configurations
- Provide migration examples
- Do not break users without warning
