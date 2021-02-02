---
title: "vim-latex配置使用说明"
date: 2014-11-29T10:11:52+08:00
tags: ["vim", "latex"]
categories: ["Linux"]
toc: true
---

## 前提
当然是安装了texlive 2014了.
## 安装
YADR的Vim插件中没有latex支持,所以得自己安装插件.一直用vim-latex suite,所以就安装了.

```zsh
yav -u https://github.com/vim-latex/vim-latex
```
## 配置
然后在~/.vimrc.after中设置它的[必须配置](http://vim-latex.sourceforge.net/documentation/latex-suite/recommended-settings.html).

## PATH
习惯性在.zprofile中加入texlive的路径. 在vim下都可行,但是在macvim下怎么就不出pdf了.

错误原因`error 32512 (driver return code) generating output`,详细见[这里](http://mlyixi.byethost32.com/blog/?p=665)

果断在/etc/paths.d中加入texlive的路径,问题解决.

## xelatex的配置
在texrc中设置(当然可以在.vimrc.after中改):
```zsh
TexLet g:Tex_DefaultTargetFormat = 'pdf'
TexLet g:Tex_CompileRule_pdf = ‘xelatex -synctex=1 -interaction=nonstopmode -file-line-error-style $*’
TexLet g:Tex_ViewRule_pdf = 'Skim'
```
## syntastic的问题
语法检查出错(注释总说是错的),在.vimrc.after中加入`let g:syntastic_tex_checkers=['chktex']`

## 正反向搜索
系统自带的Preview貌似不支持,只能安装[Skim](http://skim-app.sourceforge.net/).

打开`preference->sync->Preset->MacVim`

## 字体
xetex对中文支持较好(unicode),其对中文字体的支持宏包是xeCJK.

ctex宏包是中文版式,可使用xeCJK(针对xelatex),也可以使用latex的宏包(CJK,CCT等).

mac和linux下要使用ctex宏包,记得安装adobefonts/winfonts, macfonts貌似现在还不能用,unixfonts更不用说了.

## 常用快捷键
现在的vim-latex已经不用`\`作引导键了,因为问题多多.现在默认用`,`.

```zsh
:TTemplate 插入模板
,ll		编辑
,lv		查看
,ls		正向搜索
Ctrl-J	跳转
F5		插入包/环境(在document环境外/内)
F7		插入/展开命令
F9		插入引用/图片
Shift-F5	根据环境提示改变环境
Shift-F7	根据环境提示改变命令
F6/,rf	折叠
```
## 希腊字母
<pre>
`a -`z 
`D = \Delta
`F = \Phi
`G = \Gamma
`Q = \Theta
`L = \Lambda
`X = \Xi
`Y = \Psi
`S = \Sigma
`U = \Upsilon
`W = \Omega
</pre>

## 数学命令/符号
<pre>
`^   Expands To   \Hat{<++>}<++>
`_   expands to   \bar{<++>}<++>
`6   expands to   \partial
`8   expands to   \infty
`/   expands to   \frac{<++>}{<++>}<++>
`%   expands to   \frac{<++>}{<++>}<++>
`@   expands to   \circ
`0   expands to   ^\circ
`=   expands to   \equiv
`\   expands to   \setminus
`.   expands to   \cdot
`*   expands to   \times
`&   expands to   \wedge
`-   expands to   \bigcap
`+   expands to   \bigcup
`(   expands to   \subset
`)   expands to   \supset
`<   expands to   \le
`>   expands to   \ge
`,   expands to   \nonumber
`~   expands to   \tilde{<++>}<++>
`;   expands to   \dot{<++>}<++>
`:   expands to   \ddot{<++>}<++>
`2   expands to   \sqrt{<++>}<++>
`|   expands to   \Big|
`I   expands to   \int_{<++>}^{<++>}<++>
</pre>
## 视图模式下的选择
<pre>
`(  encloses selection in \left( and \right)
`[  encloses selection in \left[ and \right]
`{  encloses selection in \left\{ and \right\}
`$  encloses selection in $$ or \[ \]
</pre>
## Alt快捷键
Alt键的配置要改
<pre>
Alt-L
Alt-B
Alt-C
Alt-I
</pre>