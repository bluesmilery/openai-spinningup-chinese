===================
运行实验
===================


.. contents:: 目录

感受深度强化学习的最佳方法之一是运行算法并查看它们在不同任务中的执行情况。而Spinning Up代码库使得小规模（本地）实验变得更加容易。在本节中，我们将讨论两种使用它们的方式：从命令行启动或者通过脚本中的函数调用来使用。


从命令行启动
===============================

Spinning Up附有 ``spinup/run.py`` 文件。这是一个非常方便的工具，它可以让您从命令行轻松启动任何算法（超参数可随意设置）。它还可以作为观察训练好的策略以及绘图等实用工具的轻量级封装器，不过我们不会在这个页面上讨论这个功能（有关这些细节，请参阅 `实验输出`_ 和 `绘图`_ 部分）。


.. _`实验输出`: ../user/saving_and_loading.html
.. _`绘图`: ../user/plot.html

从命令行运行Spinning Up算法的标准方法是

.. parsed-literal::

    python -m spinup.run [algo name] [experiment flags]

例如：

.. parsed-literal::

    python -m spinup.run ppo --env Walker2d-v2 --exp_name walker

.. admonition:: 详细的快速入门指南

    .. parsed-literal::

        python -m spinup.run ppo --exp_name ppo_ant --env Ant-v2 --clip_ratio 0.1 0.2 
            --hid[h] [32,32] [64,32] --act tf.nn.tanh --seed 0 10 20 --dt
            --data_dir path/to/data

    在 ``Ant-v2`` Gym环境中运行PPO，各种设置由标志（flag）控制。

    ``clip_ratio`` ， ``hid`` 和 ``act`` 是设置一些算法超参数的标志。您可以为超参数提供多组值以运行多组实验。阅读文档来查看可以设置的超参数（单击此处查看 `PPO文档`_ ）。

    ``hid`` 和 ``act`` 是 `特殊缩写标志`_ ，用于设置由算法训练的神经网络的隐层大小和激活函数。

    ``seed`` 标志会设置随机数生成器的种子。 RL算法具有很高的方差，因此尝试使用多个不同的种子来了解性能如何变化。

    ``dt`` 标志确保保存目录名称中包含时间戳（否则不会包含，除非你在 ``spinup/user_config.py`` 中设置 ``FORCE_DATESTAMP=True`` ）。

    ``data_dir`` 标志允许您设置结果保存到哪个文件夹。默认值由 ``spinup/user_config.py`` 中的 ``DEFAULT_DATA_DIR`` 设置，默认值为 ``spinningup`` 文件夹中的子文件 ``data`` （除非你改变它）。

    `保存目录名称`_ 基于 ``exp_name`` 和任何使用了多组值的标志。目录名称中将显示标志的缩写，而不是完整标志名称。用户可以在标志后的方括号中提供缩写，例如 ``--hid[h]`` ;否则会使用是标志的子串作为缩写（ ``clip_ratio`` 变成 ``cli`` ）。为了说明，使用 ``clip_ratio=0.1`` ， ``hid=[32,32]`` 和 ``seed=10`` 运行的保存目录将是：

    .. parsed-literal::

        path/to/data/YY-MM-DD_ppo_ant_cli0-1_h32-32/YY-MM-DD_HH-MM-SS-ppo_ant_cli0-1_h32-32_seed10


.. _`PPO文档`: ../algorithm/ppo.html#spinup.ppo
.. _`特殊缩写标志`: ../user/escape.html＃shortcut-flags
.. _`保存目录名称`: ../user/escape.html＃where-results-are-saved


使用命令行设置超参数
---------------------------------------------

每个算法中的每个超参数都可以直接从命令行进行控制。如果 ``kwarg`` 是算法函数调用的有效关键字参数，那么你可以使用标志 ``--kwarg`` 为其设置值。要找出哪些关键字参数是可用的，请参阅算法的文档页面，或尝试

.. parsed-literal::

    python -m spinup.run [algo name] --help

来查看文档。

.. admonition:: 你所应该知道的

    参数值在使用之前会通过 ``eval()`` 传入，因此您可以直接从命令行描述一些函数和对象。例如：

    .. parsed-literal::

        python -m spinup.run ppo --env Walker2d-v2 --exp_name walker --act tf.nn.elu

    将 ``tf.nn.elu`` 设置为激活函数。

