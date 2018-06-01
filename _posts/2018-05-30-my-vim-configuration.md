---
title: "My Vim Configuration and some vim command notes"
layout: post
date: 2018-05-30 22:45
tag:
- vim
- vundle
- .vimrc
- configuration
star: true
category: blog
author: jiaqixu
---

## Summary
This is a blog about how I set up my vim on Mac OSX step by step.

## Installation
1. Use `Homebrew` to install vim: `brew install vim`.
2. After Installation, we can use `vim --version` to check the version.<br>
   We need to make sure the vim installed meet the two following requirements(since we will use YCM plugin later):
   * the vim version should large than 7.4.143
   * it should support python2/3 language. Run `vim --version` to check if the `+python` or `+python3`is included.
    ```text
    VIM - Vi IMproved 8.1 (2018 May 17, compiled May 22 2018 22:53:39)
    macOS version
    Included patches: 1
    Compiled by Homebrew
    Huge version without GUI.  Features included (+) or not (-):
    +acl               +farsi             +mouse_sgr         -tag_any_white
    +arabic            +file_in_path      -mouse_sysmouse    -tcl
    +autocmd           +find_in_path      +mouse_urxvt       +termguicolors
    -autoservername    +float             +mouse_xterm       +terminal
    -balloon_eval      +folding           +multi_byte        +terminfo
    +balloon_eval_term -footer            +multi_lang        +termresponse
    -browse            +fork()            -mzscheme          +textobjects
    ++builtin_terms    -gettext           +netbeans_intg     +timers
    +byte_offset       -hangul_input      +num64             +title
    +channel           +iconv             +packages          -toolbar
    +cindent           +insert_expand     +path_extra        +user_commands
    -clientserver      +job               +perl              +vertsplit
    +clipboard         +jumplist          +persistent_undo   +virtualedit
    +cmdline_compl     +keymap            +postscript        +visual
    +cmdline_hist      +lambda            +printer           +visualextra
    +cmdline_info      +langmap           +profile           +viminfo
    +comments          +libcall           -python            +vreplace
    +conceal           +linebreak         +python3           +wildignore
    +cryptv            +lispindent        +quickfix          +wildmenu
    +cscope            +listcmds          +reltime           +windows
    +cursorbind        +localmap          +rightleft         +writebackup
    +cursorshape       -lua               +ruby              -X11
    +dialog_con        +menu              +scrollbind        -xfontset
    +diff              +mksession         +signs             -xim
    +digraphs          +modify_fname      +smartindent       -xpm
    -dnd               +mouse             +startuptime       -xsmp
    -ebcdic            -mouseshape        +statusline        -xterm_clipboard
    +emacs_tags        +mouse_dec         -sun_workshop      -xterm_save
    +eval              -mouse_gpm         +syntax
    +ex_extra          -mouse_jsbterm     +tag_binary
    +extra_search      +mouse_netterm     +tag_old_static
   ```
   from the picture above, we can know the vim version is 8.1 and it supports python3 but does not suppot python2.


## Vim Extension
Vim itself can satisfy many requirements of developers and it also has strong expandability. So, the first thing we need is to choose a good extension management.(I strongly recommend `vundle` which is like the `pip` in python)

Vim extension usually are aslo called `bundle` or `plugin`.


## Vundle
1. Install `Vundle`: `git clone https://github.com/gmarik/Vundle.vim.git ~/.vim/bundle/Vundle.vim`
2. In the home folder of mac OS user, build configuration file by `touch ~/.vimrc`
3. Then, we can add `vundel configruation` (see below) to the top of the `.vimrc` file
```text
  1 set nocompatible              " required
  2 filetype off                  " required
  3
  4 " set the runtime path to include Vundle and initialize
  5 set rtp+=~/.vim/bundle/Vundle.vim
  6 call vundle#begin()
  7
  8 " alternatively, pass a path where Vundle should install plugins
  9 "call vundle#begin('~/some/path/here')
 10
 11 " let Vundle manage Vundle, required
 12 Plugin 'gmarik/Vundle.vim'
 13 " Add all your plugins here (note older versions of Vundle used Bundle instead of Plugin)
 14 " filesystem
 15 Plugin 'kien/ctrlp.vim'
 16 Plugin 'nathanaelkane/vim-indent-guides'
 17 Plugin 'Valloric/YouCompleteMe'
 18 "git interface
 19 Plugin 'tpope/vim-fugitive'
 20 " All of your Plugins must be added before the following line
 21 call vundle#end()            " required
 22 filetype plugin indent on    " required
 ```
 After implementing the setting above, we can add the plugins we want to the related position of the `.vimrc` file; then open the vim app, running `:PluginInstall`, it will download all the plugins for us automatically.


## The Plugins I am using
1. [kien/ctrlp.vim](https://github.com/kien/ctrlp.vim): 用于文件操作
   ```text
   <c-p>：搜索文件
   <c-v>: 垂直分割打开文件
   <c-x>: 水平分割打开文件
   <c-t>: 在一个新的tab中打开文件
   <c-r>: 转换到regex模式
   <f5>：更新目录缓存
   ```
2. [nathanaelkane/vim-indent-guides](https://github.com/nathanaelkane/vim-indent-guides):用于高亮代码缩进
3. [tpope/vim-fugitive](https://github.com/tpope/vim-fugitive)：用于git操作
   ```text
   :Gdiff: 用于查看修改前后的文件与git仓库中文件的不同
   ```

## How to install [YouCompelteMe](https://github.com/Valloric/YouCompleteMe) Plugin
The installtion for YouCompleteMe is kind different with other Plugins, it is a bit more difficult.<br>
Steps to install YCM:
1. add `Plugin 'Valloric/YouCompleteMe'` to `~/.vimrc`
2. open `vim` and run `:PluginInstall`, it will take a bit long time
3. use homebrew to install `cmake` by run `brew install cmake`
4. build a directory to save the files produced by compilation process： `mkdir ~/.ycm_build`  and go to the dirctory with `cd ~/.ycm_build`
5. produce makefile: `cmake -G "Unix Makefiles" . ~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp`
6. produce `ycm_core.so` file: run `make ycm_core`
7. cpoy `.ycm_extra_conf.py`: run `cp ~/.vim/bundle/YouCompleteMe/third_party/ycmd/examples/.ycm_extra_conf.py ~/.vim/`
8. config vim by adding: 注意下面的 python 解释器的路径要和编译 ycm_core 的时候使用的 python 解释器是相同的版本（2 或 3）
    ```text
    let g:ycm_server_python_interpreter='/usr/bin/python'
    let g:ycm_global_ycm_extra_conf='~/.vim/.ycm_extra_conf.py'
    ```



















