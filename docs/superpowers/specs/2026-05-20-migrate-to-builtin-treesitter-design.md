# Migrate from nvim-treesitter to Neovim Built-in Treesitter

## Context

The `nvim-treesitter` plugin has been archived. This plugin (`gomodifytags.nvim`) currently depends on it for exactly one function call: `ts_utils.get_node_at_cursor(0)` in `getStructNameUnderCursor()`. All other treesitter API calls already use Neovim's built-in API.

## Goal

Remove the `nvim-treesitter` dependency entirely by replacing it with Neovim's built-in `vim.treesitter` API. Minimum Neovim version: 0.9+.

## Changes

Three files are affected:

### 1. `lua/gomodifytags/main.lua`

- **Remove line 2**: `local ts_utils = require 'nvim-treesitter.ts_utils'`
- **Line 199**: Replace `ts_utils.get_node_at_cursor(0)` with `vim.treesitter.get_node()`

No other code changes. The existing calls to `node:type()`, `node:parent()`, `node:child()`, and `vim.treesitter.get_node_text()` are already built-in Neovim APIs.

`vim.treesitter.get_node()` with no arguments defaults to buffer 0 and the current cursor position, matching the previous behavior exactly.

### 2. `.luarc.json`

- Remove `"~/.local/share/nvim/lazy/nvim-treesitter"` from `workspace.library`

### 3. `README.md`

- Remove `"nvim-treesitter/nvim-treesitter",` from the `dependencies` list in the lazy.nvim install example

## Compatibility

- `vim.treesitter.get_node()` was introduced in Neovim 0.9
- No breaking changes to the public API or command behavior
- Function signatures and behavior are preserved

## Testing

Manually verify:
1. `:GoAddTags json` on a Go struct — should find the struct and add tags
2. `:GoRemoveTags` on a Go struct — should clear all tags
3. Both commands should produce identical results before and after the change

Automated tests are not required since existing behavior is preserved.
