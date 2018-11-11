============
Installation
============

.. contents:: 目录

Spinning Up需要使用Python3，MuJoCo，OpenAI Gym和OpenMPI。

Spinning Up目前仅支持Linux和OSX。可能可以在Windows上安装，但尚未经过测试。（如果您尝试并成功了，请告诉我们您是如何做到的！）


安装Python
=================

我们建议通过Anaconda来安装Python。Anaconda是一个包含Python和许多有用的Python包的库，它同时拥有一个名为conda的环境管理器，使包管理变得非常简单。

请参照Anaconda的 `安装说明`_ 。下载并安装Anaconda 3.x（撰写本文时为3.6）。然后创建一个conda env来管理Spinning Up中使用的包：

.. parsed-literal::

    conda create -n spinningup python=3.6

要从刚刚创建的环境中使用Python，请使用以下命令激活环境：

.. parsed-literal::

    source activate spinningup

.. admonition:: 你所应该知道的

    如果你对Python环境和软件包管理不了解，这些东西很快就会让你感到困惑或不知所措，你可能会在这个过程中遇到一些障碍。（特别地，你应该会遇到类似这种问题，“我刚刚安装了这个东西，但是当我尝试使用它时系统提示没有找到它！”）你可能想要阅读一些关于包管理是什么的清晰解释，为什么它是一个好的想法，以及您通常必须执行哪些命令才能正确使用它。

    `FreeCodeCamp`_ 有一个很好的解释值得一读。这也有一篇关于 `Towards Data Science`_ 的简短描述也很有帮助。最后，如果你是一个非常有耐心的人，你或许想要阅读（枯燥但非常有用的） `Conda文档页面`_ 。


.. _`安装说明`: https://docs.continuum.io/anaconda/install/
.. _`FreeCodeCamp`: https://medium.freecodecamp.org/why-you-need-python-environments-and-how-to-manage-them-with-conda-85f155f4353c
.. _`Towards Data Science`: https://towardsdatascience.com/environment-management-with-conda-python-2-3-b9961a8a5097
.. _`Conda文档页面`: https://conda.io/docs/user-guide/tasks/manage-environments.html


安装MuJoCo和OpenAI Gym
================================

首先，前往 `mujoco-py`_ 的GitHub页面。按照README中的安装说明进行操作，该说明描述了如何安装MuJoCo物理引擎和mujoco-py包（允许在Python中使用MuJoCo）。

.. admonition:: 你所应该知道的

    要使用MuJoCo模拟器，您需要获得 `MuJoCo license`_ 。任何人都可以获得免费30天的许可证，全日制学生可以获得免费1年的许可证。

接下来，前往 `Gym`_ GitHub页面，然后按照README中的"Installing Everything"说明进行操作。

一定要 *先安装MuJoCo和mujoco-py* 再安装Gym，以确保能够正确设置Gym mujoco环境。


.. _`mujoco-py`: https://github.com/openai/mujoco-py/tree/master/mujoco_py
.. _`MuJoCo license`: https://www.roboti.us/license.html
.. _`Gym`: https://github.com/openai/gym


安装OpenMPI
==================

Ubuntu
------

.. parsed-literal::

    sudo apt-get update && sudo apt-get install libopenmpi-dev


Mac OS X
--------

在Mac上安装系统软件包需要 `Homebrew`_ 。 安装Homebrew后，运行以下命令：

.. parsed-literal::

    brew install openmpi


.. _Homebrew: https://brew.sh


安装Spinning Up
======================

.. parsed-literal::

    git clone https://github.com/openai/spinningup.git
    cd spinningup
    pip install -e .


检查您的安装
==================

要查看您是否已成功安装Spinning Up，请尝试在Walker2d-v2环境中运行PPO

.. parsed-literal::

    python -m spinup.run ppo --hid [32,32] --env Walker2d-v2 --exp_name installtest

这大约会运行10分钟，与此同时您可以继续阅读文档。这不会完成对Agent的训练，但会运行足够长的时间以便在结果出现时看到 *某些* 学习进度。

完成训练后，可以观看已经训练好完成的策略的视频

.. parsed-literal::

    python -m spinup.run test_policy data/installtest/installtest_s0

并且可以绘制结果

.. parsed-literal::

    python -m spinup.run plot data/installtest/installtest_s0