.. admonition:: 你所应该知道的

    对于采用dict值的kwargs有一些很好的处理。不必须像下面这样提供

    .. parsed-literal::

        --key dict(v1=value_1, v2=value_2)

    你可以这样

    .. parsed-literal::

        --key:v1 value_1 --key:v2 value_2 

    它们会得到相同的结果。


一次启动多组实验
--------------------------------------

您可以简单地通过为给定参数提供多个值来 **串行** 启动多组实验。（每种可能的值组合都会启动一次实验）

例如，要使用不同的随机种子（0,10和20）而其他参数相同，请执行以下操作：

.. parsed-literal::

    python -m spinup.run ppo --env Walker2d-v2 --exp_name walker --seed 0 10 20

实验不是并行启动的，因为它们使用了足够多的系统资源，同时执行多个实验并不会获得加速。


特殊标志
-------------

一些标志会得到特殊对待。


环境标志
^^^^^^^^^^^^^^^^

.. option:: --env, --env_name

    *string*。OpenAI Gym中的环境名称。所有Spinning Up中的算法都被实现为可以接受 ``env_fn`` 作为参数的函数，其中 ``env_fn`` 必须是构建RL环境副本的可调用函数。由于最常见的用例是Gym环境，但是所有这些都是通过 ``gym.make(env_name)`` 构建的，我们允许你在命令行只需指定 ``env_name`` （或简称 ``env`` ），它会被转换为lambda函数，构建正确的gym环境。


缩写标志
^^^^^^^^^^^^^^

一些算法参数相对较长，我们为它们启用了缩写：

.. option:: --hid, --ac_kwargs:hidden_sizes

    *list of ints*。设置神经网络中隐藏层的大小（策略和值函数）。

.. option:: --act, --ac_kwargs:activation

    *tf op*。actor和critic中神经网络的激活函数。

这些标志对所有当前的Spinning Up算法都有效。


配置标志
^^^^^^^^^^^^

这些标志不是任何算法的超参数，但会以某种方式更改实验配置。

.. option:: --cpu, --num_cpu

    *int*。如果设置了此标志，那么将通过MPI连接的这些进程启动实验，每个进程一个。有些算法适合这种并行化，但不是全部算法都适合。如果您尝试为不兼容的算法设置 ``num_cpu`` > 1，则会引发错误。您还可以设置 ``--num_cpu auto`` ，它将自动使用机器上所有可用的CPU。

.. option:: --exp_name

    *string*。实验名称。这用于命名每个实验的保存目录。默认为 "cmd" + [算法名称]。

.. option:: --data_dir

    *path*。为一次实验或一组实验设置基本保存目录。如果没有给出，将使用 ``spinup/user_config.py`` 中的 ``DEFAULT_DATA_DIR`` 值。

.. option:: --datestamp

    *bool*。在实验的保存目录名称中包含日期和时间。


结果在哪里保存
-----------------------

一次实验的结果会存储在

::

    data_dir/[outer_prefix]exp_name[suffix]/[inner_prefix]exp_name[suffix]_s[seed]

其中

* ``data_dir`` 是 ``--data_dir`` 标志的值（如果没有给出 ``-data_dir`` ，则使用 ``spinup/user_config.py`` 中的 ``DEFAULT_DATA_DIR`` ）
* 如果设置 ``--datestamp`` 标志， ``outer_prefix`` 则是一个 ``YY-MM-DD_`` 的时间戳，否则没有
* 如果设置 ``--datestamp`` 标志， ``inner_prefix`` 则是一个 ``YY-MM-DD_HH-MM-SS-`` 的时间戳，否则没有
* ``suffix`` 是一个基于实验超参数的特殊字符串


后缀（suffix）是如何确定的？
^^^^^^^^^^^^^^^^^^^^^^^^^

如果您一次运行多组实验，那就会包含后缀，并且它们仅包含对实验不同的超参数的引用（除了随机种子）。这么做的目的是确保类似实验的结果（除了种子之外共享所有参数的结果）被存放在同一文件夹中。

通过将超参数的 *缩写* 与它们的值组合来构造后缀，其中缩写是 1）从超参数名称自动构造 或 2）由用户提供。用户可以在kwarg标志之后用方括号写入来提供缩写。

例如：

.. parsed-literal::

    python -m spinup.run ddpg --env Hopper-v2 --hid[h] [300] [128,128] --act tf.nn.tanh tf.nn.relu

