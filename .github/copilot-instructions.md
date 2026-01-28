## Core Principles

**KISS - Keep It Simple, Stupid**
- Make small, focused changes
- Each change should do one thing well
- High quality over quantity
- Always handle errors properly

**Compatibility Requirements**
- Scripts and code must work across different platforms (Linux and BSDs)
- Don't assume GNU-specific tools or syntax
- Test compatibility when using shell commands (sed, awk, etc.)

**Lua Compatibility**
- Luakit works with both Lua 5.1 and LuaJIT
- Maintain compatibility with both implementations
- Don't use features exclusive to Lua 5.2+ or LuaJIT-only APIs

1. **USE EXISTING MECHANISMS WHENEVER POSSIBLE** - Never introduce new patterns, frameworks, or approaches
2. **MAINTAIN CONSISTENCY** - Follow established conventions exactly
3. **NO DEVIATION** - If similar code exists, copy its style precisely
4. **TEST EVERYTHING** - Add tests using existing test infrastructure
5. **DOCUMENT PROPERLY** - Use existing documentation patterns

## Commit Message Style

Follow the project's established commit message conventions:

**Subject Line:**
- Use area prefix when applicable: `area: description` (e.g., `workflows:`, `extension:`, `copilot-instructions:`, `lib:`, `config:`)
- Keep subject line to 50-72 characters
- Use imperative mood ("add" not "added" or "adds")
- No period at end of subject line
- Lowercase after the colon

**Body (when needed):**
- Separate subject from body with blank line
- Wrap body at 72 characters
- Explain what and why, not how
- Include technical details when relevant
- Reference issues with `Fixes:` or `Link:` tags
- Use proper formatting for multi-paragraph explanations

## Workflow Preferences

### Patch File Generation (REQUIRED)

**ALWAYS create downloadable patch files instead of opening pull requests.**

When making changes:
1. Generate a unified diff patch file with `.patch` extension
2. Present the patch in a downloadable code block
3. Show command: `rm -vf /home/sdk/Downloads/<patchname>*.patch"
4. Show command: `git apply /home/sdk/Downloads/<patchname>*.patch"

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
- Use `msg.warn()`, `msg.info()` and `msg.error()` to inform users at runtime
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


## C Coding Standards

### File Structure

```c
/*
 * path/to/file.c - Brief description
 *
 * Copyright © YEAR Author Name <email>
 *
 * This program is free software: you can redistribute it and/or modify
 * it under the terms of the GNU General Public License as published by
 * the Free Software Foundation, either version 3 of the License, or
 * (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this program.  If not, see <http://www.gnu.org/licenses/>.
 *
 */

#include "luah.h"
#include "widgets/common.h"
```

### Naming Conventions

- **Functions**: `prefix_module_action` pattern (e.g., `luaH_entry_insert`, `widget_entry_focus`)
- **Structs**: Use `_t` suffix (e.g., `widget_t`, `property_t`)
- **Constants**: Use `L_TK_` prefix for tokens (e.g., `L_TK_TEXT`, `L_TK_POSITION`)
- **Macros**: ALL_CAPS (e.g., `LUAKIT_WIDGET_INDEX_COMMON`)

### Indentation and Formatting

- **2 spaces** for indentation (no tabs)
- Opening braces on same line for functions
- Space after keywords (`if`, `for`, `while`)
- No space between function name and opening parenthesis

```c
static gint
luaH_entry_insert(lua_State *L)
{
    widget_t *w = luaH_checkwidget(L, 1);
    
    if (lua_gettop(L) > 2) {
        pos = luaL_checknumber(L, idx++);
    }
    
    return 0;
}
```

### Widget Pattern (GTK+ Bindings)

All widgets follow this structure:

1. **Property table** with token-based lookups:

```c
static property_t widget_properties[] = {
    { L_TK_MARGIN,     "margin",      INT,    TRUE  },
    { L_TK_TEXT,       "text",        STRING, TRUE  },
    { L_TK_SHOW_FRAME, "show_frame",  BOOL,   TRUE  },
    { 0, NULL, 0, 0 }, // Null terminator
};
```

