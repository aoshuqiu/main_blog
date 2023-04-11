---
title: GCPN 图卷积策略网络分子生成
date: 2021-1-13 14:23:10
tags: ["机器学习","强化学习","图表示"]
categories: "学习"
mathjax: true
---

《Graph Convolutional Policy Network for Goal-Directed Molecular Graph Generation》

## GCPN

### 应用目标导向

1. 分子空间大：$10^{60}$
2. 化学空间离散，分子性质对微小的结构改变也很敏感

## GCPN

1. 图卷积策略网络，引导生成特定期望目标同时用基本化学规则限制输出空间
2. 图表示+强化学习+对抗训练

<!-- more -->

## 图表示

1. 部分图代表子结构 部分字符代表不了
2. 生成过程中可以进行化学检查

## 强化学习

#### 1. 期望分子性质与约束复杂且不可微

 为什么？ 

​			**分子性质对微小的结构改变也很敏感**，

​			可微：相似分子性质相似

​			事实：同分异构体都可能有完全相反的性质

​	导致？

​			不能直接通过目标函数表示这些性质

​	生成模型通过生成器生成策略（序列生成）来解决

​	强化学习用动态性质与奖励函数表示：不需要性质可微

#### 2.强化学习可以主动探索

​	探索训练集之外的化学空间（怎么探索的？）

#### 3. 直接优化

​	VAE间接优化潜在嵌入空间的分子性质

​	直接优化分子图的分子性质（什么性质？）

## 对抗训练

多属性预测困难

对抗性训练一个可学习的判别器



## 分子生成环境

**1. 状态空间**

分子图$ G(A,E,F)$,$A \in \{0,1\}^{n\times n}$是邻接矩阵，$F \in \mathbb{R}^{n \times d}$为每个节点的特征（假设有d个特征，这里是d个原子或子结构），$ E \in \{0,1\}^{b \times n \times n}$为边的种类与邻接矩阵。（A与E不重复吗？）

**2.动作空间**

在当前分子图上加上一堆待连的分子和子结构（脚手架），动作类似链路预测

**3.转移动态**

体现化学领域规则

**4.奖励设计**

中间奖励+最终奖励

中间奖励：有效性奖励+对抗奖励

最终奖励：化学领域奖励（QED）+对抗奖励

## 环境搭建

GCPN文章作者通过openai的gym搭建了一个强化学习分子生成环境

也就是gym_molecule/envs/molecule.py

其中定义了继承自gym.Env的分子环境 MoleculeEnv

MoleculeEnv中定义了一系列用于分子生成与观测的函数

### normalize_adj（adj）

结点嵌入

$H^{(l+1)}=AGG(ReLU(\{\tilde{D_i}^{-\tfrac{1}{2}}\tilde E_{i}\tilde{D_i}^{-\tfrac{1}{2}}H^{(l)}W_{i}^{(l)}\},\forall i \in (1,…,b)))$

$\tilde{D_i}^{-\tfrac{1}{2}}\tilde E_{i}\tilde{D_i}^{-\tfrac{1}{2}}$类似一个归一化的拉普拉斯矩阵$L^{norm}$，其中引入自身度矩阵来解决自传递问题，对邻接矩阵通过度矩阵进行归一化，来防止度对结点权重的影响（也就是说10条边的结点比1条边的结点影响大的情况）

```python
	#adj为图邻接矩阵，相当于得到归一化拉普拉斯矩阵用于嵌入 adj第一维度是边类型，二三维度是邻接矩阵
    def normalize_adj(self,adj):
        degrees = np.sum(adj,axis=2)
        # print('degrees',degrees)
        D = np.zeros((adj.shape[0],adj.shape[1],adj.shape[2]))
        for i in range(D.shape[0]):
            D[i,:,:] = np.diag(np.power(degrees[i,:],-0.5))
        adj_normal = D@adj@D
        adj_normal[np.isnan(adj_normal)]=0
        return adj_normal
```



## step(action)

action是一个四元组：两个端点、边类型预测、终止预测

step函数首先要执行动作：添加原子或添加化学键

生成终止条件有一个最小动作数的要求min_action

