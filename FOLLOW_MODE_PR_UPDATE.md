## ADDENDUM: Second Bug Fixed

After the initial fix was pushed, user reported another error:
```
E [core/common/lualib]: Lua error: append element error: elements not found
   (2) /usr/local/share/luakit/lib/select_wm.lua:321 in function init_frame
```

### Root Cause #2
The `init_frame()` function tries to append elements to `frame.body.parent` without checking if parent exists or is valid during page load/navigation.

### Fix #2 Applied (Commit: 96ca952)

**File:** `lib/select_wm.lua` (init_frame function)

**Changes:**
- Added check for `frame.body.parent` existence
- Wrapped `append()` calls in `pcall()` for graceful error handling
- Silently fails if overlay cannot attach (no crash)

**Code:**
```lua
-- Before
frame.body.parent:append(frame.overlay)
frame.body.parent:append(frame.stylesheet)

-- After
if frame.body.parent then
    pcall(function() frame.body.parent:append(frame.overlay) end)
    pcall(function() frame.body.parent:append(frame.stylesheet) end)
end
```

**Impact:** Follow mode works even when DOM not fully ready
