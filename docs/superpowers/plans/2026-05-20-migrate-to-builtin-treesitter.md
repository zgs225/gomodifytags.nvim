# Migrate from nvim-treesitter to Built-in Treesitter Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Remove `nvim-treesitter` dependency by replacing `ts_utils.get_node_at_cursor()` with Neovim's built-in `vim.treesitter.get_node()`.

**Architecture:** Single-function replacement. The `getStructNameUnderCursor()` function in `main.lua` uses `nvim-treesitter.ts_utils` solely for `get_node_at_cursor(0)`. All other treesitter calls already use built-in APIs. Replace with `vim.treesitter.get_node()` (Neovim 0.9+) and clean up dependency declarations in `.luarc.json` and `README.md`.

**Tech Stack:** Lua, Neovim 0.9+, LuaLS type annotations

---

### Task 1: Replace treesitter call in main.lua

**Files:**
- Modify: `lua/gomodifytags/main.lua:2`
- Modify: `lua/gomodifytags/main.lua:199`

- [ ] **Step 1: Remove the nvim-treesitter require**

Delete line 2:
```lua
local ts_utils = require 'nvim-treesitter.ts_utils'
```

- [ ] **Step 2: Replace get_node_at_cursor call**

On line 199, change:
```lua
  local node = ts_utils.get_node_at_cursor(0)
```
to:
```lua
  local node = vim.treesitter.get_node()
```

- [ ] **Step 3: Verify the file has no remaining nvim-treesitter references**

```bash
rg 'nvim-treesitter' lua/gomodifytags/main.lua
```
Expected: no output

- [ ] **Step 4: Commit**

```bash
git add lua/gomodifytags/main.lua
git commit -m "refactor: replace nvim-treesitter with built-in vim.treesitter.get_node()"
```

---

### Task 2: Clean up .luarc.json

**Files:**
- Modify: `.luarc.json:8`

- [ ] **Step 1: Remove nvim-treesitter from workspace library**

Remove the line:
```json
        "~/.local/share/nvim/lazy/nvim-treesitter",
```

After removal, the `library` array in `.luarc.json` should look like:
```json
      "library": [
        "/opt/nvim/share/nvim/runtime/lua",
        "/opt/nvim/share/nvim/runtime/lua/vim/lsp",
        "./lua/"
      ],
```

- [ ] **Step 2: Commit**

```bash
git add .luarc.json
git commit -m "chore: remove nvim-treesitter from LuaLS workspace library"
```

---

### Task 3: Update README dependencies

**Files:**
- Modify: `README.md:24-25`

- [ ] **Step 1: Remove nvim-treesitter from lazy.nvim dependencies**

Remove lines 24-25:
```lua
  dependencies = {
    "nvim-treesitter/nvim-treesitter",
  },
```

After removal, the lazy.nvim install block should look like:
```lua
{
  "zgs225/gomodifytags.nvim",
  cmd = { "GoAddTags", "GoRemoveTags", "GoInstallModifyTagsBin" },
  config = function()
    require("gomodifytags").setup()
  end,
},
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "docs: remove nvim-treesitter from install dependencies"
```

---

### Task 4: Verification

- [ ] **Step 1: Review full diff**

```bash
git log --oneline -3
git diff HEAD~3..HEAD
```

Expected: three commits covering main.lua, .luarc.json, and README.md changes.

- [ ] **Step 2: Sanity check — open a Go file with a struct, place cursor inside, confirm `getStructNameUnderCursor()` still works**

No automated test suite exists for this plugin. The change is a pure API swap with identical behavior. Manual verification is sufficient.