在这里， ``--hid`` 标志由 **用户提供了缩写** ``h`` 。用户没有给出 ``--act`` 标志的缩写，因此将自动构造一个。

在这种情况下产生的后缀是：

.. parsed-literal::
    _h128-128_ac-actrelu
    _h128-128_ac-acttanh
    _h300_ac-actrelu
    _h300_ac-acttanh

注意， ``h`` 是由用户给出的。 ``ac-act`` 缩写是由 ``ac_kwargs:activation`` （ ``act`` 标志的真实名称）构成的。


附加内容
---------------

.. admonition:: 你其实并不需要知道这个

    每个算法都位于一个文件 ``spinup/algos/ALGO_NAME/ALGO_NAME.py`` 中，这些文件可以直接从命令行运行，并带有一组有限的参数（其中一些参数不同于 ``spinup/run.py`` ）。然而，单个算法文件中的命令行支持基本上是残留的，这 **不是** 推荐的方式。

    这个文档页面不会描述那些命令行调用， *只会* 在``spinup/run.py`` 文件中描述。


从脚本中启动
======================

每个算法都被实现为python函数，可以直接从 ``spinup`` 包导入，例如

>>> from spinup import ppo

有关参数的完整说明，请参阅每个算法的文档页面。这些方法可用于设置专门的自定义实验，例如：

.. code-block:: python

    from spinup import ppo
    import tensorflow as tf
    import gym

    env_fn = lambda : gym.make('LunarLander-v2')

    ac_kwargs = dict(hidden_sizes=[64,64], activation=tf.nn.relu)

    logger_kwargs = dict(output_dir='path/to/output_dir', exp_name='experiment_name')

    ppo(env_fn=env_fn, ac_kwargs=ac_kwargs, steps_per_epoch=5000, epochs=250, logger_kwargs=logger_kwargs)


使用ExperimentGrid
--------------------

在机器学习研究中，用不同的参数运行同一算法通常很有用。Spinning Up使用一个简单的工具来实现这一点，称为 `ExperimentGrid`_ 。

考虑 ``spinup/examples/bench_ppo_cartpole.py`` 中的示例：

.. code-block:: python
   :linenos:

    from spinup.utils.run_utils import ExperimentGrid
    from spinup import ppo
    import tensorflow as tf

    if __name__ == '__main__':
        import argparse
        parser = argparse.ArgumentParser()
        parser.add_argument('--cpu', type=int, default=4)
        parser.add_argument('--num_runs', type=int, default=3)
        args = parser.parse_args()

        eg = ExperimentGrid(name='ppo-bench')
        eg.add('env_name', 'CartPole-v0', '', True)
        eg.add('seed', [10*i for i in range(args.num_runs)])
        eg.add('epochs', 10)
        eg.add('steps_per_epoch', 4000)
        eg.add('ac_kwargs:hidden_sizes', [(32,), (64,64)], 'hid')
        eg.add('ac_kwargs:activation', [tf.tanh, tf.nn.relu], '')
        eg.run(ppo, num_cpu=args.cpu)

在创建ExperimentGrid对象之后，将参数添加到其中

.. parsed-literal::

    eg.add(param_name, values, shorthand, in_name)

其中 ``in_name`` 强制参数出现在实验名称中，即使它在所有实验中具有相同的值。

添加完所有参数后，

.. parsed-literal::

    eg.run(thunk, **run_kwargs)

通过向函数 ``thunk`` 提供kwargs配置，在网格中运行所有实验（每个有效配置一个实验）。 ``ExperimentGrid.run`` 使用名为 `call_experiment`_ 的函数启动 ``thunk`` ， ``**run_kwargs`` 指定 ``call_experiment`` 的行为。更详细信息请参阅 `文档页面`_ 。

除了缺少缩写方式的kwargs（你不能在 ``ExperimentGrid`` 中使用 ``hid`` 来代替 ``ac_kwargs:hidden_​​sizes`` ）， ``ExperimentGrid`` 的运行方式与从命令行运行一样。（事实上​​， ``spinup.run`` 在在其内部就使用了 ``ExperimentGrid`` ）


.. _`ExperimentGrid`: ../utils/run_utils.html#experimentgrid
.. _`call_experiment`: ../utils/run_utils.html#spinup.utils.run_utils.call_experiment
.. _`文档页面`: ../utils/run_utils.html#experimentgrid
