---
title: "NeoVim 打造 Golang 开发环境"
subtitle: ""
summary: ""
authors: []
tags: ["Golang","vim"]
categories: []
date: 2020-07-25T17:42:23+08:00
lastmod: 2020-07-25T17:42:23+08:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: "Smart"
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

###  为什么想使用vim？

之前一直使用jetbrains家族的各个IDE,用起来也非常的方便，前几天刚好激活码也过期了，懒得找激活，就想趁着这股劲就想把vim搞起来，其实也试过目前挺火的 vacode 但是使用起来 总是感觉差点什么，常用快捷键也和goland的变化太大，又得重新去记快捷键，有点学习成本还麻烦，既然学习那为什么不学vim呢？学习成本越大的，往往后面收益越大，而且vim 的很多快捷键和 shell，git下的快捷键是通用的。网上也从未停止过vim与其他编辑器或IDE的讨论,例如 最早的eclipse 和 vim 那个更强大，后来的sublime和vim的比较等等，这两年vscode又火了，都拿来和vim来作比较，没准过个几年某乎又会出现 xxx编辑器和vim那个更强大！所以学会学好使用vim还是很有必要的。好多资深大佬也都强烈推荐学习，毕竟都是过来人。
<!--more-->

* 减少编写代码过程中鼠标使用，效率问题另说，
* 避免出个新编辑器，自己盲目跟风
* 体会vim设计理念，提升效率
* 前期成本越大，后期收益越高 
* 有时候必须在服务器上写代码
* 不用到处找激活码，虽然也不难找
* IDE的很多功能完全没用到，反而臃肿
* 前辈们推荐
* xxxx
	
	
### NeoVim及插件管理安装

环境：MacOs 10.14.4

#### NeoVim

