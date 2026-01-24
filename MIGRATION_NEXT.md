# Migration Guide for Next Release

This document tracks API changes that will be included in the next release of Luakit.

## Overview

This file documents all breaking changes and API modifications that users need to be aware of when upgrading to the next version of Luakit. Each entry includes the old and new syntax, the reason for the change, and specific migration instructions for user configuration files.

## No Breaking Changes Yet

There are currently no documented breaking changes for the next release.

---

## Template for Adding New Entries

When documenting a new API change, use the following format:

```markdown
### [Module/Class Name] - [Property/Method/Signal Name]

**Changed:** `old.api.syntax` → `new.api.syntax`

**Reason:** Brief one-sentence explanation of why this change was necessary.

**Migration:** What users need to change in their `rc.lua`, `userconf.lua`, or `theme.lua`:

```lua
-- Old code (remove this):
old.api.syntax = value

-- New code (use this instead):
new.api.syntax = value
```  
```

---

## Guidelines

- **One entry per API change:** Each modification should have its own section
- **Be specific:** Include exact code examples that users can copy
- **Test examples:** Ensure all migration code actually works
- **Link to documentation:** Reference relevant API docs where appropriate
- **Include version info:** Note when the change was introduced