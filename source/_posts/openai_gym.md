---
title: openai gym环境介绍
date: 2022-10-28 18:21:00
categories: ["学习"]
tags: ["强化学习","gym"]
mathjax: true
---

# openai gym环境



## 初始化环境

```python
import gym
env = gym.make('CartPole-v0')
```

## 环境交互

```python
import gym
env = gym.make("LunarLander-v2", render_mode="human")
env.action_space.seed(42)

observation, info = env.reset(seed=42)

for _ in range(1000):
    observation, reward, terminated, truncated, info = env.step(env.action_space.sample())

    if terminated or truncated:
        observation, info = env.reset()

env.close()
```

<img src="https://user-images.githubusercontent.com/15806078/153222406-af5ce6f0-4696-4a24-a683-46ad4939170c.gif" alt="https://user-images.githubusercontent.com/15806078/153222406-af5ce6f0-4696-4a24-a683-46ad4939170c.gif" style="zoom:50%;" />

<!--more-->

环境通过提供`env.action_space`属性来定义有效的动作。类似的，观测值的格式通过`env.observation_space`来确定。在上述的例子中通过`env.action_space.sample()`对环境中的动作进行随机采样。注意需要分别为动作空间与环境设置seed种子来确保可以得到可复制的样本。

## 检查API-规则

如果我们写了一个自定义的环境，想要对它进行完整性检测可以这样：

```python
from gym.utils.env_checker import check_env
check_env(env)
```

当环境没有遵守Gym API时上述代码会报错。它也会同时进行一些最佳实践的检测，可以通过设置`warn=False`进行关闭。默认不会对render方法进行检测，可以通过`skip_render_check=False`进行设置。

## Spaces 空间

Gym通过定义空间来确定动作与观测值的有效范围。每个环境都应该具有`action_space`与`observation_space`，他们都是 `Space`的子类的实例。Gym中有几种不同的`Space`类型：

- Box：表述一个n维连续空间。通过上界与下界可以对它进行限制
- Discrete：表述一个离散空间，其中{0, 1, ..., n-1}为动作或观测值的可选值。这些值也可以通过添加一个可选参数被偏移到{a, a+1, ..., a+n-1}。
- Dict：表示一个简单空间的字典
- Tuple：表示一个简单空间的元组
- MultiBinary：创建一个n维二进制空间。参数n可以是一个数字也可以是一个数字list。
- MultiDiscrete：包含一系列Discrete空间，其中每个元素的动作数可以不同。

```python
>>> from gym.spaces import Box, Discrete, Dict, Tuple, MultiBinary, MultiDiscrete
>>>
>>> observation_space = Box(low=-1.0, high=2.0, shape=(3,), dtype=np.float32)
>>> observation_space.sample()
[ 1.6952509 -0.4399011 -0.7981693]
>>>
>>> observation_space = Discrete(4)
>>> observation_space.sample()
1
>>>
>>> observation_space = Discrete(5, start=-2)
>>> observation_space.sample()
-2
>>>
>>> observation_space = Dict({"position": Discrete(2), "velocity": Discrete(3)})
>>> observation_space.sample()
OrderedDict([('position', 0), ('velocity', 1)])
>>>
>>> observation_space = Tuple((Discrete(2), Discrete(3)))
>>> observation_space.sample()
(1, 2)
>>>
>>> observation_space = MultiBinary(5)
>>> observation_space.sample()
[1 1 1 0 1]
>>>
>>> observation_space = MultiDiscrete([ 5, 2, 2 ])
>>> observation_space.sample()
[3 0 0]
```

## Wrappers 包装器

可以通过Wrappers对一个已有环境进行简单的更改，而不用改变原环境的底层代码。Wrappers也可以通过链式组合他们的影响。大多数通过`gym.make`生成的环境已经默认被包装过。

要包装一个环境，首先初始化一个基本环境，之后就可以将这个环境与想更改的参数传递给包装器的构造器：

```python
>>> import gym
>>> from gym.wrappers import RescaleAction
>>> base_env = gym.make("BipedalWalker-v3")
>>> base_env.action_space
Box([-1. -1. -1. -1.], [1. 1. 1. 1.], (4,), float32)
>>> wrapped_env = RescaleAction(base_env, min_action=0, max_action=1)
>>> wrapped_env.action_space
Box([0. 0. 0. 0.], [1. 1. 1. 1.], (4,), float32)
```

包装器可以做以下三件事：

- 改变传给基本环境的动作
- 改变从基本环境中获得的观测值
- 改变从基本环境中得到的奖励

这些包装器可以通过继承`ActionWrapper`, `ObservationWrapper` , 或`RewardWrapper` 来实现。

如果需要用更复杂的方式实现wrapper（e.g. 根据`info`中的数据改变奖励）。这样的wrappers可以通过继承`Wrapper`类进行实现。Gym提供了很多常用的wrapper：

- TimeLimit：为环境设置一个最大的时间步数
- ClipAction：对动作空间上的动作进行裁剪
- RescaleAction：将动作调整到指定间隔内
- TimeAwareObervation：在观测中增加时间轴信息。

想获得基本环境可以使用包装环境的unwrapped属性，对于基本环境，unwrapped属性会返回自身。

```python
>>> wrapped_env
<RescaleAction<TimeLimit<BipedalWalker<BipedalWalker-v3>>>>
>>> wrapped_env.unwrapped
<gym.envs.box2d.bipedal_walker.BipedalWalker object at 0x7f87d70712d0>
```

## 试玩一个环境

可以`gym.utils.play`中的`play`方法来试玩一个环境。

```python
from gym.utils.play import play
play(gym.make('Pong-v0'))
```

可以打开一个环境窗口来通过键盘控制智能体。

需要有一个键盘与动作的映射，映射的类型应当为`dict[tuple[int], int | None]`它将key与动作一一对应。例如：如果同时按住`w`键与`空格`就可以执行动作`2`,那么`key_to_action`字典应当为

```python
{
    # ...
    (ord('w'), ord(' ')): 2,
    # ...
}
```

以下是一个更完整的例子，可以用左右箭头控制CartPole环境

```python
import gym
import pygame
from gym.utils.play import play
mapping = {(pygame.K_LEFT,): 0, (pygame.K_RIGHT,): 1}
play(gym.make("CartPole-v0"), keys_to_action=mapping)
```

