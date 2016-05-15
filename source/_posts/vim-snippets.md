title: vim-snippets
date: 2015-09-14 22:06:59
tags:
- vim
---
# 背景
自我感觉作为一个程序员，还算得上一个比较有效率的人，最近拜读*程序人生*的技术专栏[程序员效率指南](http://mp.weixin.qq.com/s?__biz=MzA3NDM0ODQwMw==&mid=206041450&idx=1&sn=3982c8cc45d7c47f0fbc19fe8371490f&scene=0#rd), 不由得自省了一把，其他还马马虎虎，作一个VIM用了2，3年的programmer，(虽然偶尔也用WebStrom和InteliJ, 毕竟vim是自己的主力editor)，尴尬了，尽管也有neocomplete等一干plugin助力，居然没有用过snippets....亡羊补牢吧。
<!-- more -->
# Snippets Plugin for Vimer
1. [SirVer/ultisnips](https://github.com/SirVer/ultisnips)
2. [honza/vim-snippets](https://github.com/honza/vim-snippets)
ultisnips是snippets enginne，要配合honza/vim-snippets一起用。(画外音：“我知道，要两粒在一起吃才好")

## 安装
github上已经有的就不写了，如果你和我一样用的不是vundle, 而是pathogen的话，
```bash
$ cd ~/.vim/bundle
$ git clone https://github.com/SirVer/ultisnips
$ git clone https://github.com/honza/vim-snippets
```

## 配置
```vim
" Trigger configuration. Do not use <tab> if you use https://github.com/Valloric/YouCompleteMe.
let g:UltiSnipsExpandTrigger="<tab>"
let g:UltiSnipsJumpForwardTrigger="<c-j>"
let g:UltiSnipsJumpBackwardTrigger="<c-k>"

" If you want :UltiSnipsEdit to split your window.
let g:UltiSnipsEditSplit="vertical"
```
`UltiSnipsExpandTrigger`较好理解，就是触发展开snippet的热键, `UltiSnipsJumpForwardTrigger`和`UltiSnipsJumpBackwardTrigger`是控制tabstop的热键，后面有详述。

## 如何自定义
我自己写JavaScript会比较多，根据https://github.com/honza/vim-snippets/tree/master/snippets/javascript，
除了通用的[JavaScript Snippets](https://github.com/honza/vim-snippets/blob/master/snippets/javascript/javascript.snippets)之外，还有jquery, d3, requirejs, node专有的snippets, 基本够用，当然总有一天你会想自己定制snippet...

### Snip Search Path
snippet目录必须是｀runtimepath`的子目录，可以在下面配置中自己修改。
```
   let g:UltiSnipsSnippetDirectories=["UltiSnips"]
```
UltiSnips会根据以下策略去找snippet文件：
```
  ft.snippets, ft_*.snippets, or ft/* 
```
其中`ft`是当前文档的filetype。

### 语法
非常简单，`snippet`开始，`endsnippet`结束。定义格式：`snippet tab_trigger [ "description" [ options ] ]`
```
    snippet if "if ... then (if)"
    if ${2:[[ ${1:condition} ]]}; then
            ${0:#statements}
    fi
    endsnippet
```
就是说你输入`if`的话，`if...fi`整个block会被插入。

### Visual Placeholder
Snippets中特殊的placeholder: `${VISUAL}`， visual mode 中选中的内容会被替换进来。触发如下：
选中-> `<tab>` -> tag_trigger -> `<tab>`


### Interpolation
snippets中可以使用shellcode, vimscript, python等，用    `(backtick)包起来就行。

### Tabstop 和 Placeholder
这部分保证了固定结构的snippets中可以输入可变的部分，tabstop用来控制光标在几个变量间跳转。
```
snippet letter
Dear $1,
$0
Yours sincerely,
$2
endsnippet
```
输入`letter`后按`<tab>`, snippet会被插入，然后`<c-j>`,`<c-k>`就可以在`$1` `$2`之间跳转，注意`$0`是最后一个位置，到了`$0`以后就不能再跳转了，往回跳也不行。

## 其他
请参考https://github.com/SirVer/ultisnips/blob/master/doc/UltiSnips.txt
