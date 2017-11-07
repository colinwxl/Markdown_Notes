## [Vim](ftp://ftp.vim.org/pub/vim/unix/) and Its Plugins ([YouCompleteMe](http://valloric.github.io/YouCompleteMe/))
<2017.0928-0929> by colin wu
### Vim Installation
卸载老版本：`sudo apt-get remove vim (vim-runtime; gvim; vim-tiny; vim-common; vim-gui-common)`
未出现依赖库缺失情况，略过。
```
wget ftp://ftp.vim.org/pub/vim/unix/vim-8.0.tar.bz2 --no-check-certificate
tar -xjvf vim-8.0.tar.bz2
cd vim80
./configure --with-features=huge \ # 支持最大特性
	--enable-rubyinterp \ # 启用vim对ruby编写的插件的支持
	--enable-pythoninterp \ # 启用vim对python编写的插件的支持
	--with-python-config-dir=/home/wuxiaolong/tools/anaconda2/lib/python2.7/config/ \ # 指定python路径
	--enable-perlinterp \ # 启用vim对perl编写的插件支持
	--enable-gui=gtk2 \ # gtk支持，也可以使用genome，表示生成gvim
	--enable-cscope \ # vim对cscopezhichi
	--enable-luainterp \ # 启用vim对lua编写的插件支持
	--enable-multibyte \ # 多字节支持，可以在vim中输入中文
	--prefix=/home/wuxiaolong/tools/vim80 # 编译安装路径
make && make install
echo “export PATH=/home/wuxiaolong/tools/vim80/bin:$PATH” >> ~/.bashrc
```

