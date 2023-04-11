---
title: GCPN中的GAN
date: 2021-03-15 16:39:00
categories: ["学习"]
tags: ["毕设","算法","GCPN"]
mathjax: true
---

# GCPN中的GAN



## GAN

Generative Adversarial Network

**目标：训练高水平的判别器与生成器**

整个过程可以用文物鉴定来理解，判别器对应着鉴定机构，生成器对应造假团伙，样本对应着各种“文物”。这些“文物”中存在着真品国宝与赝品，鉴定机构对这些"文物"也有两种判定：真品与赝品。其中所有国宝都是已经存在的古人的作品是相对静态的，而赝品则由造假团伙源源不断地制作是动态的。feedback: 在鉴定完成后，造假团伙会根据鉴定机构的结果改变策略来生成更以假乱真的作品，鉴定机构也会通过得知文物来自造假团伙来更新自己的判断依据。

<!-- more -->

国宝：$$\{x_i\}^{N}_{i=1} $$ 满足分布$$P_{data}$$

赝品：$$P_g(x:\theta_g)$$ 通常是一个神经网络，x可能来自一个隐变量$$z \sim P_z(z)$$     ，$$ x=G(z:\theta_g)$$

因此，训练过程：

- 鉴定机构：

  - if x is from $$P_{data}$$ , then $$D(x)\uparrow$$  same as $$log D(x)\uparrow$$

  - if x is from $$P_g$$(z is from $$P_z$$) , then $$D(x)\downarrow$$ same as $$log(1- D(G(z)))\uparrow$$
  - therefore $$\max\limits_{D} \mathbb E_{x \sim P_{data}}[logD(x)]+\mathbb E_{z\sim P_z}[log(1-D(G(z)))]$$

- 造假团伙：

  - if x is from $$P_g$$   , then $$D(G(z))\uparrow$$  same as $$log(1-D(G(z)))\downarrow$$
  - $$\min \limits_{G} \mathbb E_{z \sim P_z}[log(1-D(G(z)))] $$

- 总目标：

  - $$\min \limits_{G} \max \limits_{D} \mathbb E_{x \sim P_{data}}[logD(x)]+\mathbb E_{z\sim P_z}[log(1-D(G(z)))]$$

最终的最优情况下 $$P_g = P_d , D_G = 1/2$$ 造假团队水平和古人一样，鉴定机构对造假团队失去鉴定能力。

## GCPN中的GAN

### 生成器

GCPN中的生成器是一个图卷积策略生成器，也就是gcn_policy.py中的GCNPolicy类。

首先将图中结点经过一个嵌入层计算结点嵌入，并进行降维

```python
#嵌入层
ob_node = tf.layers.dense(ob['node'],8,activation=None,use_bias=False,name='emb') # embedding layer
```

之后经过三层GCN层，将邻接矩阵中的边信息加入到结点嵌入中，得到了能代表整个图的结点嵌入。在这个结点嵌入去掉无效结点并加入聚合所有结点的代表整图的一行嵌入，得到emb_node，图嵌入。

emb_node变成了图的代表，然后只需根据emb_node生成GCPN环境中动作的四元素分布：[是否停止，第一点，第二点，边类型]，

emb_node $$\rightarrow$$ 两层网络 $$\rightarrow$$ logits_stop

emb_node $$\rightarrow$$ 两层网络 $$\rightarrow$$ logits_first 采样得出emb_first

concat(emb_first,emb_node) $$\rightarrow$$ 两层网络 $$\rightarrow$$ logits_second 采样得出emb_second

concat(emb_first,emb_second) $$\rightarrow$$ 两层网络 $$\rightarrow$$ logits_edge 

从而得出动作分布的logits

### 判别器

判别网络同样在gcn_policy.py中： discriminator_net方法：

```python
def discriminator_net(ob,args,name='d_net'):
    with tf.variable_scope(name,reuse=tf.AUTO_REUSE):
        ob_node = tf.layers.dense(ob['node'], 8, activation=None, use_bias=False, name='emb')  # embedding layer
        if args.bn==1:
            ob_node = tf.layers.batch_normalization(ob_node,axis=-1)
        emb_node = GCN_batch(ob['adj'], ob_node, args.emb_size, name='gcn1',aggregate=args.gcn_aggregate)
        for i in range(args.layer_num_d - 2):
            if args.bn==1:
                emb_node = tf.layers.batch_normalization(emb_node,axis=-1)
            emb_node = GCN_batch(ob['adj'], emb_node, args.emb_size, name='gcn1_'+str(i+1),aggregate=args.gcn_aggregate)
        if args.bn==1:
            emb_node = tf.layers.batch_normalization(emb_node,axis=-1)
        emb_node = GCN_batch(ob['adj'], emb_node, args.emb_size, is_act=False, is_normalize=(args.bn == 0), name='gcn2',aggregate=args.gcn_aggregate)
        if args.bn==1:
            emb_node = tf.layers.batch_normalization(emb_node,axis=-1)
        # emb_graph = tf.reduce_max(tf.squeeze(emb_node2, axis=1),axis=1)  # B*f
        emb_node = tf.layers.dense(emb_node, args.emb_size, activation=tf.nn.relu, use_bias=False, name='linear1')
        if args.bn==1:
            emb_node = tf.layers.batch_normalization(emb_node,axis=-1)


        if args.gate_sum_d==1:
            emb_node_gate = tf.layers.dense(emb_node,1,activation=tf.nn.sigmoid,name='gate')
            emb_graph = tf.reduce_sum(tf.squeeze(emb_node*emb_node_gate, axis=1),axis=1)  # B*f
        else:
            emb_graph = tf.reduce_sum(tf.squeeze(emb_node, axis=1), axis=1)  # B*f
        logit = tf.layers.dense(emb_graph, 1, activation=None, name='linear2')
        pred = tf.sigmoid(logit)
        # pred = tf.layers.dense(emb_graph, 1, activation=None, name='linear1')
        return pred,logit
```

GCPN中有两个判别器，可以理解为过程判别器与结果判别器，这两个判别器是分别训练的。

与生成器类似将分子图经过三层GCN后，在经过两层网络得到logits，即判别器结果分布。

### Feedback

在pposgd_simple_gcn.py的traj_segment_generator()中，这个方法是生成一段状态、动作、奖励、价值序列。

在GAN的公式中已经可以体现出D与G两者是互相影响的，在训练D网络的同时已经利用了来自G生成的数据，GCPN在生成动作时将$$- \mathbb E_{z \sim P_z}[log(1-D(G(z)))] $$作为生成过程中的一项奖励，也就是说在执行完一个动作后，得到的奖励为：环境奖励（写在gym内的关于化学性质的）+ 过程判别器奖励 + 结果判别器奖励（如果状态结束了） 