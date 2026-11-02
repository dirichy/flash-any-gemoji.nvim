# flash-any-gemoji.nvim

[English](README.md)

为 [flash-any.nvim](https://github.com/dirichy/flash-any.nvim) 提供 emoji 码表，
用英文别名和关键词跳转到 emoji，支持 `👩‍💻`、`🇨🇳` 等由多个 Unicode 码点组成的条目。

本项目也可作为编写第三方码表 source 的示例。

## 安装

需要 Neovim 0.11+、flash-any.nvim 和 flash.nvim。使用 lazy.nvim 时，将本项目加入
flash-any.nvim 的依赖，并启用 `gemoji` source：

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

如果已经配置 flash-any.nvim，只需增加依赖，并将 `"gemoji"` 加入现有的 `sources`。
例如 `sources = { "pinyin", "gemoji", "unicode" }` 可混用中文编码、emoji 关键词和
Unicode 名称 fallback。扩展须在 flash-any.nvim 解析 source 前加载，无需调用注册函数。

## 使用

按 `s` 后输入 emoji 关键词，再选择跳转标签。输入不需要包围关键词的冒号。

| 文本 | 输入示例 |
| --- | --- |
| 😀 | `grinning`、`happy`、`smile` |
| 👩‍💻 | `coder`、`technologist`、`woman` |
| ❤️ | `heart`、`love` |
| 🇨🇳 | `china`、`cn`、`flag` |

默认使用 `matcher = "prefix"`，也可输入 `grin`、`cod` 等前缀。若希望只用完整编码匹配：

```lua
sources = { { name = "gemoji", matcher = "full_code" } }
```

## 编写第三方 source

插件需要两个 Lua 文件，放在 `lua/flash-any/` 命名空间下：

```text
flash-any-gemoji.nvim/
└── lua/flash-any/
    ├── sources/gemoji.lua
    └── tables/gemoji.lua
```

### Source 工厂

`lua/flash-any/sources/gemoji.lua` 返回函数：

```lua
return function()
  return {
    codetable = "gemoji",
    matcher = "prefix",
  }
end
```

`sources = { "gemoji" }` 中的名称用于寻找这个工厂；`codetable` 指向 `flash-any.tables`
下的数据模块，由主插件加载并管理缓存。工厂可接收 `opts`，读取扩展自己的配置选项。
`matcher` 可选 `full_code`、`initial`、`prefix`，也可提供自定义匹配函数。

### 码表

多码点 Lua 文件返回 `codes`、`prefixes`、`max_chars`。下面是可独立使用的简化
`lua/flash-any/tables/gemoji.lua` 示例：

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

- `codes` 保存文本到非空 ASCII 编码的映射；同一条文本可以有多个编码。
- `prefixes` 列出所有还能加长的文本真前缀，值为 `true`。即使前缀本身也有编码，
  仍应列入；叶子键省略。
- `max_chars` 是最长条目的 Unicode 码点数，与编码长度、显示出的字符数量不同。
  这里 `👩‍💻` 由三个码点组成。

提供者在文件中准备好前缀集合即可，无需自定义 matcher，也无需在 source 中配置长度。
只有单码点键的码表可直接返回编码映射。可选的 `tables/gemoji.tsv` 保存完整数据，
Lua 只放常用子集。TSV 应完整包含 Lua 编码条目，按 UTF-8 字节序排序，行格式为
`文本<Tab>编码1,编码2`；`max_chars` 应覆盖完整数据。

### 生成码表

准备 TSV，emoji 和逗号分隔的编码之间使用真实制表符：

```text
😀	grinning,happy,smile
👩‍💻	coder,technologist,woman
```

在已加载 flash-any.nvim 的 Neovim 中，从扩展根目录执行：

```vim
:FlashAnyBuildTable emoji.tsv lua/flash-any/tables/gemoji.lua
```

builder 会合并重复编码，并自动生成前缀集合和最长文本长度。像上面这样省略 TSV
输出路径，就只生成 Lua。如需同时生成排序后的完整 TSV：

```vim
:FlashAnyBuildTable emoji.tsv lua/flash-any/tables/gemoji.lua lua/flash-any/tables/gemoji.tsv
```

自定义 matcher、字符范围及完整接口约定见
[source 开发说明](https://github.com/dirichy/flash-any.nvim/blob/master/docs/sources.md)。
