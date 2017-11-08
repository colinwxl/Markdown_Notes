## [gVim](http://www.vim.org/) on Windows
<2017-11-8> by Colin Wu [colinabc@qq.com]
Refer to https://codingstyle.cn/topics/250

### Installation
1. Version: [Vim80](ftp://ftp.vim.org/pub/vim/pc/gvim80-586.exe)
2. Methods: Full Selection; not under system disk

### Configuration and Plugins
#### Basic Congiguration `_vimrc`

- Startup

```
" Startup {{{
filetype indent plugin on

" vim 文件折叠方式为 marker
augroup ft_vim
    au!
    au FileType vim setlocal foldmethod=marker
augroup END
" }}}
```

- General

```
" General {{{
set nocompatible
set nobackup
set noswapfile
set history=1024
set autochdir
set whichwrap=b,s,<,>,[,]
set nobomb
set backspace=indent,eol,start whichwrap+=<,>,[,]
" Vim 的默认寄存器和系统剪贴板共享
set clipboard+=unnamed
" 设置 alt 键不映射到菜单栏
set winaltkeys=no
" }}}
```

- Lang & Encoding

```
" Lang & Encoding {{{
set fileencodings=utf-8,gbk2312,gbk,gb18030,cp936
set encoding=utf-8
set langmenu=zh_CN
let $LANG = 'en_US.UTF-8'
"language messages zh_CN.UTF-8
" }}}
```

- GUI

```
" GUI {{{
set backgroud=dark
colorscheme solarized
source $VIMRUNTIME/delmenu.vim
source $VIMRUNTIME/menu.vim
set cursorline
set hlsearch
set number
" 窗口大小
set lines=35 columns=140
" 分割出来的窗口位于当前窗口下边/右边
set splitbelow
set splitright
"不显示工具/菜单栏
set guioptions-=T
set guioptions-=m
set guioptions-=L
set guioptions-=r
set guioptions-=b
" 使用内置 tab 样式而不是 gui
set guioptions-=e
set nolist
" set listchars=tab:\ ,eol:¬,trail:·,extends:>,precedes:<
set guifont=Inconsolata:h12:cANSI
" }}}
``` 

- Format

```
" Format {{{
set autoindent
set smartindent
set tabstop=4
set expandtab
set softtabstop=4
set foldmethod=indent
syntax on
syntax enable
" }}}
```

- Keymap

```
" Keymap {{{
let mapleader=","

nmap <leader>s :source $VIM/_vimrc<cr>
nmap <leader>e :e $VIM/_vimrc<cr>

map <leader>tn :tabnew<cr>
map <leader>tc :tabclose<cr>
map <leader>th :tabp<cr>
map <leader>tl :tabn<cr>

" 移动分割窗口
nmap <C-j> <C-W>j
nmap <C-k> <C-W>k
nmap <C-h> <C-W>h
nmap <C-l> <C-W>l

" 正常模式下 alt+j,k,h,l 调整分割窗口大小
nnoremap <M-j> :resize +5<cr>
nnoremap <M-k> :resize -5<cr>
nnoremap <M-h> :vertical resize -5<cr>
nnoremap <M-l> :vertical resize +5<cr>

" 插入模式移动光标 alt + 方向键
inoremap <M-j> <Down>
inoremap <M-k> <Up>
inoremap <M-h> <left>
inoremap <M-l> <Right>

" IDE like delete
inoremap <C-BS> <Esc>bdei

nnoremap vv ^vg_
" 转换当前行为大写
inoremap <C-u> <esc>mzgUiw`za
" 命令模式下的行首尾
cnoremap <C-a> <home>
cnoremap <C-e> <end>

nnoremap <F2> :setlocal number!<cr>
nnoremap <leader>w :set wrap!<cr>

imap <C-v> "+gP
vmap <C-c> "+y
vnoremap <BS> d
vnoremap <C-C> "+y
vnoremap <C-Insert> "+y
imap <C-V>      "+gP
map <S-Insert>      "+gP
cmap <C-V>      <C-R>+
cmap <S-Insert>     <C-R>+

exe 'inoremap <script> <C-V>' paste#paste_cmd['i']
exe 'vnoremap <script> <C-V>' paste#paste_cmd['v']

" 打开当前目录 windows
map <leader>ex :!start explorer %:p:h<CR>

" 打开当前目录CMD
map <leader>cmd :!start<cr>
" 打印当前时间
map <F3> a<C-R>=strftime("%Y-%m-%d %a %I:%M %p")<CR><Esc>

" 复制当前文件/路径到剪贴板
nmap ,fn :let @*=substitute(expand("%"), "/", "\\", "g")<CR>
nmap ,fp :let @*=substitute(expand("%:p"), "/", "\\", "g")<CR>

" 设置切换Buffer快捷键"
nnoremap <C-left> :bn<CR>
nnoremap <C-right> :bp<CR>

" }}}
```

#### Plugins [Using Vundle]
- Install Vundle
vundle安装插件的原理依赖于git和curl
1. 安装[chocolatey](https://chocolatey.org/)
chocolatey是windows下实用的包管理器，类似于ubuntu下的apt-get，用cmd.exe安装，使用管理员权限。
```
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

2. Install Git and Curl
choco install -y git
choco install -y curl

3. Install Vundle
`git clone https://github.com/VundleVim/Vundle.vim.git /e/Program\ Files/Vim/vimfiles/bundle`

4. Config _vimrc
首先，添加一个环境变量VIM到windows下:
cmd (Administrator)
set VIM=E:\Program Files\Vim
其次配置Vundle：

```
set nocompatible
filetype off 
" Plugins Management
set rtp+=$VIM/vimfiles/bundle/Vundle.vim/
call vundle#begin()

" 可以在此次安装插件
Plugin 'VundleVim/Vundle.vim'

call vundle#end()
filetype plugin indent on
```

5. Some Knowlegde (参见[linux] vim and its plugins(YouCompleteMe).md)
Vundle安装插件通过配置文件有两种形式
<1> 在`vundle#begin()` 和 `vundle#end` 之间，配置运行`Plugin '插件名称'`
<2> 直接配置一行 `Bundle '插件名称'`
在normal模式下，运行
`:PluginInstall` or `:BundleInstall`
<*> 常用命令：
```
:BundleInstall // 安装插件
:BundleInstall! // 更新插件
:BundleClean // 卸载插件
```
- Plugins 
1. vim-airline
```
Plugin 'vim-airline/vim-airline'
Plugin 'vim-airline/vim-airline-themes'
```
2. nerdTree
```
Plugin 'scrooloose/nerdtree'
```
```
" nerdTree快捷键映射
let NERDTreeWinPos='left'
let NERDTreeWinSize=30
map <F2> :NERDTreeToggle<CR>
```

3. emmet/vim-closetag
```
Plugin 'mattn/emmet-vim'
```
```
" 设置emmet快捷
let g:user_emmet_expandabbr_key = '<c-tab>'
let g:user_emmet_settings = {'indentation': '    '}
let g:user_emmet_install_global = 0
autocmd FileType html,css EmmetInstall
```
```
Plugin 'alvan/vim-closetag'
```

4. markdown
```
" markdown插件
Plugin 'godlygeek/tabular'
Plugin 'plasticboy/vim-markdown'
```

5. 快速打开文件
```
Plugin 'ctrlpvim/ctrlp.vim'
```
```
let g:ctrlp_map = '<c-p>'
let g:ctrlp_cmd = 'CtrlP'
set wildignore+=*\\node_modules\\*,*.git*,*.svn*,*.zip*,*.exe* " 使用vim的忽略文件
```

6. 多窗口
```
Plugin 'szw/vim-maximizer'
```
```
nnoremap <silent><F3> :MaximizerToggle<CR>
vnoremap <silent><F3> :MaximizerToggle<CR>gv
inoremap <silent><F3> <C-o>:MaximizerToggle<CR>
```

7. 全局搜索
```
Plugin 'dkprice/vim-easygrep'
```
<leader>vv 可以在当前目录下全局搜索指针选择的单词
:Grep 搜索字符串
:Replace [target] [replacement]

8. 注释代码
```
Plugin 'scrooloose/nerdcommenter'
```
```
let g:NERDSpaceDelims = 1
```

9. 光标快速移动
```
Plugin 'easymotion/vim-easymotion'
```

10. 浏览当前打开文件
```
Plugin 'jlanzarotta/bufexplorer'
```
```
nnoremap <silent><F8> :BufExplorer<CR>
```