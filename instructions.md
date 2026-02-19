# NeoVim Setup and Configuration (macOS)

## Overview
Neovim (NVIM) is a terminal-based text editor derived from Vim that emphasizes keyboard-driven editing.
This document specifies the steps required to install Neovim on macOS and configure a minimal,
modern development environment using Lua-based configuration, core editor options, and a Tree-sitter + LSP
for syntax parsing.

> [!WARNING]
> **MacOS only. This guide will only be successful for macOS users.**

Prerequisites:
- Terminal access
- Homebrew installed
- Basic shell usage

Scope:
- Install Neovim
- Initialize configuration directory
- Configure core options and key mappings
- Install and configure Tree-sitter using lazy.nvim
- Install and configure a Language Processing Server for code autocomplete

---

## Installing NVIM

1. Open a new terminal window.
2. Type ```brew --version``` to ensure Homebrew is installed on your system.
   - To install Homebrew check out [Homebrew Install](https://brew.sh/).
3. Once Homebrew is installed run ```brew install neovim```.
4. To open NeoVim:
```
nvim <filename> # to open a file
nvim .          # to open a directory view
```

> [!Note]
> When you open NeoVim with the above commands it will turn your terminal into an editor and look something like this!
> <img width="714" height="446" alt="image" src="https://github.com/user-attachments/assets/e6340dbd-ccdf-42a9-90c3-6e30350a6124" />


5. To exit NeoVim:
```
:q             quit
:q!            quit without saving
:wq            save and quit
<shift>ZZ      save and quit
```

Congratulations, you've successfully installed NVIM. Next let's make this into a more usable editor.
To follow the rest of the guide get comfortable with basic [VIM mouse-free movement](https://phoenixnap.com/kb/vim-keybindings).


## Setting up the Config Folder
What separates NVIM from other editors like VSCode, is the options it gives you to customize and
mold it to your workflow. Before you start configuring NVIM, you need to set up a file structure
to keep everything organized. 

***In a terminal***
1. Enter ```cd ~/.config``` to go to the .config folder of your user directory.
2. Type ```ls``` and see if you have a directory called ```nvim```.
   - If you have the folder move on.
   - Otherwise create it with the command ```mkdir nvim```.
3. Enter the folder by doing ```cd nvim```.
4. Create your init.lua file ```nvim init.lua``` - Lua is the language NeoVim uses for configuration.
5. Save the file by entering ```:wq!```.
6. Make a new sub directory called lua ```mkdir lua```.
7. Go back to your main nvim directory by typing ```cd ~/.config/nvim```.

When you type ```ls``` you should see an ```init.lua``` file and a ```/lua``` directory. 
Next as we configure our NVIM setup the initialization will start in the init file, and automatically 
load Lua modules from the /lua directory, allowing you to split your config into clean, reusable files.


## Adding Useful Key-binds
NVIM comes with a lot of possible settings and key-binds to adjust. Here is how to do it with your current setup!

***In a terminal***
1. Navigate to your nvim config folder ```cd ~/.config/nvim```.
2. Go into your /lua directory ```cd /lua```.
3. Create a new file named options.lua ```nvim options.lua``` This file is where you will adjust all
   your favorite built-in NVIM options.
4. To have relative line numbers insert the lines of text below:
```
vim.wo.relativenumber = true
vim.opt.number = true
``` 
If you save and reopen this file you should now see line numbers. Like this ↓↓↓
![ScreenRecording2026-02-18at11 42 25PM-ezgif com-video-to-gif-converter](https://github.com/user-attachments/assets/496b5867-adab-4592-ac12-078c7bfb7e8a)

5. Now to set your leader key to be the space-bar ```vim.g.mapleader = " "```.
6. To add a keymapping to go to directory view ```vim.keymap.set("n", "<leader>pv", vim.cmd.Ex)```.

Here is an example options file:
```
-- Line Numbers
vim.wo.relativenumber = true      -- Show relative line numbers for easier movement
vim.opt.number = true             -- Show absolute line number on the current line

-- Indentation & Tabs
vim.o.tabstop = 4                 -- A TAB character displays as 4 spaces
vim.o.expandtab = true            -- Convert TAB key presses into spaces
vim.o.softtabstop = 4             -- Spaces inserted when pressing TAB
vim.o.shiftwidth = 4              -- Spaces used for auto-indentation
vim.opt.smartindent = true        -- Auto-indent based on code structure

-- Keybindings
vim.g.mapleader = " "             -- Set leader key to SPACE
vim.keymap.set("n", "<leader>pv", vim.cmd.Ex)

-- Editor Behavior
vim.opt.wrap = false              -- Disable line wrapping
vim.opt.virtualedit = "all"       -- Allow cursor movement past end of line/file
vim.opt.scrolloff = 8             -- Keep context lines visible when scrolling
vim.opt.updatetime = 50           -- Faster updates (better LSP & git responsiveness)

-- Search
vim.opt.hlsearch = false          -- Disable persistent search highlighting
vim.opt.incsearch = true          -- Show matches as you type

-- UI & Visuals
vim.opt.termguicolors = true      -- Enable 24-bit RGB colors
vim.opt.signcolumn = "yes"        -- Always show sign column (LSP, git)
vim.opt.colorcolumn = "99"        -- Vertical guide at column 99
vim.opt.guicursor = ""            -- Use block cursor in all modes

-- File Handling
vim.opt.isfname:append("@-@")     -- Allow '@' in filenames

```

7. Save and exit your file ```<shift>ZZ```.
8. Go back to your init file ```cd ..; nvim init.lua```.
9. Link your options file to your init.lua by inserting ```require("options")```.

> [!TIP]
> All vim options need to be declared as such ```vim.opt.<option_name> = <value>```.
>
> All vim keybinds follow the same format ```vim.keymap.set("n", "<keybind>", "<command-to-execute>")```.


Now that you know how to add some basic vim options here is a [list of more](https://vimhelp.org/options.txt.html#option-summary) to choose from.


## Installing a Tree-sitter Plugin
Every modern code editor has a Tree-sitter. Essentially it parses your code in real time to determine
what parts are related to each other for better code highlighting. 

> [!NOTE]
>For more efficient package management LazyVim will be used as a dependency manager


1. Navigate to your nvim config folder ```cd ~/.config/nvim```.
2. Open your init.lua ```nvim init.lua```.
3. Press ```<shift>o``` to enter insert mode above your current line.
4. Add the snippet to the top of your code. This ensures lazyVim is installed and installs it if not:
```
local lazypath = vim.fn.stdpath("data") .. "/lazy/lazy.nvim"
if not vim.loop.fs_stat(lazypath) then
  vim.fn.system({
    "git", "clone", "--filter=blob:none",
    "https://github.com/folke/lazy.nvim.git",
    "--branch=stable",
    lazypath,
  })
end
vim.opt.rtp:prepend(lazypath)
```
5. Save the file ```<shift>ZZ```.
6. Go to your /lua directory ```cd /lua```.
7. Create a new plugins.lua file ```nvim plugins.lua```.
   - if this is your first time running nvim with lazyvim there will be a pop-up. To close this
     type ```:q!```.
8. insert the following code:
```
require("lazy").setup({



})
```
9. To add a plugin all you have to do is add a lua dictionary inside the above function call. 
   For your Tree-sitter add:
```
require("lazy").setup({


  {
    "nvim-treesitter/nvim-treesitter",
    build = ":TSUpdate",
    config = function()
      require("nvim-treesitter.configs").setup({
        ensure_installed = { "go", "lua", "python", "javascript" }, -- add any languages you want
        highlight = {
          enable = true,            -- false will disable the whole extension
          additional_vim_regex_highlighting = true,
        },
        indent = {
          enable = true,
        },
      })
    end,
  },

})
```

10. Save your file ```:wq```.
11. Open your init file ```cd ..; nvim init.lua```.
12. Go to the bottom of your file ```<shift>g```.
13. Insert ```require("plugins")```.
14. Save your file ```<shift>ZZ```.

## Add an LSP
An LSP allows for code syntax error highlighting and autocomplete.

1. Go to your plugins file ```cd ~/.config/nvim/lua; nvim plugins.lua```.
2. Inside the require method add: 
```
  { "williamboman/mason.nvim" },
  { "williamboman/mason-lspconfig.nvim" },
```
3. Save your file ```<shift>ZZ```.
4. Create a new file to configure masonLSP ```nvim mason-setup.lua```.
5. Add the following lines:
```
require("mason").setup()

local mason_lsp = require("mason-lspconfig")
mason_lsp.setup({
    ensure_installed = { "lua_ls", "pyright", "clangd"}, -- Install selected servers automatically

    automatic_installation = true,
})
```
6. To add or remove specific language processing capabilities edit ```ensure_installed```.
   here are all the ones that are supported [LSPs](https://mason-registry.dev/registry/list).
7. Save your file ```:wq```.
8. Open your init.lua ```cd ..; nvim init.lua```.
9. At the very bottom add ```require("mason-setup")```.

---

## Conclusion
Now you have a modern and customizable text editor that you can mold to your liking. For addtional
packages refer to the process outlined above!