2. **Index function** for property getters:

```c
static gint
luaH_entry_index(lua_State *L, widget_t *w, luakit_token_t token)
{
    switch(token) {
      LUAKIT_WIDGET_INDEX_COMMON(w)
      
      /* push class methods */
      PF_CASE(INSERT,        luaH_entry_insert)
      /* push integer properties */
      PI_CASE(POSITION,      gtk_editable_get_position(GTK_EDITABLE(w->widget)))
      /* push string properties */
      PS_CASE(TEXT,          gtk_entry_get_text(GTK_ENTRY(w->widget)))
      /* push boolean properties */
      PB_CASE(SHOW_FRAME,    gtk_entry_get_has_frame(GTK_ENTRY(w->widget)))
      
      default:
        break;
    }
    return 0;
}
```

3. **Newindex function** for property setters:

```c
static gint
luaH_entry_newindex(lua_State *L, widget_t *w, luakit_token_t token)
{
    switch(token) {
      LUAKIT_WIDGET_NEWINDEX_COMMON(w)
      
      case L_TK_TEXT:
        gtk_entry_set_text(GTK_ENTRY(w->widget),
            luaL_checklstring(L, 3, &len));
        break;
        
      default:
        return 0;
    }
    return 0;
}
```

4. **Class registration**:

```c
LUA_OBJECT_FUNCS(widget_class, widget_t, widget);
```

### Token System

**CRITICAL**: When adding new widget properties:

1. Add token to `common/tokenize.list` (alphabetically sorted)
2. Run `make` to regenerate `common/tokenize.h` and `common/tokenize.c`
3. Use `L_TK_*` constants in switch statements
4. Never use string comparisons for property lookup (performance)

**Example**: Adding a new property `placeholder`:
1. Add `placeholder` to `common/tokenize.list`
2. Rebuild with `make`
3. Use `L_TK_PLACEHOLDER` in code

---

## 🐚 Lua Code Style Standards

### Module Structure

```lua
--- Module description (one line summary)
--
-- Additional documentation providing details about the module.
-- Can span multiple lines.
--
-- @module module_name
-- @author Name <email>
-- @copyright Year Name <email>

local _M = {}

-- Require dependencies at top
local lousy = require("lousy")
local widget = require("widget")

-- Private data using weak tables (for garbage collection)
local priv = setmetatable({}, { __mode = "k" })

-- Local helper functions
local function helper_function(arg)
    -- implementation
end

-- Public API functions
--- Function description
-- @tparam type param_name Parameter description
-- @treturn type Return value description
function _M.public_function(param)
    -- implementation
end

return _M

-- vim: et:sw=4:ts=8:sts=4:tw=80
```

### Key Conventions

1. **Module pattern**: Always return a single table with exported functions
2. **Require at top**: Load all dependencies before any code
3. **Weak tables**: Use `setmetatable({}, { __mode = "k" })` for private data to avoid memory leaks
4. **Footer**: Every file must end with: `-- vim: et:sw=4:ts=8:sts=4:tw=80`

### Indentation and Formatting

- **4 spaces** for indentation (no tabs)
- **80 character** line width
- Local variables declared with `local`
- Use `_M` for module table (not `M` or other variants)

### Signal System Usage

Luakit uses the `lousy.signal` library for event handling:

```lua
local lousy = require("lousy")

-- Setup an object for signals
lousy.signal.setup(my_object)

-- Add signal handler
my_object:add_signal("event_name", function(obj, arg1, arg2)
    -- Handler code
end)

-- Emit signal
my_object:emit_signal("event_name", value1, value2)

-- Remove specific handler
my_object:remove_signal("event_name", handler_func)
```

**Important**: 
- Module objects are setup with `module=true`, which means the object itself is NOT passed as the first argument to handlers
- Regular objects receive the object as the first argument to handlers

### Settings System

Use the `settings` module for configuration:

```lua
local settings = require("settings")

-- Register setting
settings.register_setting("my_module.option", {
    type = "string",
    default = "value",
    validator = function(value)
        return type(value) == "string"
    end,
    desc = "Description of the setting",
})

-- Get setting value
local value = settings.get_setting("my_module.option")
```

---

## 🧪 Testing Standards

### Test Organization

- **`tests/async/`**: Functional tests for actual behavior
- **`tests/style/`**: Code quality and style tests
- **`tests/lib.lua`**: Test framework utilities

### Async Test Pattern

```lua
--- Test description
-- @copyright Year Name <email>

local T = {}

T.test_feature_name = function ()
    -- test implementation
end

T.test_another_feature = function ()
    -- another test
end

return T

-- vim: et:sw=4:ts=8:sts=4:tw=80
```

### Test Framework Functions

From `tests/lib.lua`:

```lua
test.wait_for_view(view)              -- Wait for webview to load
test.wait_for_signal(obj, signal)     -- Wait for signal emission  
test.wait_until(predicate)            -- Poll predicate function
test.delay(ms)                        -- Sleep briefly
test.find_files(dirs, patterns, exc)  -- Find files matching patterns
```

### Example Async Test

```lua
local T = {}
local test = require "tests.lib"

T.test_config_loads = function ()
    require "config.rc"
end

T.test_widget_creation = function ()
    local widget = require("widget")
    local w = widget{ type = "entry" }
    assert(w.type == "entry")
end

return T
```

### Style Test Pattern

Style tests validate code quality:

```lua
local test = require "tests.lib"

local T = {}

function T.test_module_blurb_not_empty ()
    local file_list = test.find_files({"lib", "doc/luadoc"}, "%.lua$",
        {"lib/lousy/widget/", "lib/lousy/init.lua"})
    
    local errors = {}
    for _, file in ipairs(file_list) do
        -- validation logic
        if not valid then
            table.insert(errors, { file = file, err = "Error message" })
        end
    end
    
    if #errors > 0 then
        error("Error summary:\n" .. test.format_file_errors(errors))
    end
end

return T
```

### Running Tests

```bash
# Run all tests
make run-tests

# Run specific async test
./luakit -c tests/run_test.lua tests/async/test_config.lua

# Run style tests
./luakit -c tests/run_test.lua tests/style/
```

---

## 📚 Documentation Standards

### LuaDoc Format

All modules and functions must be documented with LuaDoc annotations:

```lua
--- Module/function description (first line is summary)
--
-- Additional details can span multiple lines.
-- Provide examples and usage notes here.
--
-- @module module_name
-- @author Name <email>
-- @copyright Year Name <email>

--- Create a new instance
-- @tparam table opts Options table
-- @tparam string opts.name The name option
-- @tparam number opts.count The count option
-- @treturn table New instance
-- @usage
-- local obj = module.create{ name = "test", count = 5 }
function _M.create(opts)
    -- implementation
end
```

### LuaDoc Annotations Reference

- `@module` - Module identifier
- `@class` - Class definition
- `@function` - Function with parameters
- `@property` - Object property
- `@tparam type name` - Typed parameter
- `@treturn type` - Return type and description
- `@readonly` - Property is read-only
- `@readwrite` - Property can be read and written
- `@deprecated` - Mark as deprecated
- `@usage` - Example usage
- `@see` - Cross-reference
- `DOCMACRO(available:ui)` - Documentation macro

### Property Documentation

```lua
--- @property text
-- The text content of the widget.
-- @type string
-- @readwrite

--- @property position
-- The cursor position within the text.
-- @type integer
-- @readonly
```

### Class Documentation

Located in `doc/luadoc/`:

```lua
--- GTK+ user interface widgets
--
-- DOCMACRO(available:ui)
--
-- The `widget` class provides a wrapper around GTK's widget system,
-- allowing Lua code to build and modify the user interface.
--
-- @class widget
-- @author Mason Larobina
-- @copyright 2010 Mason Larobina <mason.larobina@gmail.com>

--- @function __call
-- Create a new widget.
-- @tparam table props Initial widget properties. A `type` field is mandatory.
-- @treturn widget The newly-constructed widget.
```

