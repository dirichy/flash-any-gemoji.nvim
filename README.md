# flash-any-gemoji.nvim

[简体中文](README.zh-CN.md)

An emoji source for [flash-any.nvim](https://github.com/dirichy/flash-any.nvim).
Jump to emoji using English aliases and keywords, including emoji made of multiple
Unicode codepoints, such as `👩‍💻` and `🇨🇳`.

This repository also serves as an example of writing a third-party code-table source.

## Installation

Requires Neovim 0.11+, flash-any.nvim, and flash.nvim. With lazy.nvim, add this
extension to flash-any.nvim's dependencies and enable the `gemoji` source:

```lua
{
  "dirichy/flash-any.nvim",
  dependencies = {
    "folke/flash.nvim",
    "dirichy/flash-any-gemoji.nvim",
  },
  opts = {
    sources = { "gemoji" },
  },
  keys = {
    {
      "s",
      mode = { "n", "x", "o" },
      function() require("flash-any").jump() end,
      desc = "Flash any",
    },
  },
}
```

If flash-any.nvim is already configured, add the dependency and append `"gemoji"`
to your existing `sources` list. For example, `sources = { "pinyin", "gemoji", "unicode" }`
combines Chinese codes, emoji keywords, and Unicode name fallback. The extension
must be loaded before flash-any.nvim resolves its sources; no registration call is needed.

## Usage

Press `s`, type an emoji keyword, and select a jump label. No surrounding colons
are needed.

| Text | Example input |
| --- | --- |
| 😀 | `grinning`, `happy`, `smile` |
| 👩‍💻 | `coder`, `technologist`, `woman` |
| ❤️ | `heart`, `love` |
| 🇨🇳 | `china`, `cn`, `flag` |

The source defaults to `matcher = "prefix"`, so prefixes such as `grin` and `cod`
also match. For complete-code matching, configure:

```lua
sources = { { name = "gemoji", matcher = "full_code" } }
```

## Writing a third-party source

The plugin needs two Lua files, under the `lua/flash-any/` namespace:

```text
flash-any-gemoji.nvim/
└── lua/flash-any/
    ├── sources/gemoji.lua
    └── tables/gemoji.lua
```

### Source factory

`lua/flash-any/sources/gemoji.lua` returns a function:

```lua
return function()
  return {
    codetable = "gemoji",
    matcher = "prefix",
  }
end
```

The name in `sources = { "gemoji" }` resolves this factory. `codetable` names the
data module under `flash-any.tables`; flash-any.nvim loads it and owns its caches.
A factory can accept `opts` for extension-specific configuration. Its `matcher`
may be `full_code`, `initial`, `prefix`, or a custom matcher function.

### Code table

The factory can accept `opts` to read extension-specific configuration options.

The `matcher` field supports `full_code`, `initial`, and `prefix`, or a custom matching function.

### Code Table

The Lua module returns a table containing `codes` and optionally `scope`.

Below is a minimal, self-contained example of `lua/flash-any/tables/gemoji.lua`:

```lua
return {
  codes = {
    ["😀"] = { "grinning", "happy", "smile" },
    ["👩‍💻"] = { "coder", "technologist", "woman" },
  },
  scope = { "😀", "👩‍💻" }, -- Optional, for fast preliminary filtering.
}
```

- `codes` maps text entries to non-empty ASCII codes. Each entry can have multiple codes.
- `scope` specifies the minimum and maximum strings in the TSV file, allowing fast preliminary filtering.

Providers only need to prepare the code table; no custom matcher is required.

Currently, code table entries are limited to a single grapheme cluster. Entries containing multiple grapheme clusters are not yet supported.

An optional `tables/gemoji.tsv` file can store the complete dataset, while the Lua module contains only a commonly used subset.

The TSV must include all entries defined in the Lua code table and be sorted in UTF-8 byte order. Each line has the following format:

`text<Tab>code1,code2`

The `scope` must cover the complete dataset.

### Generating the Code Table

Prepare a TSV with actual tabs between the emoji and its comma-separated codes:

```text
😀	grinning,happy,smile
👩‍💻	coder,technologist,woman
```

With flash-any.nvim loaded, run this from the extension's root directory:

```vim
:FlashAnyBuildTable emoji.tsv lua/flash-any/tables/gemoji.lua
```

The builder merges duplicate codes and generates the prefix set and maximum text
length. Omit the TSV output path for Lua-only output, as above. To also generate
a sorted complete TSV:

```vim
:FlashAnyBuildTable emoji.tsv lua/flash-any/tables/gemoji.lua lua/flash-any/tables/gemoji.tsv
```

See the [source author guide](https://github.com/dirichy/flash-any.nvim/blob/master/docs/sources.md)
for custom matchers, character scopes, and the complete source contract.