```python
        if action[0,3]==0 or self.counter < self.min_action: # not stop
            stop = False
            if action[0, 1] >= total_atoms:
                self._add_atom(action[0, 1] - total_atoms)  # add new node
                action[0, 1] = total_atoms  # new node id
                self._add_bond(action)  # add new edge
            else:
                self._add_bond(action)  # add new edge
        else: # stop
            stop = True
```



其中还要完成奖励的设置

中间奖励+最终奖励

中间奖励： 主要是化合价的检测

- 化合价正确，动作成功执行，奖励为正。

- 化合价错误或化学键已经存在，奖励为负

```python
        ### calculate intermediate rewards 
        if self.check_valency(): 
            if self.mol.GetNumAtoms()+self.mol.GetNumBonds()-self.mol_old.GetNumAtoms()-self.mol_old.GetNumBonds()>0:
                reward_step = self.reward_step_total/self.max_atom # successfully add node/edge
                self.smile_list.append(self.get_final_smiles())
            else:
                reward_step = -self.reward_step_total/self.max_atom # edge exist
        else:
            reward_step = -self.reward_step_total/self.max_atom  # invalid action
            self.mol = self.mol_old
```



最终奖励：目标性质+化学有效性

化学有效性通过RDKit完成，先把分子对象Mol转化成SMILES形式，之后如果能成功转化回来，说明分子符合化学有效性。

```python
    #检测化学有效性
    def check_chemical_validity(self):
        """
        Checks the chemical validity of the mol object. Existing mol object is
        not modified. Radicals pass this test.
        :return: True if chemically valid, False otherwise
        """
        s = Chem.MolToSmiles(self.mol, isomericSmiles=True)
        m = Chem.MolFromSmiles(s)  # implicitly performs sanitization
        if m:
            return True
        else:
            return False
```

反之，则分子不符合化学有效性，最终奖励中要施加惩罚



目标性质三种：

- 药物相似性qed
- 油水分配系数LogP

- 综合可获得性Synthetic accessibility（SA）

其中 qed与MolLogP都在rdkit.chem.descriptors中有对应的计算函数

SA通过第三方库sascorer.py中的calculateScore进行计算

## GCN

### 计算结点嵌入

l+1层的结点嵌入 $H^{(l+1)} \in \mathbb R^{(n+c)\times k}$ 

n是图中结点数，c是脚手架结点数，k是嵌入后结点表示维数

$H^{(l+1)}=AGG(ReLU(\{\tilde{D_i}^{-\tfrac{1}{2}}\tilde E_{i}\tilde{D_i}^{-\tfrac{1}{2}}H^{(l)}W_{i}^{(l)}\},\forall i \in (1,…,b)))$

$E_i$是第i类边的邻接矩阵，$\tilde E_i = E_i + I$是考虑到自身结点信息的修改后的邻接矩阵，$\tilde D_i = \sum_{k}\tilde E_{ijk}$ 是修改后的度矩阵。

$\tilde{D_i}^{-\tfrac{1}{2}}\tilde E_{i}\tilde{D_i}^{-\tfrac{1}{2}}$类似一个归一化的拉普拉斯矩阵$L^{norm}$，其中引入自身度矩阵来解决自传递问题，对邻接矩阵通过度矩阵进行归一化，来防止度对结点权重的影响（也就是说10条边的结点比1条边的结点影响大的情况）

$L^{norm}H^{(l)}$相当于叠加结点自身与邻居结点的特征，通过多层$l$来利用更多层次邻居（邻居的邻居）的信息。

### 动作预测

链路预测

动作：四个组成 两个端点、边类型预测、终止预测

$ a_t = CONCAT(a_{first},a_{second},a_{edge},a_{stop})$

$f_{first}(s_{t}) = SOFTMAX(m_f(X)),\quad \quad \quad a_{first} \sim f_{first}(s_t) \in \{0,1\}^{n}$

$f_{second}(s_{t}) = SOFTMAX(m_s(X_{a_{first}},X)),\quad \quad a_{second} \sim f_{second}(s_t) \in \{0,1\}^{n+c}$

$f_{edge}(s_{t}) = SOFTMAX(m_e(X_{a_{first}},X_{a_{second}})),\quad \quad a_{edge} \sim f_{edge}(s_t) \in \{0,1\}^{b}$