### Chrome Page Documentation

Chrome pages (internal UI pages) follow this pattern:

```lua
--- Description of the chrome page
--
-- Additional documentation...
--
-- @module module_name_chrome
-- @copyright Year Name <email>

chrome.add("module_name", function ()
    local html = [==[
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <title>Page Title</title>
</head>
<body>
    <!-- page content -->
</body>
</html>
    ]==]
    return html
end)
```

---

## 🔧 Build System Integration

### Makefile

The build system uses GNU Make:

```makefile
# Key targets
make all            # Build everything (luakit, docs, man page)
make luakit         # Main executable
make luakit.so      # WebKit extension library
make apidoc         # Generate API documentation
make clean          # Remove build artifacts
make install        # Install to system
make run-tests      # Run test suite
```

### Token Generation

When adding new properties:

1. Edit `common/tokenize.list` (keep alphabetically sorted)
2. Run `make` - automatically regenerates:
   - `common/tokenize.h` (token constants)
   - `common/tokenize.c` (token lookup tables)

### Documentation Generation

API docs are generated from source:

```bash
# Generate API documentation
make apidoc

# Uses build-utils/docgen/makedoc.lua
# Extracts LuaDoc comments from:
# - doc/luadoc/*.lua
# - lib/*.lua
# - common/clib/*.c
```

### Adding New Files

1. **C source files**: Automatically picked up by Makefile wildcards
2. **Lua modules**: Place in `lib/` or subdirectories
3. **Tests**: Add to `tests/async/` or `tests/style/`
4. **Documentation**: Add to `doc/luadoc/` for reference docs

---

## 📡 Signal System Conventions

### Core Signal Functions

From `lousy.signal`:

```lua
-- Setup object for signals
lousy.signal.setup(object, module)

-- Add signal handler
object:add_signal("signal-name", function(obj, ...)
    -- Handler receives object as first arg (unless module=true)
end)

-- Emit signal
object:emit_signal("signal-name", arg1, arg2)

-- Remove handler
object:remove_signal("signal-name", handler_func)

-- Remove all handlers
object:remove_signals("signal-name")
```

### Common Signals

**Window signals:**
- `"new-window"` - New window created
- `"close"` - Window closing

**Webview signals:**
- `"load-status"` - Page load status changed
- `"navigation-request"` - Navigation requested
- `"link-hover"` - Mouse over link
- `"link-unhover"` - Mouse left link

**Widget signals:**
- `"destroy"` - Widget destroyed
- `"property::*"` - Property changed

### Signal Naming

- Use lowercase with hyphens: `"load-status"`
- Use colons for namespacing: `"property::text"`
- Be descriptive: `"navigation-request"` not `"nav"`

---

## ⚠️ Common Mistakes to Avoid

### ❌ DON'T

1. **Don't use tabs** - Always use spaces (2 for C, 4 for Lua)
2. **Don't forget vim modeline** - Every Lua file must end with: `-- vim: et:sw=4:ts=8:sts=4:tw=80`
3. **Don't use string property lookups** - Always use token system in C code
4. **Don't skip the GPLv3 header** - All new C files need full license text
5. **Don't hardcode paths** - Use `buildopts.h` constants
6. **Don't introduce new patterns** - Use existing mechanisms
7. **Don't forget weak tables** - Use `__mode = "k"` for private data tables
8. **Don't skip documentation** - All public APIs need LuaDoc
9. **Don't commit generated files** - tokenize.c/h are auto-generated
10. **Don't use global variables** - Always use `local`

### ✅ DO

1. **Do follow existing patterns** - Find similar code and copy its style
2. **Do add tests** - Both async (functionality) and style (quality) tests
3. **Do run tests before committing** - `make run-tests`
4. **Do check whitespace** - `git diff --check`
5. **Do update tokens** - Add to `common/tokenize.list` when adding properties
6. **Do document everything** - Functions, modules, properties
7. **Do use weak tables** - For temporary/private data
8. **Do handle errors** - Check return values and use assertions
9. **Do emit signals** - For state changes
10. **Do maintain 80-char line width** - Keep code readable

