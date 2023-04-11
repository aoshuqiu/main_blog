---
title: NCE阅读笔记
date: 2021-12-06 22:02:21
tags: ["机器学习","图嵌入","NCE"]
categories: "学习"
mathjax: true
---

# GRAPH GRAMMERS WITH NEIGHBOURHOOD-CONTROLLED EMBEDDING

邻居控制图嵌入，是一种图的序列嵌入，在用于以序列形式生成图的方法中便利且准确度高，因此可以更好地适配强化学习的序列决策过程。

<!-- more -->

## 预备知识

- $$X$$与$$Y$$都为集合。那么，$$Id_x$$表示$$X$$上的恒等关系，$$2^X$$表示$$X$$的子集集合，$$X\backslash Y$$ 表示集合$$\left\{x|x\in X,x\notin Y \right\}$$。如果$$X$$是有限的，那么$$\#X$$表示$$X$$的基数。
- $$X,Y,Z$$都为集合，$$f$$为从$$X$$到$$Y$$的函数，$$g$$为从$$Y$$到$$Z$$的函数。那么， $$g\circ f$$表示$$f$$和$$g$$的组合（先$$f$$,后$$g$$)
- 一个无向结点标记图是一个系统$$H=(V,E,\Sigma,\phi)$$，其中$$V$$是一个有限非空集合，称作结点集；$$E$$是一个关于$$V$$中两个元素的二重集合，称作边集；$$\Sigma$$是一个有限非空集，称作标签集（或字母表）；$$\phi$$是从$$V$$到$$\Sigma$$的函数，称作标识函数。$$H$$称作标签集$$\Sigma$$上的一个图。**之后图$$H$$中的元素会标记为$$V_H,E_H,\Sigma_H,\phi_H$$。**
- H为一个图其中一条边$$\{x,y\}\in E_H$$。我们说边$$\{x,y\}$$与结点$$x,y$$相关，$$x,y$$互为邻居。
- H为一个图其中一个结点$${x}\in V_H$$。那么$$x$$的度表示为$$deg(x)$$，它代表与$$x$$相关的边的数目。
- $$A$$与$$B$$都为图，若$$V_A \subseteq V_B,E_A \subseteq E_B\cap\{\{x,y\}|x,y\in V_A\},\Sigma_A \sube \Sigma_B$$，则称$$A$$为$$B$$的子图。其中若$$E_A = E_B\cap\{\{x,y\}|x,y\in V_A\}$$，则称则称$$A$$为$$B$$的全子图，在这种情况下称$$A$$为在$$B$$中由$$V_A$$支撑生成的子图。**$$B-A$$为在$$B$$中由$$ V_B \backslash V_A$$支撑生成的子图。**
- $$H$$与$$\bar H$$为同一字母表$$\Sigma$$上的图，从$$H$$到$$\bar H$$的同构为一个双射函数$$h$$，其中$$\phi_{\bar H}\circ h=\phi_H \quad and\quad E_A=\{\{h(x),h(y)\}|\{x,y\}=E_H\} $$，则定义$$H$$与$$\bar H$$同构
- 完全图中$$E_H = \{\{x,y\}|x,y \in V_H\}$$
- 离散图中$$E_H=\varnothing$$
- 连接图$$H$$中每个$$x,y \in V_H$$,都存在着一个结点序列$$x_1,x_2,…,x_n$$，使得$$x_1=x, x_n=y$$，对于$$1\le i\le n-1$$，$$x_i$$为$$x_{i+1}$$的邻居。

## NCE基本定义

**Definition 1.**一个NCE语法图$$G=(\Sigma,\Delta,P,Z)$$，其中$$\Sigma$$为有限非空集，称作全字母表；$$\Delta$$为$$\Sigma$$的子集，称作终结字母表；P为产生式的有限集，产生式的形式为$$(\alpha,\beta,\psi)$$，其中$$\alpha$$为一个连接图，$$\beta$$为一个图，$$\psi$$为一个函数$$V_\alpha \times V_\beta \times \Sigma \rightarrow \{0,1\}$$，$$\psi$$被称作产生式的嵌入函数；Z是$$\Sigma$$上的一个图，称作公理图。

​	NCE语法下的一个派生步骤如下。H为一个图，$$\pi=(\alpha,\beta,\psi)$$为P的一个产生式，$$\hat \alpha$$为H的一个支撑子图且$$\hat \alpha$$与$$\alpha$$同构（h为同构映射)，$$\hat\beta$$与$$\beta$$同构（g为同构映射）而且$$V_{\hat\beta} \cap V_{H-\hat\alpha}=\varnothing$$ 。那么$$\pi$$到$$\hat\alpha$$上的应用首先从H中移除$$\hat\alpha$$，之后用$$\hat\beta$$替换$$\hat\alpha$$，最终对每个$$n\in V_\beta,v \in V_{H}\backslash V_{\hat a}$$，若满足以下条件则加入边{n,v}：

 - 存在一个结点$$m \in V_\alpha$$，$$\{h(m),v\} \in E_H$$ 并且，

 - $$\psi(m,g(n),\phi_H(v))=1$$。

   