$f_{stop}(s_{t}) = SOFTMAX(m_t(AGG(X))),\quad \quad a_{stop} \sim f_{stop}(s_t) \in \{0,1\}$

### ppo？

#### 策略梯度方法推导

轨迹$\tau = (s_{0},a_{0},…,s_{h},a_{h})$

目标：找到最优神经网络参数$\theta^{*}$最大化收益关于轨迹分布的期望

$\theta^{*}=\mathop{\arg\max}\limits_{\theta} \mathbb{E}_{\tau \sim p_{\theta}(\tau)}\bigg[\sum\limits_{t}r(s_{t},a_{t})\bigg]$

目标函数 $J(\theta)=\mathbb{E}_{\tau\sim p_{\theta}(\tau)}\bigg[\sum\limits_{t}r(s_t,a_t)\bigg]$

定义轨迹收益$r(\tau)=\sum\limits_t r(s_t,a_t)$，目标函数变成$J(\theta)=\mathbb{E}_{\tau\sim p_{\theta}(\tau)}[r(\tau)]$

假设$\tau$的分布函数$p_{\theta}(\tau)$是可微的，拆开期望： $J(\theta)=\int p_{\theta}(\tau)r(\tau)\,{\rm d}\tau$

对两边求梯度：$\nabla_{\theta} J(\theta)=\int \nabla_{\theta} p_{\theta}(\tau)r(\tau)\,{\rm d}\tau$

其中等式$\nabla_{\theta}p_{\theta}(\tau) = p_{\theta}(\tau)\nabla_\theta logp_\theta(\tau)$成立

因此
$$
\begin{aligned} 
\nabla_{\theta} J(\theta)&=\int p_{\theta}(\tau)\nabla_\theta logp_\theta(\tau)r(\tau)\,{\rm d}\tau\\
&= \mathbb{E}_{\tau\sim p_\theta(\tau)}\bigg[\nabla_\theta logp_\theta(\tau)r(\tau)\bigg]\end{aligned}
$$
其中$p_{\theta}(\tau)=p(s_1)\prod\limits_{t=1}^{T}\pi_{\theta}(a_t,s_t)p(s_{t+1})$

则$\nabla_\theta logp_\theta(\tau) = \nabla_\theta \sum \limits_{t=1}^{T}log\pi_{\theta}(a_t,s_t)$

$\nabla_{\theta} J(\theta) = \mathbb{E}_{\tau\sim p_\theta(\tau)}\bigg[\bigg( \sum \limits_{t=1}^{T}\nabla_\theta log\pi_{\theta}(a_t,s_t)\bigg)\bigg(\sum \limits_{t=1}^{T}r(s_t,a_t)\bigg)\bigg]$

实际抽样时用平均来进行估算

$\nabla_{\theta} J(\theta) \approx \frac{1}{N} \sum \limits_{i=1}^{N} \bigg[\bigg( \sum \limits_{t=1}^{T}\nabla_\theta log\pi_{\theta}(a_{i,t},s_{i,t})\bigg)\bigg(\sum \limits_{t=1}^{T}r(s_{i,t},a_{i,t})\bigg)\bigg]$

之后用 $\theta \gets \theta + \nabla_{\theta} J(\theta)$更新参数$\theta$

##### 解决高方差问题

###### 改变因果关系

当前时间策略只能影响之后的奖励

$\nabla_{\theta} J(\theta) \approx \frac{1}{N} \sum \limits_{i=1}^{N}\sum \limits_{t=1}^{T} \bigg[\bigg( \nabla_\theta log\pi_{\theta}(a_{i,t},s_{i,t})\bigg)\bigg(\sum \limits_{t'=t}^{T}r(s_{i,t'},a_{i,t'})\bigg)\bigg]$

###### 引入基线

$\nabla_{\theta} J(\theta)= \mathbb{E}_{\tau\sim p_\theta(\tau)}\bigg[\nabla_\theta logp_\theta(\tau)(r(\tau)-b)\bigg]$

eg： $b_t = \frac{1}{N} \sum \limits_{i} Q^\pi(s_{i,t},a_{i,t})$