---

## ✅ Pre-Commit Checklist

Before committing code, verify:

- [ ] **Code follows style guide**
  - [ ] C: 2-space indent, GPLv3 header, token-based properties
  - [ ] Lua: 4-space indent, vim modeline, weak tables for private data
- [ ] **Tokens updated** (if adding widget properties)
  - [ ] Added to `common/tokenize.list` (alphabetically)
  - [ ] Ran `make` to regenerate token files
- [ ] **Documentation complete**
  - [ ] LuaDoc comments on all public functions
  - [ ] Module description at file top
  - [ ] Property documentation with types
- [ ] **Tests added/updated**
  - [ ] Async tests for functionality
  - [ ] Style tests for code quality
  - [ ] All tests pass: `make run-tests`
- [ ] **Build succeeds**
  - [ ] `make clean && make` completes without errors
  - [ ] No compiler warnings
- [ ] **Whitespace clean**
  - [ ] `git diff --check` shows no issues
  - [ ] No trailing whitespace
  - [ ] Proper line endings
- [ ] **Commit message proper**
  - [ ] First line is short summary (no period)
  - [ ] Detailed explanation in body if needed
  - [ ] References issue number if applicable

---

## 📖 Exemplary Files for Learning

### C Widget Implementation
- **`widgets/entry.c`** - Simple text entry widget
  - Property table definition
  - Index/newindex functions
  - GTK+ integration
  - Token usage

### Lua Modules
- **`lib/lousy/signal.lua`** - Signal system implementation
  - Module structure
  - Weak tables
  - Documentation
  - Public API design

- **`lib/settings.lua`** - Settings system
  - Setting registration
  - Validation
  - Signal emission

### Testing
- **`tests/async/test_config.lua`** - Simple async test
- **`tests/style/test_documentation.lua`** - Style validation test
- **`tests/lib.lua`** - Test framework utilities

### Documentation
- **`doc/luadoc/widget.lua`** - Widget class documentation
  - Class description
  - Property documentation
  - Function documentation
  - Usage examples

### Chrome Pages
- **`lib/help_chrome.lua`** - Help page implementation
  - HTML structure
  - Chrome registration
  - Data integration

---

## 🔍 Code Review Focus Areas

When reviewing code (or preparing for review), check:

1. **Consistency**: Does it match existing code style?
2. **Documentation**: Are all public APIs documented?
3. **Tests**: Are there tests for new functionality?
4. **Tokens**: Are new properties in tokenize.list?
5. **Memory**: Are weak tables used correctly?
6. **Signals**: Are signals emitted for state changes?
7. **Errors**: Are errors handled properly?
8. **Performance**: Are expensive operations optimized?

---

## 🚀 Quick Reference

### File Locations
- **C code**: `clib/`, `common/`, `widgets/`
- **Lua code**: `lib/`
- **Tests**: `tests/async/`, `tests/style/`
- **Docs**: `doc/luadoc/`
- **Build**: `Makefile`, `config.mk`
- **Tokens**: `common/tokenize.list`

### Key Files
- `common/luaclass.c` - Lua object system
- `lib/lousy/signal.lua` - Signal system
- `lib/settings.lua` - Settings system
- `lib/chrome.lua` - Chrome page system
- `tests/lib.lua` - Test utilities

### Essential Patterns
- **C widget**: property table → index/newindex → registration
- **Lua module**: header → requires → private data → functions → return
- **Test**: require test.lib → test table → test functions → return
- **Doc**: module description → @annotations → function docs

---

## 📞 Getting Help

If you're unsure about how to implement something:

1. **Search the codebase** - Find similar existing code
2. **Read the exemplary files** - Listed in this document
3. **Check existing tests** - See how features are tested
4. **Ask maintainers** - Open an issue for discussion

**Remember**: When in doubt, copy an existing pattern rather than inventing a new one!
