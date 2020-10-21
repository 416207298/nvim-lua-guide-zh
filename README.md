# 在neovim中使用Lua

[nvim-lua-guide](https://github.com/nanotee/nvim-lua-guide) 中文版.

## 目录

* [简介](#introduction)
  * [学习 Lua](#learning-lua)
  * [其他关于在neovim中使用lua的教程](#existing-tutorials-for-writing-lua-in-neovim)
  * [相关的插件](#companion-plugins)
* [Lua文件的位置](#where-to-put-lua-files)
    * [警告](#caveats)
    * [提示](#tips)
    * [包说明](#a-note-about-packages)
* [在Vim文件中调用Lua](#using-lua-from-vimscript)
  * [:lua](#lua)
    * [Caveats](#caveats-1)
  * [:luado](#luado)
  * [:luafile](#luafile)
    * [luafile vs require():](#luafile-vs-require)
  * [luaeval()](#luaeval)
  * [v:lua](#vlua)
    * [Caveats](#caveats-2)
* [Vim命名空间](#the-vim-namespace)
    * [Tips](#tips-1)
* [在Lua中调用vimscript](#using-vimscript-from-lua)
  * [vim.api.nvim_eval()](#vimapinvim_eval)
    * [Caveats](#caveats-3)
  * [vim.api.nvim_exec()](#vimapinvim_exec)
  * [vim.api.nvim_command()](#vimapinvim_command)
    * [Tips](#tips-2)
* [管理vim设置选项](#managing-vim-options)
  * [使用api函数](#using-api-functions)
  * [使用元访问器](#using-meta-accessors)
    * [Caveats](#caveats-4)
* [管理vim内部变量](#managing-vim-internal-variables)
  * [使用api函数](#using-api-functions-1)
  * [使用元访问器](#using-meta-accessors-1)
    * [Caveats](#caveats-5)
* [调用vimscript函数](#calling-vimscript-functions)
  * [vim.call()](#vimcall)
  * [vim.fn.{function}()](#vimfnfunction)
    * [Tips](#tips-3)
    * [Caveats](#caveats-6)
* [定义键位映射](#defining-mappings)
* [定义用户命令](#defining-user-commands)
* [定义自动命令](#defining-autocommands)
* [定义语法高亮](#defining-syntaxhighlights)
* [一般提示和备注](#general-tips-and-recommendations)
* [杂项](#miscellaneous)
  * [vim.loop](#vimloop)
  * [vim.lsp](#vimlsp)
  * [vim.treesitter](#vimtreesitter)
  * [Transpilers](#transpilers)

## 简介

Lua作为Neovim中的一流语言的集成正在成为它的杀手级特性之一。 然而，学习如何用Lua编写插件的教程数量并不像用Vimscript编写插件那样多。 这是一种尝试，试图提供一些基本信息,让人们可以使用Lua编写neovim插件。
本指南假定您使用的是最新的Neovim[Nighly build](https://github.com/neovim/neovim/releases/tag/nightly)]。 由于Neovim的0.5版本是开发版本，请记住，正在积极开发的一些API并不十分稳定，
在发布之前可能会发生变化。

### 学习Lua

不同于原版教程，一下资源适用于国内用户:
- [在Y分钟内学习X关于Lua的页面](https://learnxinyminutes.com/docs/lua/)
- [Lua菜鸟教程](https://www.runoob.com/lua/lua-tutorial.html)
- [Lua用户维基](http://lua-users.org/wiki/LuaDirectory) 
- [Lua的官方参考手册](https://www.lua.org/manual/5.1/)

Lua是一种非常干净和简单的语言。 它很容易学习，特别是如果你有其他编程语言基础的例如typescript/javascript等，会更加容易上手Lua。注意：Neovim嵌入的Lua版本是LuaJIT 2.1.0，它与Lua 5.1保持兼容(带有几个5.2扩展)

### 现有的一些在neovim中使用Lua的教程

已经编写了一些教程来帮助人们用Lua编写插件。 他们中的一些人在写这本指南时提供了不少的帮助。 非常感谢它们的作者。

- [teukka.tech - 从init.vim转到init.lua](https://teukka.tech/luanvim.html)
- [2n.pl - 如何使用Lua编写neovim插件](https://www.2n.pl/blog/how-to-write-neovim-plugins-in-lua.md)
- [2n.pl - 如何使用Lua制作neovim UI](https://www.2n.pl/blog/how-to-make-ui-for-neovim-plugins-in-lua)
- [ms-jpq - NeoVim 异步教程](https://ms-jpq.github.io/neovim-async-tutorial/)

### 相关插件

- [Vimpeccable](https://github.com/svermeulen/vimpeccable) - Plugin to help write your .vimrc in Lua
- [plenary.nvim](https://github.com/nvim-lua/plenary.nvim) - All the lua functions I don't want to write twice
- [popup.nvim](https://github.com/nvim-lua/popup.nvim) - An implementation of the Popup API from vim in Neovim
- [nvim_utils](https://github.com/norcalli/nvim_utils)
- [nvim-luadev](https://github.com/bfredl/nvim-luadev) - REPL/debug console for nvim lua plugins 
- [nvim-luapad](https://github.com/rafcamlet/nvim-luapad) - Interactive real time neovim scratchpad for embedded lua engine
- [nlua.nvim](https://github.com/tjdevries/nlua.nvim) - Lua Development for Neovim 
- [galaxyline.nvim](https://github.com/glepnir/galaxyline.nvim) - neovim statusline plugin written in lua
- [BetterLua.vim](https://github.com/euclidianAce/BetterLua.vim) - Better Lua syntax highlighting in Vim/NeoVim 

## Lua 文件位置

Lua文件通常位于您的`runtimepath`中的`lua/`文件夹中(对于大多数用户来说，在*nix系统上为`~/.config/nvim/lua`，在Windows系统上为`~/appdata/Local/nvim/lua`)。 `Package.path`和`Package.cpath`全局变量会自动调整为包含该文件夹下的Lua文件。 这意味着您可以`require()`这些文件作为Lua模块

我们以下面的文件夹结构为例：

```text
📂 ~/.config/nvim
├── 📁 after
├── 📁 ftplugin
├── 📂 lua
│  ├── 🌑 myluamodule.lua
│  └── 📂 other_modules
│     ├── 🌑 anothermodule.lua
│     └── 🌑 init.lua
├── 📁 pack
├── 📁 plugin
├── 📁 syntax
└── 🇻 init.vim
```

下面的Lua代码将加载`myluamodule.lua`

```lua
require('myluamodule')
```

注意没有`.lua`扩展名。

类似地，加载`ther_module/anthermodule e.lua`的过程如下：

```lua
require('other_modules.anothermodule')
-- or
require('other_modules/anothermodule')
```

路径分隔符可以用点`.`表示，也可以用斜杠`/`表示。

文件夹如果包含`init.lua`文件，可以直接引用该文件夹而不必指定该文件的名称

```lua
require('other_modules') -- loads other_modules/init.lua
```

更多信息 `:help lua-require`

#### 警告

与.vim文件不同，.lua文件不会自动从您的`runtimepath`目录中获取。 相反，您必须从Vimscript source/require 它们。 计划增加`init.lua`文件加载选项，替代`init.vim`：
- [Issue #7895](https://github.com/neovim/neovim/issues/7895)
- [Corresponding pull request](https://github.com/neovim/neovim/pull/12235)

#### 提示

多个Lua插件在它们的`lua/`文件夹中可能有相同的文件名。 这可能会导致命名空间冲突。如果两个不同的插件有一个`lua/main.lua`文件，那么执行`require('main')`是不明确的：我们想要加载哪个文件？
最好将您的配置或插件命名为顶级文件夹，如下所示：`lua/plugin_name/main.lua`。
