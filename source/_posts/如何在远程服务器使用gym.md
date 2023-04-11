---
title: 如何在远程服务器使用gym的render()
date: 2022-11-16 21:09:00
categories: ["学习"]
tags: ["强化学习","gym"]
mathjax: true
---

# 如何在远程服务器使用gym的render()

SMB环境github主页：https://github.com/Kautenja/gym-super-mario-bros

```python
from nes_py.wrappers import JoypadSpace
import gym_super_mario_bros
from gym_super_mario_bros.actions import SIMPLE_MOVEMENT
import gym
env = gym_super_mario_bros.make('SuperMarioBros-v0')
env = JoypadSpace(env, SIMPLE_MOVEMENT)
env = gym.wrappers.Monitor(env, "recording")

done = True
for step in range(5000):
    if done:
        state = env.reset()
    state, reward, done, info = env.step(env.action_space.sample())
    env.render()

env.close()
```
<!--more-->

Wrappers是gym环境中为环境设计的包装器，其中`gym.wrappers.Monitor`可以记录下屏幕上gym环境的render，放到第二个参数名的文件夹中。注意其中`recording`文件夹如果已有的话，会报错，建议使用一个新的文件夹。

之后需要在远程服务器上下载Xvbf，用于创造一个虚拟的输出屏幕环境。可以用`sudo apt-get install xvfb`进行安装。

在运行时使用xvfb-run命令：

`xvfb-run -s "-screen 0 640x480x24" python monitor.py`

即可在`recording`文件夹中获得输出的MP4文件。

## Monitor not found

AttributeError: module ‘gym.wrappers‘ has no attribute ‘Monitor‘

出现错误的原因是gym的版本好过高，gym新版本中把Monitor从wrappers里删了。从0.23.0版本就去掉了

因此需要将gym版本降低，或者使用wrappers中的RecordVideo代替。

## Unknown encoder ‘libx264‘

在运行时报错，可以看到与ffmpeg有关，是在输出mp4文件时出问题。

将ffmpeg卸载后重新用conda安装

```shell
conda uninstall ffmpeg
conda install -c conda-forge ffmpeg
```

可以看到录制的文件啦

![GIF 2022-10-28 17-27-42](如何在远程服务器使用gym\GIF 2022-10-28 17-27-42.gif)
