---
title: My Personal Config for Some IDEs
date: 2025-05-17 12:17:46
tags: 
  - Vim
  - VSCode
  - JetBrains
categories:
  - Development
---

## VSCode Vim Keymap
```json
"vim.searchHighlightColor": "rgba(150, 255, 255, 0.3)",
  "vim.surround": true,
  "vim.leader": "<space>",
  "vim.easymotion": true,
  "vim.incsearch": true,
  "vim.useSystemClipboard": true,
  "vim.useCtrlKeys": true,
  "vim.hlsearch": true,
  "vim.handleKeys": {
    "<C-a>": false,
    "<C-f>": false
  },
  "vim.insertModeKeyBindings": [
    {
      "before": ["j", "k"],
      "after": ["<Esc>"]
    }
  ],
  "vim.normalModeKeyBindings": [
    {
      "before": ["leader", "e"],
      "commands": [
        {
          "command": "workbench.action.toggleSidebarVisibility"
        }
      ]
    },
    {
      "before": ["leader", "h"],
      "commands": [
        {
          "command": ":noh"
        }
      ]
    },
    {
      "before": ["leader", "q"],
      "commands": [":q"]
    },
    {
      "before": ["leader", "w"],
      "commands": [":w"]
    },
    {
      "before": ["leader", "o"],
      "commands": [
        {
          "command": "workbench.action.quickOpen"
        }
      ]
    },
    {
      "before": ["leader", "t"],
      "commands": [
        {
          "command": "workbench.action.terminal.focus"
        }
      ]
    },
    {
      "before": ["leader", "d"],
      "commands": [
        {
          "command": "workbench.action.closeActiveEditor"
        }
      ]
    },
    {
      "before": ["leader", "s"],
      "commands": [
        {
          "command": "workbench.action.files.saveAll"
        }
      ]
    },
    {
      "before": ["H"],
      "commands": [
        {
          "command": "workbench.action.previousEditor"
        }
      ]
    },
    {
      "before": ["L"],
      "commands": [
        {
          "command": "workbench.action.nextEditor"
        }
      ]
    }
  ],
  "vim.visualModeKeyBindings": [
    {
      "before": ["leader", "l"],
      "commands": [
        {
          "command": "editor.action.indentLines"
        }
      ]
    },
    {
      "before": ["leader", "h"],
      "commands": [
        {
          "command": "editor.action.outdentLines"
        }
      ]
    },
    {
      "before": ["J"],
      "commands": [
        {
          "command": "editor.action.moveLinesDownAction"
        }
      ]
    },
    {
      "before": ["K"],
      "commands": [
        {
          "command": "editor.action.moveLinesUpAction"
        }
      ]
    }
  ]
```

## IdeaVim Keymap
```.ideavimrc
" Basic setting
set showmode
set relativenumber
set clipboard += unnamed
set keep-english-in-normal

" Set leader key
let mapleader = " "

" Search setting
set incsearch
set hlsearch
set ignorecase
set smartcase

" Plugins
set easymotion
set surround
set NERDTree

" Basic key mapping
inoremap jk <esc>
nnoremap ; :
nnoremap <leader>h :nohlsearch<cr>
nnoremap <leader>d <c-d>
nnoremap <leader>u <c-u>
nnoremap <leader>q :wq<cr>
nnoremap <leader>w :w<cr>

" Windows operation
nnoremap \ <c-W>v
nnoremap - <c-W>s
nnoremap <leader>wh <c-W>h
nnoremap <leader>wj <c-W>j
nnoremap <leader>wk <c-W>k
nnoremap <leader>wl <c-W>l

" Tabs operation
nnoremap H gT
nnoremap L gt

" Config IdeaVim
nnoremap <leader>ve :e ~/.ideavimrc<cr>
nnoremap <leader>vs :source ~/.ideavimrc<cr>

" Coding
nnoremap gi :action GotoImplementation<cr>
nnoremap gd :action GotoDeclaration<cr>
nnoremap <leader>e :action ActivateProjectToolWindow<cr>
nnoremap <leader>gf :action GotoFile<cr>
nnoremap <leader>/ :action CommentByLineComment<cr>
vnoremap <leader>/ :action CommentByLineComment<cr>
nnoremap <leader>nj :action NewClass<cr>
nnoremap <leader>mr :action Maven.Reimport<cr>
nnoremap <leader>= :action ReformatCode<cr>
nnoremap <leader>- :action OptimizeImports<cr>
```