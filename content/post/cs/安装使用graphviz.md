---
title: "安装使用graphviz"
date: 2015-01-29T14:09:52+08:00
tags: ["Graphviz"]
categories: ["CS"]
toc: true
---

Graphviz是一个脚本化创建图表并自动布局的开源应用程序.应用广泛.

## 安装

```zsh
brew install graphviz
```
## 使用
### 无向图graph

```zsh
graph example1 {
Server1 -- Server2
Server2 -- Server3
Server3 -- Server1
}
```
### 有向图digraph

```zsh
digraph example2 {
Server1 -> Server2
Server2 -> Server3
Server3 -> Server1
}
```

## 对象和属性
### 对象

有全局对象和局部对象之分

* 点node
* 线edge

### 属性
#### 图属性:

* 大小size: "4,4" 以英寸为单位

#### 点属性

* 形状shape: 方形box, 圆circle, 椭圆**ellipse**, 表record, 文字plaintext
* 宽度width:
* 高度height: 
* 颜色color: **black**,颜色值可按RGB=".7 .3 1.0"指定或查颜色名.
* 填充颜色fillcolor
* 样式style: bold, dotted, filled
* `标签label`: 默认为**点对象的名字**,可以用它指定样式;当形状为record时,可进行更复杂的操作(后面作介绍)
* 自定义形状shape: shape=polygon, sides=5, peripheries=3, distortion=.7, skew=.4


```zsh
Server1 [shape=box, label="Server1nWeb Server", fillcolor="#ABACBA", style=filled]
```
#### 边属性
* 颜色color
* 样式style: bold, dotted, filled
* 方向dir: **forward**,back,both,none

### 子图subgraph

```zsh
subgraph sg1 {
Server1 -> Server2
Server2 -> Server3
Server3 -> Server1
}
```

### 布局

有木有发现graphviz产生的图有些不好看,这就要说下graphviz的布局算法了.

* dot: 默认布局方式,主要用于有向图
* neato: 无向图
* twopi: 径向布局
* circo: 圆环布局
* fdp: 无向图
* sfdp: 超大无向图
* patchwork: 树形布局

怎么使用? 把dot换成这些程序名即可.

### 例子
#### UML图

```zsh
digraph st2{
rankdir=TB;
 
node [fontsize = 10, color="skyblue", shape="record"];
 
edge [fontsize = 10, color="crimson", style="solid"];
 
st_hash_type [label="{<head>st_hash_type|(*compare)|(*hash)}"];
st_table_entry [label="{<head>st_table_entry|hash|key|record|<next>next}"];
st_table [label="{st_table|<type>type|num_bins|num_entries|<bins>bins}"];
 
st_table:bins -> st_table_entry:head;
st_table:type -> st_hash_type:head;
st_table_entry:next -> st_table_entry:head [style="dashed", color="forestgreen"];
}
```

#### 组成图

```zsh
digraph idp_modules{
fontsize = 12;
 
node [ shape = "record" ]; 
 
    subgraph cluster_sl{
        label="IDP支持层";
        bgcolor="mintcream";
        node [shape="Mrecord", color="skyblue", style="filled"];
        network_mgr [label="网络管理器"];
        log_mgr [label="日志管理器"];
        module_mgr [label="模块管理器"];
        conf_mgr [label="配置管理器"];
        db_mgr [label="数据库管理器"];
    };
 
    subgraph cluster_md{
        label="可插拔模块集";
        bgcolor="lightcyan";
        node [color="chartreuse2", style="filled"];
        mod_dev [label="开发支持模块"];
        mod_dm [label="数据建模模块"];
        mod_dp [label="部署发布模块"];
    };
 
mod_dp -> mod_dev [label="依赖..."];
mod_dp -> mod_dm [label="依赖..."];
mod_dp -> module_mgr [label="安装...", color="yellowgreen", arrowhead="none"];
mod_dev -> mod_dm [label="依赖..."];
mod_dev -> module_mgr [label="安装...", color="yellowgreen", arrowhead="none"];
mod_dm -> module_mgr [label="安装...", color="yellowgreen", arrowhead="none"];
}
```

#### 向量机

```zsh
digraph automata_0 {
 
size = "8.5, 11";
 
node [shape = circle];
 
0 [ style = filled, color=lightgrey ];
2 [ shape = doublecircle ];
 
0 -> 2 [ label = "a " ];
0 -> 1 [ label = "other " ];
1 -> 2 [ label = "a " ];
1 -> 1 [ label = "other " ];
2 -> 2 [ label = "a " ];
2 -> 1 [ label = "other " ];
 
"Machine: a" [ shape = plaintext ];
}
```

#### 树形图

```zsh
digraph ast{
node [shape = circle];
node [shape="plaintext"];

mul [label="mul(*)"];
add [label="add(+)"];
 
add -> 3
add -> 4;
mul -> add;
mul -> 5;
}
```
#### 状态图

```zsh
digraph finite_state_machine {
 
rankdir = LR;
size = "8,5"
 
node [shape = doublecircle]; 
 
LR_0 LR_3 LR_4 LR_8;
 
node [shape = circle];
 
LR_0 -> LR_2 [ label = "SS(B)" ];
LR_0 -> LR_1 [ label = "SS(S)" ];
LR_1 -> LR_3 [ label = "S($end)" ];
LR_2 -> LR_6 [ label = "SS(b)" ];
LR_2 -> LR_5 [ label = "SS(a)" ];
LR_2 -> LR_4 [ label = "S(A)" ];
LR_5 -> LR_7 [ label = "S(b)" ];
LR_5 -> LR_5 [ label = "S(a)" ];
LR_6 -> LR_6 [ label = "S(b)" ];
LR_6 -> LR_5 [ label = "S(a)" ];
LR_7 -> LR_8 [ label = "S(b)" ];
LR_7 -> LR_5 [ label = "S(a)" ];
LR_8 -> LR_6 [ label = "S(b)" ];
LR_8 -> LR_5 [ label = "S(a)" ];
}
```


## vim插件
[vmgraphviz](!https://github.com/wannesm/wmgraphviz.vim)

```zsh
yar -u https://github.com/wannesm/wmgraphviz.vim
```

它使用`<LocalLeader>`作为命令前缀,命令和latex-suite差不多

```vim
let g:WMGraphviz_output="png"
let maplocalleader=","
let mapleader=","
```

使用`,ll`,`,lv`,`,li`编译查看.

自动补全:Ctr-X Ctr-O