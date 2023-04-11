---
title: GCPN源码分析
date: 2021-03-11 11:00:00
categories: ["学习"]
tags: ["毕设","算法","GCPN"]
mathjax: true
---
# GCPN源码分析



## run_molecule.py

整个程序的入口，主要任务是确定调用时参数、连接各模块

- train(args, seed, writer=none)
  
  - 创建环境：```env = gym.make('molecule-v0') ```  这个环境也是源码中定义的，需要安装库gym_molecule，在gym-molecule文件夹内命令行输入```pip install -e.```进行安装，在代码开头需要import gym_molecule 来使用 molecule-v0 环境。
  
  - 定义得出策略方法：
  
    ```python
    #定义用于从观测中得出策略的函数#
    def policy_fn(name, ob_space, ac_space):
        return gcn_policy.GCNPolicy(name=name, ob_space=ob_space, ac_space=ac_space, atom_type_num=env.atom_type_num,args=args)
    ```
  
    policy_fn，输入name（用于tf scope命名），ob_space，ac_space ，直接返回一个策略类
  
  - 调用pposgd_simple_gcn.learn()开始训练
<!-- more -->

- arg_parser()

  ​	定义一个新的argparser对象

- molecule_arg_parser()

  ​	在创建的argparser中加入一系列参数，例如'--dataset'，'--has_d_step'。

- main()

  ​	入口函数，排列参数后调用train（）进行训练
  
## gcn_policy.py

- GCN_batch(adj, node_feature, out_channels, is_act=True, is_normalize=False, name='gcn_simple', aggregate="sum")
  - 定义具体一个GCN层
  - adj: 拉普拉斯规范化邻接矩阵，规范化部分是在gym-molecule环境内完成的 (B,e,n,n) B:batchsize, e:edgesize, n: nodesize
  - node_feature: 结点特征矩阵 (B,1,n,d) d:feature dimension
  - out_channels: 输出特征维度
  - is_act: 要不要用激活函数
  - is_normalize: 结果要不要规范化
  - name: 本层中trainable变量所在scope名称，相当于一个gcn层的名字
  - aggregate: 最终输出要对边类型进行聚合，聚合方式：[sum,mean,concat]
  - return: node_embedding (B,1,n,f) gcn后结点嵌入

- discriminator_net(ob,args,name='d_net')

  - 判别网络对gym观测进行判别，判别网络也是先经过三层gcn，之后降维到一维，用逻辑斯特分布判断是否为过程/结果分子。

  - ob: 观测返回的ob，含有adj与node矩阵

  - args：调用程序时的参数，主要用于判断每层后是否要用batch_normalization

  - name: scope name

  - return:     pred:sigmoid分布，logit：逻辑斯特分布中的x

class *GCNPolicy(object)*:

- scope: 策略中所有trainable变量所在scope名称

- _init(self, ob_space, ac_space, kind, atom_type_num, args)

  - 初始化策略类，所有动作生成过程都在这个函数中，最终会生成一个从分布中采样的动作。

  - ob_space,ac_space: env返回的东西，ac_space没用到因为ac是作者指定的
  - kind: 没用到 
  - atom_type_num:  可用原子个数，主要用于蒙版蒙掉脚手架上原子来从已有原子中选边的第一个结点
  - args：参数
  
- act(self, stochastic, ob):

  - 通过观测ob返回一个采样的动作

  - stochastic： 随机？ 干嘛的 代码中没有调用过, 是一个多余的变量，但他调用act时都将它置为true

    

## pposgd_simple_gcn.py

- traj_segment_generator(args, pi, env, horizon, stochastic, d_step_func, d_final_func)
  - 顾名思义，生成一段动作，这一段动作不一定是生成同一个分子的过程，但是都是连续的时间段中的动作，且一开始都是从头生成。
  - args: 运行是输入的参数，这个函数中主要用来定义得到判别步骤的奖励的方法。
  - pi: 一个GCNPolicy类，待采样的策略
  - env: gym环境，这里是molecule-v0
  - horizon: 时间段的长度，也就是返回结果中列表的大小
  - stochastic: 用于act的参数，同样调用的时候都是true
  - d_step_func: 计算中间产物判别奖励的方法
  - d_final_func: 计算最终产物判别奖励的方法
  - 返回用的是yield以便多次生成: horizon：时间段长度，N: episode个数
    -  ob_adj: 当前分子图邻接矩阵 horizon
    - ob_node: 当前分子图结点特征 horizon
    - ac : 在当前分子图条件下的动作 horizon
    - prevac: 得到当前分子图之前的动作 horizon
    - rew:  当前ob执行ac后得到的奖励 horizon
    - vpred: 当前ob的V horizon
    - nextvpred: 当前ob执行ac后的ob的V horizon
    - news:  如果为true证明该分子图是个新生成的分子 horizon
    - ob_adj_final: 这个时间段中最终生成分子的邻接矩阵 N
    - ob_node_final: 这个时间段中最终生成分子的结点特征 N
    - ep_rets: 时间段中完整episode的奖励return N
    - ep_lens: 时间段中完整episode的动作长度 N
    - ep_lens_valid: 时间段中完整episode的有效动作长度 N
    - ep_final_rew: 时间段中完整episode的进入最终状态的奖励 N
    - ep_final_rew_stat：时间段中完整episode的进入最终状态信息 N
    - ep_rets_env：时间段中完整episode的环境奖励return N
    - ep_rets_d_step： 时间段中完整episode的中间判别奖励return N
    - ep_rets_d_final：时间段中完整episode的最终判别奖励return N

- add_vtarg_and_adv(seg, gamma, lam)
  - 给之前返回的时间段添加vtarg与adv，在此步骤中通过gae评估v与q与adv
  - seg: 时间段
  - gamma：衰减系数
  - lam: GAE参数 在0-1之间
  - seg["adv"]: advantage (Q(ob,ac)-V(ob))
  - seg["tdlamret"]: Q(ob,ac)

- learn(args, env, policy_fn, timesteps_per_actorbatch, clip_param, entcoeff, optim_epochs, optim_stepsize, optim_batchsize, gamma, lam, max_timesteps=0, max_episodes=0, max_iters=0, max_seconds=0, callback=None, adam_epsilon=1e-5,schedule='constant',writer = None)
  - 主要函数，开始训练，并输出最终生成分子
  - args: 运行参数
  - env: gym环境
  - policy_fn: 策略生成函数
  - clip_param: PPO超参裁剪参数
  - entcoeff: 在PPOloss计算中要用到pi分布的熵，系统希望actor可以有更多的选择，因此熵越大越好，但这个熵也有限度，通过entcoeff(熵权系数)来控制熵的影响程度。
  - optim_epochs: 优化epoch数
  - optim_stepsize: 优化梯度方向步长
  - optim_batchsize: 优化batch大小
  - gamma, lam: 用于adv评估，上一个函数的参数
  - max_timesteps=0, max_episodes=0, max_iters=0, max_seconds=0  时间控制
  - callback: 回调函数
  - adam_epsilon=1e-5 :  变量adam优化系数
  - schedule: 决定步长是常量还是随着时间递减
  - writer：输出信息

