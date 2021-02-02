---
title: "MathJax基础教程和快速参考"
date: 2014-10-06T14:08:52+08:00
tags: ["MathJax"]
categories: ["CS"]
toc: true
---

更多资料请参考[此文](http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference)和[latex手册](http://mohu.org/info/symbols/symbols.htm)

## MathJax基础
* 用mathjax渲染的公式可以在现代浏览器里面查看latex代码，方法是在公式上*右键*->*Show Math As*->*Tex commands*

## Wordpress中使用latex
Wordpress本身会提供latex插件:WP-latex,是取自官方插件包jetpack,但是使用上不方便,所以就用了国人写的LaTeX for WordPress插件,可惜更新不太快.

* 对于行内公式，使用`$ \$..\$ $`;对于行间公式，使用`$ \$\$..\$\$ $`.但是对于一些插件，这个是可以改变的，比如我现在用的wordpress插件使用的是行内`$ \$\$..\$\$ $`,行间`$ \$\$!..\$\$ $`.
* 公式换行使用`\\\`而不是`\\`.

## Latex基础
### 常用函数
* `\sum` 
* `\int`, `\iint`
* `\max`
* `\frac{x}{y}`
* `\sqrt[3]{x}`
* `\lim_{x\to 0}`
* `\sin x`

### 常用关系符
* `\lt \gt \le \ge \neq \ll \gg`: `$\lt \gt \le \ge \neq \ll \gg$`

* `\times \div \pm \mp \cdot \cdots  \ldots`: `$\times \div \pm \mp \cdot \cdots \ldots$`

* `\cup \cap \setminus \subset \subseteq \subsetneq \in \notin \emptyset \varnothing`: `$\cup \cap \setminus \subset \subseteq \subsetneq \in \notin \emptyset \varnothing$`
* `{n+1 \choose 2k}`或`\binom{n+1}{2k}`: `$\binom{n+1}{2k}$`
* `\to \rightarrow \leftarrow \Rightarrow \Leftarrow \mapsto`:
`$\to \rightarrow \leftarrow \Rightarrow \Leftarrow \mapsto$`
* `\land \lor \lnot \forall \exists \top \bot \vdash \vDash`:
`$\land \lor \lnot \forall \exists \top \bot \vdash \vDash$`
* `\star \ast \oplus \circ \bullet`:`$\star \ast \oplus \circ \bullet$`
* `\approx \sim \cong \equiv \prec`:`$\approx \sim \cong \equiv \prec$`
* `\infty \aleph_0`:`$\infty \aleph_0$`
* `\nabla \partial`:`$\nabla \partial$`
* `\Im \Re`: `$\Im \Re$`
* `\text \, \;`:`$\text{Hello}\,\text{World}\;\text{!}$`


## 字体
* 希腊字母:应该都会拼吧,不会拼自己查wiki去
* 黑板粗体: `\mathbb`或`\Bbb`: 
$$\Bbb{A}\Bbb{B}\Bbb{C}\Bbb{D}\Bbb{E}\Bbb{F}\Bbb{G}\Bbb{H}\Bbb{I}\Bbb{J}\Bbb{K}\Bbb{L}\Bbb{M}\Bbb{N}\Bbb{O}\Bbb{P}\Bbb{Q}\Bbb{R}\Bbb{S}\Bbb{T}\Bbb{U}\Bbb{V}\Bbb{W}\Bbb{X}\Bbb{Y}\Bbb{Z}$$

* 黑体: `\mathbf`:
$$\mathbf{A}\mathbf{B}\mathbf{C}\mathbf{D}\mathbf{E}\mathbf{F}\mathbf{G}\mathbf{H}\mathbf{I}\mathbf{J}\mathbf{K}\mathbf{L}\mathbf{M}\mathbf{N}\mathbf{O}\mathbf{P}\mathbf{Q}\mathbf{R}\mathbf{S}\mathbf{T}\mathbf{U}\mathbf{V}\mathbf{W}\mathbf{X}\mathbf{Y}\mathbf{Z}$$

* 打字机体: `\mathtt`: 常见的字体就不列举了.
$$\mathtt{A}\mathtt{B}\mathtt{C}\mathtt{D}\mathtt{E}\mathtt{F}\mathtt{G}\mathtt{H}\mathtt{I}\mathtt{J}\mathtt{K}\mathtt{L}\mathtt{M}\mathtt{N}\mathtt{O}\mathtt{P}\mathtt{Q}\mathtt{R}\mathtt{S}\mathtt{T}\mathtt{U}\mathtt{V}\mathtt{W}\mathtt{X}\mathtt{Y}\mathtt{Z}$$

* 罗马字体: `\mathrm`:
$$\mathrm{A}\mathrm{B}\mathrm{C}\mathrm{D}\mathrm{E}\mathrm{F}\mathrm{G}\mathrm{H}\mathrm{I}\mathrm{J}\mathrm{K}\mathrm{L}\mathrm{M}\mathrm{N}\mathrm{O}\mathrm{P}\mathrm{Q}\mathrm{R}\mathrm{S}\mathrm{T}\mathrm{U}\mathrm{V}\mathrm{W}\mathrm{X}\mathrm{Y}\mathrm{Z}$$

* 书法体: `\mathcal`:
$$\mathcal{A}\mathcal{B}\mathcal{C}\mathcal{D}\mathcal{E}\mathcal{F}\mathcal{G}\mathcal{H}\mathcal{I}\mathcal{J}\mathcal{K}\mathcal{L}\mathcal{M}\mathcal{N}\mathcal{O}\mathcal{P}\mathcal{Q}\mathcal{R}\mathcal{S}\mathcal{T}\mathcal{U}\mathcal{V}\mathcal{W}\mathcal{X}\mathcal{Y}\mathcal{Z}$$

* 草书体: `\mathscr`: 
$$\mathscr{A}\mathscr{B}\mathscr{C}\mathscr{D}\mathscr{E}\mathscr{F}\mathscr{G}\mathscr{H}\mathscr{I}\mathscr{J}\mathscr{K}\mathscr{L}\mathscr{M}\mathscr{N}\mathscr{O}\mathscr{P}\mathscr{Q}\mathscr{R}\mathscr{S}\mathscr{T}\mathscr{U}\mathscr{V}\mathscr{W}\mathscr{X}\mathscr{Y}\mathscr{Z}$$

* Fraktur字体: `\mathfrak`: 
$$\mathfrak{A}\mathfrak{B}\mathfrak{C}\mathfrak{D}\mathfrak{E}\mathfrak{F}\mathfrak{G}\mathfrak{H}\mathfrak{I}\mathfrak{J}\mathfrak{K}\mathfrak{L}\mathfrak{M}\mathfrak{N}\mathfrak{O}\mathfrak{P}\mathfrak{Q}\mathfrak{R}\mathfrak{S}\mathfrak{T}\mathfrak{U}\mathfrak{V}\mathfrak{W}\mathfrak{X}\mathfrak{Y}\mathfrak{Z}$$

## 方言和可区分标志
* `\hat \widehat \bar \overline`:
$$\hat{x} \widehat{xyz} \bar{x} \overline{xyz}$$
* `\vec \overrightarrow`:
$$\vec{x} \overrightarrow{x}$$
* `\dot \ddot`:

## 矩阵

```tex
begin{matrix}
        1 & x 
        1 & y 
end{matrix}
```
$$
\begin{matrix}
        1 & x \\
        1 & y \\
\end{matrix}
$$

也可以使用pmatrix,bmatrix,Bmatrix,vmatrix,Vmatrix输出小括号,中括号,大括号,行列式,范数等.

$$
\begin{vmatrix}
        1 & x \\
        1 & y \\
\end{vmatrix}
$$
## 公式对齐

```tex
begin{align}
sqrt{37} & = sqrt{frac{73^2-1}{12^2}} 
 & = sqrt{frac{73^2}{12^2}cdotfrac{73^2-1}{73^2}}  
 & = sqrt{frac{73^2}{12^2}}sqrt{frac{73^2-1}{73^2}} 
 & = frac{73}{12}sqrt{1 - frac{1}{73^2}}  
 & approx frac{73}{12}left(1 - frac{1}{2cdot73^2}right) 
end{align}
```
```math
\begin{align}
\sqrt{37} & = \sqrt{\frac{73^2-1}{12^2}} \\
 & = \sqrt{\frac{73^2}{12^2}\cdot\frac{73^2-1}{73^2}} \\
 & = \sqrt{\frac{73^2}{12^2}}\sqrt{\frac{73^2-1}{73^2}} \\
 & = \frac{73}{12}\sqrt{1 - \frac{1}{73^2}} \\
 & \approx \frac{73}{12}\left(1 - \frac{1}{2\cdot73^2}\right)
\end{align}
```