关于neovim是什么，为什么有了vim,又出来个neovim? 可查看[官方文档](https://github.com/neovim/neovim/wiki/Installing-Neovim)，[NeoVim 科普，21世纪的Vim](https://zhuanlan.zhihu.com/p/21364426)，
[如何对比评价vim-8和neovim？](https://www.zhihu.com/question/54663891)，具体使用根据个人喜好，没必要纠结，我使用的是Neovim，Mac下使用homebrew安装neovim，如果使用其他操作系统阅读官方文档安装即可：

```bash
brew install neovim 
pip3 install --user --upgrade pynvim 
```

neovim可以直接使用vim的配置文件，基本可以无缝切换，但是我为了和vim区分开使用的neovim默认的配置文件，默认地址在 ~/.config/nvim/init.vim 这个文件需要自己创建，

#### 插件管理

插件管理使用的是[vim-plug](https://github.com/junegunn/vim-plug)，
Unix环境下Neovim安装：

```bash
curl -fLo ~/.local/share/nvim/site/autoload/plug.vim --create-dirs \
    https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
```

之后打开init.vim ，添加：

```bash
call plug#begin('~/.vim/plugged')
call plug#end()
```

begin括号里面的意思就是之后安装的插件全放在这个目录下，可根据个人情况修改，

Reload .vimrc and `:PlugInstall` to install plugins，其他命令使用，摘自官网：

| Command                           | Description                                                  |
| --------------------------------- | ------------------------------------------------------------ |
| PlugInstall [name ...] [#threads] | Install plugins                                              |
| PlugUpdate [name ...] [#threads]  | Install or update plugins                                    |
| PlugClean[!]                      | Remove unlisted plugins (bang version will clean without prompt) |
| PlugUpgrade                       | Upgrade vim-plug itself                                      |
| PlugStatus                        | Check the status of plugins                                  |
| PlugDiff                          | Examine changes from the previous update and the pending changes |
| PlugSnapshot[!] [output path]     | Generate script for restoring the current snapshot of the plugins |

### 插件配置介绍

一般来说我们平常在IDE会用到的基本功能，

1. 代码提示，自动补全
2. 好看的主题配色
2. 代码跳转，前进，后退
3. 错误提示
4. 目录管理
5. 代码格式化,包的自动导入
6. 快速在项目中搜索函数
7. 文件function tree 

IDE也有一些其他比较强大的功能，比如说Goland中的dataBase功能，就我个人而言平时基本不会使用到，终端连接或者专门的数据库客户端都可以满足我的需求，而且体验更好更方便，IDE大而全的功能一方面使用方便，另一方面对于基本不会用到它的功能的人来说却是臃肿。vim的话可定制化非常高，各种各样插件五花八门，首先没有必要把大量的精力浪费在插件配置，vim本身设计才是最重要的。我们只需要安装基本的插件就可以达到IDE中代码编写体验，安装自己必须必要的插件即可，没必要把vim也搞得臃肿，毕竟它只是编辑器。下面是我最近使用安装的插件，也有一些其他的插件，安装了没什么用就删除了，


#### [nerdtree](https://github.com/vim-nerdtree/nerdtree)

一个老牌的项目目录管理插件，

```vimrc
Plug 'scrooloose/nerdtree'

nnoremap <leader>v :NERDTreeFind<cr>
nnoremap <leader>g :NERDTreeToggle<cr>
```
这里配置两个快捷键 ,g 和 ,v ，,g的话可以打开侧边栏目录，基础使用j k进行上下选择，o或回车打开文件目录，<C-w> w可以切换窗口， 再次使用 ,g 可关闭侧边栏目录， 当我们在编写代码是想快速跳到当前文件在目录位置时就可以 ,v 快捷键，其他的一些详细操作可在目录栏使用 ？ 查看官方文档。

####  [airline](https://github.com/vim-airline/vim-airline)

状态栏美化插件，

```bash
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
```
不需要做其他配置，与其他插件集成度非常好

#### [tagbar](https://github.com/majutsushi/tagbar)

类似于goland funtion tree
tagbar使用Ctags对文件进行解析，所以要正常使用必须安装依赖：

```bash
brew tap universal-ctags/universal-ctags
brew install --HEAD universal-ctags
```
安装：

```bash
Plug 'majutsushi/tagbar'
nnoremap <leader>t :TagbarToggle<cr>
```
快捷键 ,t 可以打开/关闭右侧 tarbar窗口

#### [vim-startify](https://github.com/mhinz/vim-startify)

vim开屏页美化插件，可以记录最近编辑的文件，使用对应数字编号就可以快速打开文件，使用起来非常方便。

```bash
Plug 'mhinz/vim-startify'
```

#### [molokai](https://github.com/fatih/molokai)

主题的话根据个人喜好选择，我之前在phpstrom,goland都是使用这个配色，

```bash
Plug 'fatih/molokai'

syntax enable
set t_Co=256
let g:rehash256 = 1
let g:molokai_original = 1
colorscheme molokai
```

#### [vim-go](https://github.com/fatih/vim-go)

vim-go 提供了大量的功能，

```bash
Plug 'fatih/vim-go'
```
安装完vim-go之后，打开nvim 使用 :GoInstallBinaries 安装依赖包，需要配置在环境变量PATH配置GOBIN。国情原因，有些包地址是被墙的，可以使用国内代理。

```bash
export GOPROXY=https://goproxy.io
```
默认打开go文件，语法高亮是不全的，也没有像vscode 一样保存自动格式化，所以还需做简单配置：

```bash
" vim-go
let g:go_fmt_command = 'goimports'
let g:go_autodetect_gopath = 1
" let g:go_bin_path = '$GOBIN'

let g:go_highlight_types = 1
let g:go_highlight_fields = 1
let g:go_highlight_functions = 1
let g:go_highlight_function_calls = 1
let g:go_highlight_extra_types = 1
let g:go_highlight_generate_tags = 1

" Open :GoDeclsDir with ctrl-g
nmap <C-g> :GoDeclsDir<cr>
imap <C-g> <esc>:<C-u>GoDeclsDir<cr>

augroup go
  autocmd!
  autocmd BufNewFile,BufRead *.go setlocal noexpandtab tabstop=4 shiftwidth=4
augroup END
```
这里参考vim-go 作者的vim配置，[vim-go-tutorial](https://github.com/webmrsong/vim-go-tutorial)，配置非常精简，非常值得借鉴学习。配置完成之后就可以使用vim愉快的编写go代码了。
用的比较多的，gd 跳转到代码定义 < C-i > < C-o > 后退/前进，保存代码就会自动格式化。其他的使用方法看文档就行。

#### [coc.nvim](https://github.com/neoclide/coc.nvim)

其实安装完vim-go就已经有了代码补全，但是体验不是很好。刚开始也使用过 [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe) 但是被劝退了，编译复杂 依赖多 有的包网不好还下载不下来，虽然最后勉强成功安装，然而没太看懂配置，就放弃了。最后了解到国人开发的 coc.nvim，使用 LSP,异步加载，配置方便，开箱即用！ 在nvim下也有像IDE一样的悬浮窗功能。各个功能使用下来和IDE代码补全不相上下。

```bash
Plug 'neoclide/coc.nvim', {'branch': 'release'}
```
coc.nvim 使用 :CocConfig 打开配置文件，默认路径 ~/.config/nvim/coc-settings.json ,这里只介绍golang的配置，配置文件添加：

```json
{
    "languageserver": {
        "golang": {
            "command": "gopls",
            "rootPatterns": [
                "go.mod"
            ],
            "filetypes": [
                "go"
            ]
        }
    },
    "suggest.noselect": false,
    "coc.preferences.diagnostic.displayByAle": true,
    "suggest.floatEnable": true
}
```

init.vim 添加配置，也可不配置不常用到：

```bash
" Remap keys for gotos
" nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gm <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)
inoremap <expr> <cr> pumvisible() ? "\<C-y>" : "\<C-g>u\<CR>"

nnoremap <silent> K :call <SID>show_documentation()<CR>
function! s:show_documentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  else
    call CocAction('doHover')
  endif
endfunction
```
正确配置之后，就可以使用代码补全了 例如我们输入 fmt. 就会提示fmt包中的方法，默认选择第一个，使用< C-n > < C-p > 上下选车，回车选择，nvim下可以使用悬浮窗功能。

#### [ale](https://github.com/dense-analysis/ale)

其实coc.nvim已经自带错误处理，但是为了让专业的插件干专业的事，所以这里使用ale来处理代码错误，需要在coc.nvim配置文件中添加 "coc.preferences.diagnostic.displayByAle": true , 否则插件之间会有冲突。ale是一个异步的错误处理的插件，支持各种语言，配置相对简单。

```bash
Plug 'dense-analysis/ale'

" ale-setting {{{
let g:ale_set_highlights = 1
let g:ale_set_quickfix = 1
"自定义error和warning图标
let g:ale_sign_error = '✗'
let g:ale_sign_warning = '⚡'
"在vim自带的状态栏中整合ale
let g:ale_statusline_format = ['✗ %d', '⚡ %d', '✔ OK']
"显示Linter名称,出错或警告等相关信息
let g:ale_echo_msg_error_str = 'E'
let g:ale_echo_msg_warning_str = 'W'
let g:ale_echo_msg_format = '[%linter%] %s [%severity%]'
"打开文件时不进行检查
let g:ale_lint_on_enter = 1

"普通模式下，sp前往上一个错误或警告，sn前往下一个错误或警告
nmap sp <Plug>(ale_previous_wrap)
nmap sn <Plug>(ale_next_wrap)
"<Leader>s触发/关闭语法检查
" nmap <Leader>l :ALEToggle<CR>
"<Leader>d查看错误或警告的详细信息
nmap <Leader>d :ALEDetail<CR>
let g:ale_linters = {
    \ 'go': ['golint', 'go vet', 'go fmt'],
    \ }
```

使用 sn sp 就可以在错误之间进行快速跳转，修复代码，之后如果要添加其他的语法处理，只需要做简单修改即可:

```bash
let g:ale_linters = {
    \ 'go': ['golint', 'go vet', 'go fmt'],
    \ 'python': ['flake8', 'pylint'],
    \ 'rust': ['cargo'],
    \ }
```

####  [vim-commentary](https://github.com/tpope/vim-commentary)

快速注释插件，支持各种语言。

```bash
Plug 'tpope/vim-commentary'
```
使用原生快捷键，只需记住两个命令即可。

gcc 快速注释/取消注释光标所在行，

gc  注释/取消选中区域，

#### [vim-easymotion](https://github.com/easymotion/vim-easymotion)

一个页面可视化区域快速移动光标插件，不要被官网复杂的配置吓到，配置一个快捷键便可使用。

```bash
Plug 'easymotion/vim-easymotion'

nmap ss <Plug>(easymotion-s2)
```
一般情况下，如果需要编辑某一行的某个单词，首先得jjjj 或者 行号+gg 才能将光标移动到目标所在行，再w w或者f x跳到这个单词，这样移动光标特别麻烦，使用easymotion的话，首先在normal模式下，使用快捷键 ss 再输入目标单词的2字母，easymotion就会在目标单词随机高亮生一个字母，再次按下这个字母，即可快速移动光标到目标单词。注意是当前屏幕显示区域，不是当前整个文件。

#### [vim-surround](https://github.com/tpope/vim-surround)

如果我们想快速 替换/添加/删除 单词两边的" }等，默认情况下插入模式替换非常麻烦，vim-surround使用非常简单的命令便可快速完成修改，[官方文档](https://github.com/tpope/vim-surround)有非常详细的示例，这里不做详细介绍。

```bash
Plug 'tpope/vim-surround'
```

#### [auto-pairs](https://github.com/jiangmiao/auto-pairs)

默认情况下vim 输入括号等都不是成对输入的，成对输入后回车也没有自动缩进，编写代码比较麻烦，auto-pairs完美解决了这个问题。

```bash
Plug 'jiangmiao/auto-pairs'
```
#### [vim-snippets](https://github.com/honza/vim-snippets)

快速生成代码块插件。

```bash
Plug 'SirVer/ultisnips'
Plug 'honza/vim-snippets'
```
默认快捷键<tab>，例如在编写go代码，有大量重复的代码编写操作，手动编写会稍微影响效率，使用vim-snippets的话，假如我们想编写一个for range 循环结构, 只需输入 forr 在使用 < tab > 就可生成如下代码：

```go
	for k, v := range  {
	}
```

需要注意 < tab > 快捷键有可能会和其他代码补全插件产生冲突

#### [fzf.vim](https://github.com/junegunn/fzf.vim)

模糊匹配查找插件，类似的插件还有， [CtrlP](https://github.com/kien/ctrlp.vim)，[LeaderF](https://github.com/Yggdroot/LeaderF) 

fzf的安装方式有两种，如果你在系统中也想使用fzf，先 

```bash
brew install fzf
```
init.vim 添加：

```bash
Plug '/usr/local/opt/fzf'
Plug 'junegunn/fzf.vim'
```

使用 :Ag 命令的还需安装：

```bash
brew install the_silver_searcher
```
只是在 vim 中使用fzf,使用下面:

```bash
Plug 'junegunn/fzf', { 'dir': '~/.fzf', 'do': './install --all' }
Plug 'junegunn/fzf.vim'
```
快捷键配置：

```bash
map <leader>f :Files<CR>
map <leader>b :Buffers<CR>
let g:fzf_action = { 'ctrl-e': 'edit' }
" 用 leader+ag 搜索当前 cursor 下单词 see: https://github.com/junegunn/fzf.vim/issues/50
nnoremap <silent> <Leader>ag :Ag <C-R><C-W><CR>
```
fzf 已和原生 :Files :Buffers 集成，使用 ,f 就可快速搜索项目中的文件，,b 打开buffer列表快速切换文件，使用< C-n > < C-p > 上下选择，<c-r> 选中，上下选择一般情况下在其他插件中都是通用的，所以尽可能不要所以修改快捷键。

如果说我们要在项目中搜索某个函数或是某个字符串，commands模式 :Ag 目标字符串 就可在项目中快速查找，另一种是光标已经在目标单词上，我们想要查找其他地方使用情况，光标停留在目标单词上，使用 ,ag 就可完成快速查找。

### 最终配置文件

#### 效果图
![最终效果](./nvim.png)

#### coc-settings.json

```json
{
    "languageserver": {
        "golang": {
            "command": "gopls",
            "rootPatterns": [
                "go.mod"
            ],
            "filetypes": [
                "go"
            ]
        }
    },
    "suggest.noselect": false,
    "coc.preferences.diagnostic.displayByAle": true,
    "suggest.floatEnable": true
}
```
#### init.vim

```bash
set number
filetype plugin indent on
set showmatch
set shiftwidth =4
set tabstop=4
set encoding=utf-8
set hlsearch
set autoindent
set clipboard=unnamed
set nocursorcolumn              " Do not highlight column (speeds up highlighting)
set nocursorline 
set backspace=2

call plug#begin('~/.vim/plugged')
"""""""""""""""""""""
"      Plugins      "
"""""""""""""""""""""
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
Plug 'fatih/vim-go'
Plug 'tpope/vim-surround'
Plug 'mhinz/vim-startify'
Plug 'scrooloose/nerdtree'
Plug 'tpope/vim-commentary'
Plug 'neoclide/coc.nvim', {'branch': 'release'}
Plug 'dense-analysis/ale'
Plug 'majutsushi/tagbar'
Plug 'easymotion/vim-easymotion'
Plug '/usr/local/opt/fzf'
Plug 'jiangmiao/auto-pairs'
Plug 'junegunn/gv.vim'
Plug 'junegunn/fzf.vim'
Plug 'SirVer/ultisnips'
Plug 'honza/vim-snippets'
Plug 'fatih/molokai'
Plug 'tpope/vim-fugitive'
call plug#end()

" Enable to copy to clipboard for operations like yank, delete, change and put
" http://stackoverflow.com/questions/20186975/vim-mac-how-to-copy-to-clipboard-without-pbcopy
if has('unnamedplus')
  set clipboard^=unnamed
  set clipboard^=unnamedplus
endif

" This enables us to undo files even if you exit Vim.
if has('persistent_undo')
  set undofile
  set undodir=~/.config/vim/tmp/undo//
endif

" Colorscheme
syntax enable
set t_Co=256
let g:rehash256 = 1
let g:molokai_original = 1
colorscheme molokai

" Set leader shortcut to a comma ','. By default it's the backslash
let mapleader = ','


" vim-go
let g:go_fmt_command = 'goimports'
let g:go_autodetect_gopath = 1
" let g:go_bin_path = '$GOBIN'

let g:go_highlight_types = 1
let g:go_highlight_fields = 1
let g:go_highlight_functions = 1
let g:go_highlight_function_calls = 1
let g:go_highlight_extra_types = 1
let g:go_highlight_generate_tags = 1

" Open :GoDeclsDir with ctrl-g
nmap <C-g> :GoDeclsDir<cr>
imap <C-g> <esc>:<C-u>GoDeclsDir<cr>

augroup go
  autocmd!
  autocmd BufNewFile,BufRead *.go setlocal noexpandtab tabstop=4 shiftwidth=4
augroup END

" build_go_files is a custom function that builds or compiles the test file.
" It calls :GoBuild if its a Go file, or :GoTestCompile if it's a test file
function! s:build_go_files()
  let l:file = expand('%')
  if l:file =~# '^\f\+_test\.go$'
    call go#test#Test(0, 1)
  elseif l:file =~# '^\f\+\.go$'
    call go#cmd#Build(0)
  endif
endfunction

map <leader>f :Files<CR>
map <leader>b :Buffers<CR>
let g:fzf_action = { 'ctrl-e': 'edit' }
" 用 leader+ag 搜索当前 cursor 下单词 see: https://github.com/junegunn/fzf.vim/issues/50
nnoremap <silent> <Leader>ag :Ag <C-R><C-W><CR>

nnoremap <leader>v :NERDTreeFind<cr>
nnoremap <leader>g :NERDTreeToggle<cr>

nnoremap <leader>t :TagbarToggle<cr>
nmap ss <Plug>(easymotion-s2)

" ale-setting {{{
let g:ale_set_highlights = 1
let g:ale_set_quickfix = 1
"自定义error和warning图标
let g:ale_sign_error = '✗'
let g:ale_sign_warning = '⚡'
"在vim自带的状态栏中整合ale
let g:ale_statusline_format = ['✗ %d', '⚡ %d', '✔ OK']
"显示Linter名称,出错或警告等相关信息
let g:ale_echo_msg_error_str = 'E'
let g:ale_echo_msg_warning_str = 'W'
let g:ale_echo_msg_format = '[%linter%] %s [%severity%]'
"打开文件时不进行检查
let g:ale_lint_on_enter = 1

"普通模式下，sp前往上一个错误或警告，sn前往下一个错误或警告
nmap sp <Plug>(ale_previous_wrap)
nmap sn <Plug>(ale_next_wrap)
"<Leader>s触发/关闭语法检查
" nmap <Leader>l :ALEToggle<CR>
"<Leader>d查看错误或警告的详细信息
nmap <Leader>d :ALEDetail<CR>
let g:ale_linters = {
    \ 'go': ['golint', 'go vet', 'go fmt'],
	\ 'python': ['flake8', 'pylint'],
	\ 'rust': ['cargo'],
    \ }

" Remap keys for gotos
" nmap <silent> gd <Plug>(coc-definition)
nmap <silent> gy <Plug>(coc-type-definition)
nmap <silent> gm <Plug>(coc-implementation)
nmap <silent> gr <Plug>(coc-references)
inoremap <expr> <cr> pumvisible() ? "\<C-y>" : "\<C-g>u\<CR>"

nnoremap <silent> K :call <SID>show_documentation()<CR>
function! s:show_documentation()
  if (index(['vim','help'], &filetype) >= 0)
    execute 'h '.expand('<cword>')
  else
    call CocAction('doHover')
  endif
endfunction
" Jump to next error with Ctrl-n and previous error with Ctrl-m. Close the
" quickfix window with <leader>a
map <C-n> :cnext<CR>
map <C-m> :cprevious<CR>
nnoremap <leader>a :cclose<CR>

```
### 关于vim学习使用的一些看法

作为一个之前使用i a wq vim渣渣，使用之前先去慕课网找了两套视频，学习了大概10个小时，为什么看视频学习呢，最主要的就是想看看高手是怎么使用vim的，之后阅读学习了一下vim的官方文档，再看看博客，了解一下插件配置，就配置使用起来了，一周时间基本达到了之前使用IDE的水平，收获还是不错的，而且越往后收获也会更大。

刚开始的也使用过别人的vimrc配置文件，[SpaceVim](https://spacevim.org/) ,[spf13](https://vim.spf13.com/), 安装下来自己完全懵逼了，酷炫的界面，集成各种vim功能，介绍高大上，有些还乱改快捷键，插件复杂的和IDE一样，对于一个新手来说完全不会用，直接劝退，所以新手还是根据自己喜好动手一步一步配置最好，可以借鉴学习别人配置，但是不要拿来直接用，就像有的快捷键适合自己操作习惯才是最好的。

最近也折腾了不少的插件，浪费了不少时间，vim的可定制自由度非常高，但是完全没必要浪费时间在插件上面就行，vim自身才是最重要的，插件安装折腾下来感受就是：插件越少越好，各个插件的快捷键尽量越少越好。插件安装使用尽可能开箱即用，不需要太多的配置，必须知道自己安装每一个插件的用法，熟练掌握插件中快捷键的用途，尽可能使用vim命令技巧。

### vim学习资料

*  [优雅玩转Vim](https://www.imooc.com/learn/1049)
*  [玩转Vim 从放弃到爱不释手](https://www.imooc.com/learn/1129)
*  [Vim实用技巧](https://book.douban.com/subject/25869486/)
*  [Vim官方文档](http://vimcdoc.sourceforge.net/doc/help.html)