### Vim Configuration
配置文件：`~/.vimrc`，注释符号：`"`
```
set nocompatible "去掉vi的一致性
set number "显示行号
" 隐藏滚动条    
set guioptions-=r 
set guioptions-=L
set guioptions-=b

set showtabline=0 "隐藏顶部标签栏
set guifont=Consolas:h13 "设置字体       
syntax on    "开启语法高亮
syntax enable
let g:solarized_termcolors=256    "solarized主题设置在终端下的设置
"设置背景色
set t_Co=256 " add "export TERM=xterm-256color
set termguicolors
if has('gui_running')
	set background=light
else
	set background=dark
endif

colorscheme solarized " 设置主题
"set nowrap    "设置不折行
set fileformat=unix    "设置以unix的格式保存文件
set cindent        "设置C样式的缩进格式
set tabstop=4    "设置table长度
set shiftwidth=4        "同上
set showmatch    "显示匹配的括号
set scrolloff=5        "距离顶部和底部5行
set laststatus=2    "命令行为两行
set fenc=utf-8      "文件编码
set backspace=2
set mouse=a        "启用鼠标
set selection=exclusive
set selectmode=mouse,key
set matchtime=5
"set ignorecase        "忽略大小写
set incsearch
set hlsearch        "高亮搜索项
set noexpandtab        "不允许扩展table
set whichwrap+=<,>,h,l
set autoread
set cursorline        "突出显示当前行
"set cursorcolumn        "突出显示当前列
set pastetoggle=<F9>
set smartindent
set foldenable
set foldmethod=syntax
```
### Plugins Installation
~/.vim/bundle
~/.vim/colors
~/.vim/plugin
> [Git](https://www.kernel.org/pub/software/scm/git/); Installation
Source: Github; [vim-Scripts](http://vim-scripts.org/vim/scripts.html)

#### Vundle Installation
1> Set up Vundle

`git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`
2> Configure Plugins `~/.vimrc`
```
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
" Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Install L9 and avoid a Naming conflict if you've already installed a
" different version somewhere else.
" Plugin 'ascenator/L9', {'name': 'newL9'}

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
```
3> Install Plugins:
Launch vim and run `:PluginInstall`

#### Plugins Setting
##### [NERDTree](https://github.com/scrooloose/nerdtree)
1> Installation
`git clone https://github.com/scrooloose/nerdtree.git ~/.vim/bundle/nerdtree`
2> Setting `~/.vimrc`
1. open a NERDTree automatically when vim starts up:
   `autocmd vimenter * NERDTree`
2. open a NERDTree automatically when vim starts up if no files were specified:
```
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
```
3.  open NERDTree automatically when vim starts up on opening a directory:
```
autocmd StdinReadPre * let s:std_in=1
autocmd VimEnter * if argc() == 1 && isdirectory(argv()[0]) && !exists("s:std_in") | exe 'NERDTree' argv()[0] | wincmd p | ene | endif
```
4. map a specific key or shortcut to open NERDTree:
   `map <C-n> :NERDTreeToggle<CR>`
5. close vim if the only window left open is a NERDTree:
   `autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTree") && b:NERDTree.isTabTree()) | q | endif`
6. have different highlighting for different file extensions:
```
" NERDTress File highlighting
function! NERDTreeHighlightFile(extension, fg, bg, guifg, guibg)
 exec 'autocmd filetype nerdtree highlight ' . a:extension .' ctermbg='. a:bg .' ctermfg='. a:fg .' guibg='. a:guibg .' guifg='. a:guifg
 exec 'autocmd filetype nerdtree syn match ' . a:extension .' #^\s\+.*'. a:extension .'$#'
endfunction
call NERDTreeHighlightFile('jade', 'green', 'none', 'green', '#151515')
```
https://github.com/scrooloose/nerdtree/issues/433#issuecomment-92590696
7. change default arrows:
```
let g:NERDTreeDirArrowExpandable = '▸'
let g:NERDTreeDirArrowCollapsible = '▾'
```

##### [indentLine](https://github.com/Yggdroot/indentLine)
This plugin is used for displaying thin vertical lines at each indentation level for code indented with spaces. For code indented with tabs I think there is no need to support it, because you can use `:set list lcs=tab:\|\` (here is a space).
1> Installation
   `git clone https://github.com/Yggdroot/indentLine.git ~/.vim/bundle/indentLine`
2> Setting `~/.vimrc`
1. Change Character Color
   `let g:indentLine_setColors = 0`
```
" Vim
let g:indentLine_color_term = 239
" Background (Vim, GVim)
let g:indentLine_bgcolor_term = 202
let g:indentLine_bgcolor_gui = '#FF5F00'
```
2. Change Indent Char
`let g:indentLine_char = 'c'`
where 'c' can be any ASCII character. You can also use one of ¦, ┆, │, ⎸, or ▏ to display more beautiful lines. However, these characters will only work with files whose encoding is UTF-8.
3. Change Conceal Behaviour
```
let g:indentLine_concealcursor = 'inc'
let g:indentLine_conceallevel = 2
let g:indentLine_setConceal = 0
```
4. Disable by defaul
   `let g:indentLine_enabled = 0`
5. Commands
   `:IndentLinesToggle`

##### [auto-pairs](https://github.com/jiangmiao/auto-pairs)
1> Installation
`git clone git://github.com/jiangmiao/auto-pairs.git ~/.vim/bundle/auto-pairs`
2> Setting `~/.vimrc`
1. add `let g:AutoPairsFlyMode = 1` .vimrc to turn it on
Default Options:
```
let g:AutoPairsFlyMode = 0
let g:AutoPairsShortcutBackInsert = '<M-b>' #"Alt+b"
```
2. Shortcuts
System Shortcuts:
```
<CR>  : Insert new indented line after return if cursor in blank brackets or quotes.
<BS>  : Delete brackets in pair
<M-p> : Toggle Autopairs (g:AutoPairsShortcutToggle)
<M-e> : Fast Wrap (g:AutoPairsShortcutFastWrap)
<M-n> : Jump to next closed pair (g:AutoPairsShortcutJump)
<M-b> : BackInsert (g:AutoPairsShortcutBackInsert)
If <M-p> <M-e> or <M-n> conflict with another keys or want to bind to another keys, add
let g:AutoPairsShortcutToggle = '<another key>'
to .vimrc, if the key is empty string '', then the shortcut will be disabled.
```

##### [nerdcommenter](https://github.com/scrooloose/nerdcommenter)
1> Installation
Launch vim and `:PluginInstall 'scrooloose/nerdcommenter'`
2> Setting
```
" Add spaces after comment delimiters by default
let g:NERDSpaceDelims = 1

" Use compact syntax for prettified multi-line comments
let g:NERDCompactSexyComs = 1

" Align line-wise comment delimiters flush left instead of following code indentation
let g:NERDDefaultAlign = 'left'

" Set a language to use its alternate delimiters by default
let g:NERDAltDelims_java = 1

" Add your own custom formats or override the defaults
let g:NERDCustomDelimiters = { 'c': { 'left': '/**','right': '*/' } }

" Allow commenting and inverting empty lines (useful when commenting a region)
let g:NERDCommentEmptyLines = 1

" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace = 1
```
##### [vim-easymotion](https://github.com/easymotion/vim-easymotion)
1> Installation
`git clone https://github.com/easymotion/vim-easymotion ~/.vim/bundle/vim-easymotion`
add `Plugin 'easymotion/vim-easymotion'` to .vimrc
2> Setting
```
let g:EasyMotion_smartcase = 1
"let g:EasyMotion_startofline = 0 " keep cursor colum when JK motion
map <Leader><leader>h <Plug>(easymotion-linebackward)
map <Leader><Leader>j <Plug>(easymotion-j)
map <Leader><Leader>k <Plug>(easymotion-k)
map <Leader><leader>l <Plug>(easymotion-lineforward)
" 重复上一次操作, 类似repeat插件, 很强大
map <Leader><leader>. <Plug>(easymotion-repeat)
```
跳转到当前光标前后的位置(w/b); 搜索跳转(s); 行级跳转(jk); 行内跳转(hl); 重复上一次动作(.)

##### [python-mode](https://github.com/python-mode/python-mode)
1> Installation
`git clone https://github.com/python-mode/python-mode.git ~/.vim/bundle/vim-easymotion`
add `Plugin 'python-mode/python-mode'` to .vimrc
2> Setting
```
" Override go-to.definition key shortcut to Ctrl-]
let g:pymode_rope_goto_definition_bind = "<C-]>"

" Override run current python file key shortcut to Ctrl-Shift-e
let g:pymode_run_bind = "<C-S-e>"

" Override view python doc key shortcut to Ctrl-Shift-d
let g:pymode_doc_bind = "<C-S-d>"
```
1. Press K for documentation
2. Run python code: Press <leader>r (,r) to run python code
3. Pylint: By default, pylint check your code every save.
4. 
 

#### [YouCompleteMe](https://github.com/Valloric/YouCompleteMe) ★★★★★
Dcumentation: [YouCompleteMe](http://valloric.github.io/YouCompleteMe/)
##### Installation
1> Python: using anaconda2
version: [Anaconda2-4.4.0-Linux-x86_64](https://repo.continuum.io/archive/Anaconda2-4.4.0-Linux-x86_64.sh); (内带：python2.7.13)
> 本来想安装Anaconda2-5.0.0版本，一直提示`/lib64/libc.so.6: version 'GLIBC_2.7' not found`。 本地安装glibc-2.7后，设置LD_LIBRARY_PATH后Anaconda2-5.0.0-Linux-x86_64.sh还是出现相同错误，添加LDFKLAGS参数也无效。 修改.sh文件后，无法运行。 最终放弃，选择4.4.0版本。

```
sh Anaconda2-4.4.0-Linux-x86_64.sh
<Enter>
<enter> ...
yes
/home/wuxiaolong/tools/anaconda2
```
设置参数： `~/.bashrc`
```
export PATH="/home/wuxiaolong/tools/anaconda2/bin:$PATH"
export PYTHONPATH="/home/wuxiaolong/tools/anaconda2/lib/python2.7/site-packages"
export pythonpath="/home/wuxiaolong/tools/anaconda2/bin/"
```

2> cmake: 原版本太低，本地安装覆盖原版本
version: [cmake-3.9.3](https://cmake.org/files/v3.9/cmake-3.9.3.tar.gz)
```
cd ~/software
wget https://cmake.org/files/v3.9/cmake-3.9.3.tar.gz
tar -xzvf cmake-3.9.3.tar.gz
cd cmake-3.9.3
./configue --prefix=/home/wuxiaolong/tools/cmake
make && make install
echo "export PATH=/home/wuxiaolong/tools/cmake/bin:$PATH" >> ~/.bashrc
```
> 同理安装curl-7.55.1

3> 原c++编译器器不支持C++ 11: `Your C++ compiler does NOT support C++11`
本地安装gcc，version: [gcc-4.8.2](http://gcc.skazkaforyou.com/releases/gcc-4.8.2/gcc-4.8.2.tar.gz)
参考： [Installing GCC without Root Privileges](http://luiarthur.github.io/gccinstall); [linux下安装或升级GCC4.8，以支持C++11标准](http://www.cnblogs.com/lizhenghn/p/3550996.html)
```
cd ~/software
wget http://gcc.skazkaforyou.com/releases/gcc-4.8.2/gcc-4.8.2.tar.gz
tar -xzvf gcc-4.8.2.tar.gz
cd gcc-4.8.2
./contrib/download_prerequisites
# --enable-languages表示你要让你的gcc支持那些语言[c,c++,fortran,go]，--disable-multilib不生成编译为其他平台可执行代码的交叉编译器。--disable-checking生成的编译器在编译过程中不做额外检查，也可以使用--enable-checking=xxx来增加一些检查
./configure --enable-checking=release --enable-languages=c,c++ --disable-multilib --prefix=/home/wuxiaolong/tools/gcc-4.8.2
make && make install # takes many hours; usa make -j 4 instead
```
设置参数： `~/.bashrc`
```
export PATH=~/tools/gcc-4.8.2/bin:$PATH
export LD_LIBRARY_PATH=~/gcc-4.8.2/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=~/gcc-4.8.2/lib64:$LD_LIBRARY_PATH
```
或者在运行命令前指定： CXX=~/tools/gcc-4.8.2/bin/c++
R 编译器指定： add `CXX=~/tools/gcc-4.8.2/bin/g++` to `~/.R/Makevars` (create the file if not exits)

4> YouCompleteMe <2017-9-28>
注： 不涉及C家族的语义化补全
1) use Vundle installation: not complete
`Plugin 'Valloric/YouCompleteMe'`
2）git
```
cd ~/.vim/bundle/
git clone --recursive https://github.com/Valloric/YouCompleteMe.git
cd YouCopleteMe
git submodule update --init --recursive
```
3) 编译构建ycm_core, 这里对cmake，gcc, python-dev有要求
`CXX=~/tools/gcc-4.8.2/bin/c++ ./install.py` (不支持C家族)
以下支持C家族，可是有问题
`CXX=~/tools/gcc-4.8.2/bin/c++ ./install.py --clang-completer` or `CXX=~/tools/gcc-4.8.2/bin/c++ ./install.py --clang-completer`
`~/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/CMakeLists.txt` 提供的 `tar -xJf` 无法正确解压 `clang+llvm-5.0.0-linux-x86_64-ubuntu14.04.tar.xz`。 修改`CMakeLists.txt`中的命令。
修改后，编译成功，启动不了。
vim, `:YcmToggleLogs ycmd[tab]`, to check the log file
- 错误1： `ImportError: libtinfo.so.5: cannot open shared object file: No such file or directory`
- 原因: `readelf -d ~/.vim/bundle/YouCompleteMe/third_party/ycmd/libclang.so.5`
- Solution:
```
mkdir ~/tools/lib64
cd ~/tools/lib64
ln -s /usr/lib64/libncurses.so.5 /home/wuxiaolong/tools/lib64/libtinfo.so.5
echo "export LD_LIBRARY_PATH=~/tools/lib64:$LD_LIBRARY_PATH"
```
- Error2: ImportError: /lib64/libc.so.6: version `GLIBC_2.15' not found (required by /home/wuxiaolong/.vim/bundle/YouCompleteMe/third_party/ycmd/libclang.so.5)
- Thoughts: install glibc-2.15
- Error3: `configure: error: assembler too old, .cfi_personality support missing`
- Thoughts: install binutils-2.29.1.tar.gz
- Couldn't be solved
- Final solution：
- conda 虚拟环境安装glibc-2.18; 无法加LD_LIBRARY_PATH；一加就崩溃
- 补救"LD_PRELOAD=/lib64/libc.so.6 + 命令"
- 查看libc.so.6支持版本: /lib64/libc.so.6 | grep "GLIBC"
- conda install -c rmg glibc
- 最终解决的方法：安装devtoolset-2；
- http://www.cnblogs.com/zhangyanpei/p/6277079.html
4> 启动vim，遇到: `Error detected while processing function youcompleteme#Enable[5]..<SNR>52_SetUpPython`
网上说要重编译vim，半信半疑的进行vim重装：
```
tar -xjvf vim-8.0.tar.bz2
cd vim80
make clean
./configure --with-features=huge --enable-rubyinterp \
	--enable-pythoninterp \
	--with-python-config-dir=/home/wuxiaolong/tools/anaconda2/lib/python2.7/config/ \
	--enable-python3interp \
	--enable-perlinterp --enable-gui=gtk2 --enable-cscope --enable-luainterp --enable-multibyte \
	--prefix=/home/wuxiaolong/tools/vim80
make && make install
```
设置参数: add `export PATH=~/tools/vim80/bin:$PATH` to `~/.bashrc`
**Surprising!!!** It works. 

5> YouCompleteMe 配置
1) 设置自动加载`.ycm_extra_conf.py`
`cp ~/.vim/bundle/YouCompleteMe/third_party/ycmd/examples/.ycm_extra_conf.py ~/.vim/`
add to `~/.vimrc`
```
let g:ycm_server_python_interpreter='/home/wuxiaolong/tools/anaconda2/bin/python'
104 let g:ycm_path_to_python_interpreter='/home/wuxiaolong/tools/anaconda2/bin/python'
105 let g:ycm_global_ycm_extra_conf='~/.vim/.ycm_extra_conf.py'
```
2) 其他配置
```
"打开vim时不再询问是否加载ycm_extra_conf.py配置"
let g:ycm_confirm_extra_conf=0
set completeopt=longest,menu
"是否开启语义补全"
let g:ycm_seed_identifiers_with_syntax=1
"是否在注释中也开启补全"
let g:ycm_complete_in_comments=1
let g:ycm_collect_identifiers_from_comments_and_strings = 0
"开始补全的字符数"
let g:ycm_min_num_of_chars_for_completion=2
"补全后自动关机预览窗口"
let g:ycm_autoclose_preview_window_after_completion=1
" 禁止缓存匹配项,每次都重新生成匹配项"
let g:ycm_cache_omnifunc=0
"字符串中也开启补全"
let g:ycm_complete_in_strings = 1
"离开插入模式后自动关闭预览窗口"
autocmd InsertLeave * if pumvisible() == 0|pclose|endif
"回车即选中当前项"
inoremap <expr> <CR>       pumvisible() ? '<C-y>' : '\<CR>'     
"上下左右键行为"
inoremap <expr> <Down>     pumvisible() ? '\<C-n>' : '\<Down>'
inoremap <expr> <Up>       pumvisible() ? '\<C-p>' : '\<Up>'
inoremap <expr> <PageDown> pumvisible() ? '\<PageDown>\<C-p>\<C-n>' : '\<PageDown>'
inoremap <expr> <PageUp>   pumvisible() ? '\<PageUp>\<C-p>\<C-n>' : '\<PageUp>'
```
### Unfinished: color scheme displaying incorrect

### Others about Vim
#### 开启文件类型检测
1. `:filetype`命令查看你的文件类型检测功能有没有打开。
   `filetype plugin indent on`等价于： `filetype on`, `file plugin on`, `filetype indent on`.
1） filetype
> `filetype on`命令打开文件类型检测功能，相当于文件类型检测功能的开关。执行时，vim实际上执行的是$vimRUNTIME/filetype.vim脚本，该脚本使用自动命令（autocmd）根据文件名来判断文件的类型，如果无法判断，又会调用$vimRUNTIME/scripts.vim根据文件的内容设置文件类型。 未被正确检测出来的，可以通过`set filetype=c`手动设置。
> `modelines`模式行，用于文件类型检测，默认5行。
> 文件类型检测，用于初始化选项值、定义不同建绑定等，包括注释格式、缩进格式、关键字等等。

2） filetype plugin
> `filetype plugin on`，允许vim加载文件类型插件。 开启时，vim会根据检测到的文件类型，在runtimepath中搜索该类型的所有插件，并执行它们。 它实际上是执行$vimRUNTIME/ftplugin.vim脚本，该脚本中设置了自动命令，在runtimepath中搜索文件类型插件。
> runtimepath的定义在不同的系统上不一样，对UNIX系统来说，这些路径包括：$HOME/.vim、$vim/vimfiles、$vimRUNTIME、$vim/vimfiles/after、$HOME/.vim/after。
> 举一个例子，当我们对一个c类型的文件打开”filetype plugin on”时，它会在上述这几个目录的ftplugin子目录中搜索所有名为c.vim、c_\*.vim，和c/\*.vim的脚本，并执行它们。在搜索时，它按目录在runtimepath中出现的顺序进行搜索。缺省的，它会执行$vimRUNTIME/ftplugin/c.vim，在这个脚本里，会设置c语言的注释格式、智能补全函数等等。

3） filetype indent
> `filetype indent on`，允许vim为不同类型的文件定义不同的缩进格式。这条命令通过$vimRUNTIME/indent.vim脚本来完成加载。 该脚本的自动命令在runtimepath的indent子目录中搜索缩进设置。 对此类型文件来说， 它只是打开了cindent选项。
