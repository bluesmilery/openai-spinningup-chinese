==========
算法
==========


.. contents:: 目录


包含的内容
===============

Spinning Up包中实现了以下算法：

- `Vanilla Policy Gradient`_ (VPG)
- `Trust Region Policy Optimization`_ (TRPO)
- `Proximal Policy Optimization`_ (PPO)
- `Deep Deterministic Policy Gradient`_ (DDPG)
- `Twin Delayed DDPG`_ (TD3)
- `Soft Actor-Critic`_ (SAC)

它们都是用MLP (non-recurrent) actor-critics实现的，使它们适用于完全观察的，非基于图像的RL环境，例如 `Gym Mujoco`_ 环境。


.. _`Vanilla Policy Gradient`: ../algorithms/vpg.html
.. _`Trust Region Policy Optimization`: ../algorithms/trpo.html
.. _`Proximal Policy Optimization`: ../algorithms/ppo.html
.. _`Deep Deterministic Policy Gradient`: ../algorithms/ddpg.html
.. _`Twin Delayed DDPG`: ../algorithms/td3.html
.. _`Soft Actor-Critic`: ../algorithms/sac.html
.. _`Gym Mujoco`: https://gym.openai.com/envs/#mujoco


为什么是这些算法？
=====================

我们在这个软件包中选择了核心的深度强化学习算法，以反映该领域近期历史中有用的思想进展，特别是两个算法——PPO和SAC——在策略学习算法领域能够在可靠性和样本效率方面接近SOTA。他们还揭示了在深度强化学习中设计和使用算法时所做的一些权衡。

On-Policy算法
------------------------

Vanilla Policy Gradient是深度强化学习领域中最基本的入门级算法，因为它先于深度强化学习出现。VPG的核心要素可以追溯到80年代末90年代初。它开创了一系列研究，最终产生了更强大的算法，例如TRPO以及后来的PPO。

这一系列工作的一个关键特点是，所有这些算法都是 *on-policy* 的：也就是说，它们不使用会降低样本效率的旧数据。但这也是有充分理由的：这些算法直接优化您在意的目标——策略性能——并且数学上也需要on-policy数据来计算更新。因此，这一系列算法在权衡样本效率和算法稳定性时更倾向于后者——但您可以看到技术的进展（从VPG到TRPO再到PPO）弥补了样本效率的不足。

Off-Policy算法
-------------------------

DDPG是与VPG类似的基础算法，虽然它更年轻——推导出DDPG的确定策略梯度(deterministic policy gradients)理论，直到2014年才出版。DDPG与Q-learning算法密切相关，它同时学习Q-function和策略，它们不断更新互相改善。

像DDPG和Q-Learning这样的算法是 *off-policy* 的，因此它们能够非常有效地利用旧数据。它们通过利用Bellman方程优化来获得好处，这种好处是可以训练一个Q-function以满足使用 *任何* 环境交互数据（只要有足够的来自环境中的高回报区域的经验）。

但问题是无法保证只要做好满足Bellman方程式的事情就会带来很好的策略表现。 *凭经验* 可以获得很好的性能——当它发生时，样本效率很好——但缺乏保证使得这类算法可能会变得脆弱和不稳定。TD3和SAC是DDPG的后代，它们利用各种见解来缓解这些问题。


代码格式
===========

Spinning Up中的所有实现都遵循标准模板。它们分为两个文件：一个算法文件，其中包含算法的核心逻辑，以及一个核心文件，其中包含运行算法所需的各种实用工具。

算法文件
------------------

算法文件始终以一个experience buffer对象的类定义开始，该对象用于存储Agent与环境的交互信息。

接下来，有一个运行算法的函数，执行以下任务（按此顺序）：

    1) 日志设置

    2) 随机种子设定

    3) 环境实例化

    4) 设定用于计算图的占位符

    5) 通过将 ``actor_critic`` 函数作为参数传递给算法函数来构建actor-critic计算图

    6) 实例化experience buffer

    7) 构建特定于算法的损失函数和诊断的计算图

    8) 设定训练操作

    9) 设定TF Session并初始化参数

    10) 设置通过logger来保存模型

    11) 定义运行算法主循环所需的函数（例如核心更新函数，获取动作函数和测试Agent函数，具体取决于算法）

    12) 运行算法的主循环：

        a) 在环境中运行Agent

        b) 根据算法的主要方程定期更新Agent的参数

        c) 记录关键性能指标并保存Agent

最后，对于直接在命令行将算法运行在Gym环境中提供一些支持。

核心文件
-------------

核心文件不像算法文件那样严格遵守模板，但也有一些相似的结构：

    1) 与制作和管理占位符相关的函数

    2) 用于构建与特定算法的 ``actor_critic`` 方法相关的计算图部分的函数

    3) 任何其他有用的函数

    4) 与算法兼容的MLP actor-critic的实现，其中策略和值函数都由简单的MLP表示。
