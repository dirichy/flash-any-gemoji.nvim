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

A multi-codepoint Lua file returns `codes`, `prefixes`, and `max_chars`. This is a
small standalone example for `lua/flash-any/tables/gemoji.lua`:

```lua
return {
  codes = {
    ["😀"] = { "grinning", "happy", "smile" },
    ["👩‍💻"] = { "coder", "technologist", "woman" },
  },
  prefixes = {
    ["👩"] = true,
    ["👩‍"] = true,
  },
  max_chars = 3,
}
```

- `codes` maps text entries to nonempty ASCII encodings. One entry may have several codes.
- `prefixes` lists every proper text prefix that can be extended, with value `true`.
  Include a prefix even when it also has its own codes. Omit terminal keys.
- `max_chars` is the longest text entry in Unicode codepoints, not the encoding
  length or the number of displayed glyphs. Here `👩‍💻` has three codepoints.

Keep the prefix set ready in the file; no custom matcher or source length option
is needed. Tables containing only single-codepoint keys may instead return the
encoding map directly. An optional `tables/gemoji.tsv` can hold the complete data
while Lua contains a common subset. TSV must contain every Lua code entry, be
sorted by UTF-8 bytes, and use `text<Tab>code1,code2`; `max_chars` must cover the full data.

### Generating a table

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
