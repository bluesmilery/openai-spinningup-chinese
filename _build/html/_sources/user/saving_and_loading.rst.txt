==================
实验输出
==================


.. contents:: 目录

在本节中，我们将介绍

- Spinning Up的算法实现会输出什么，
- 它们存储的格式以及它们的组织方式，
- 存储它们的位置以及如何更改，
- 以及如何加载和运行经过训练的策略。

.. admonition:: You Should Know

    Spinning Up的算法实现目前无法恢复训练已经被部分训练的agents。如果您认为此功能很重要，请告诉我们---或者将其视为一个具有挑战的项目！


算法输出
=================

每个算法都能保存训练运行的超参数配置，学习进度，训练好的agents和值函数，以及环境副本（如果可能的话，以便于同时加载agent和环境）。输出目录包含以下内容：

* ``simple_save/`` 。包含重建训练好的agent和值函数所需的所有东西的目录。( `详见下文`_ )
* ``config.json`` 。包含启动训练时所用到的参数的详尽描述。如果你传递了一些无法解析为JSON的东西，它应该由记录器来处理，并且会以字符串来呈现配置文件。注意：这仅用于记录保存。目前还不支持从配置文件启动实验。
* ``progress.txt`` 。一个使用制表符作为分隔值文件，包含记录器在整个训练过程中记录的指标记录。例如， ``Epoch`` ， ``AverageEpRet`` 等。
* ``vars.pkl`` 。一个序列化文件，包含应该存储的算法状态的任何内容。目前，所有算法仅使用它来保存环境的副本。

.. admonition:: You Should Know

    有时保存环境会失败是因为环境无法被序列化，并且 ``vars.pkl`` 文件是空的。这对于老版本的Gym中的Gym Box2D环境来说是个问题，它不能以这种方式保存。


.. _`详见下文`:

``simple_save`` 目录中包含一下内容：

* ``variables/`` 。该目录内含有Tensorflow Saver的输出内容。关于 `Tensorflow SavedModel`_ 请见文档。
* ``model_info.pkl`` 。包含加载已保存的模型后对模型进行解包所需的信息。
* ``saved_model.pb`` 。一个protocol buffer文件，用于 `Tensorflow SavedModel`_.

.. admonition:: You Should Know

    这里唯一一个你必须“手工”使用的文件是 ``config.json`` 文件。我们的agent测试工具将从 ``simple_save /`` 目录和 ``vars.pkl`` 文件中加载内容，我们的绘图仪解释 ``progress.txt`` 的内容，这些是用于连接这些产出的正确工具。但是没有用于 ``config.json`` 的工具，所以如果你忘记了你运行实验的超参数，你可以仔细查看这个文件。

.. _`Tensorflow SavedModel`: https://github.com/tensorflow/tensorflow/blob/master/tensorflow/python/saved_model/README.md


保存目录的位置
=======================

默认情况下，实验结果将保存在与Spinning Up软件包相同的目录中，名为 ``data`` 的文件夹中：

.. parsed-literal::

    spinningup/
        **data/**
            ...
        docs/
            ...
        spinup/
            ...
        LICENSE
        setup.py

您可以通过修改 ``spinup/user_config.py`` 中的 ``DEFAULT_DATA_DIR`` 来更改结果的默认保存目录。


加载和运行经过训练的策略
====================================


如果环境成功保存了
---------------------------------

对于环境与agent一起成功保存的情况，可以使用以下方法观察受过训练的agent在环境中的行为：

.. parsed-literal::

    python -m spinup.run test_policy path/to/output_directory

这里有几个选项标志：

.. option:: -l L, --len=L, default=0

    *int*。测试 episode / trajectory / rollout 的最大长度。默认值为0表示没有最大episode长度，episode只在agent到达环境的终止状态时才会结束。（注意：设置L=0不会阻止TimeLimit包装器包含的Gym环境在达到预设的最大episode长度时结束）

.. option:: -n N, --episodes=N, default=100

    *int*。运行agent的测试episode数。

.. option:: -nr, --norender

    不要将测试episode渲染到屏幕上。在这种情况下， ``test_policy`` 只会打印episode的回报和长度。（使用案例：渲染器减慢了测试过程，您只是想快速了解agent的执行情况，因此您不必特别注意观察它。）

.. option:: -i I, --itr=I, default=-1

    *int*。这个选项用于在此程序包中的算法不支持，但它们很容易修改的特殊情况。使用案例：有时在训练中从许多不同时刻观察训练的agent是很好的（例如，在迭代50,100,150等处观看）。记录器可以执行此操作---从这些不同的时刻保存agenmt的快照，以便以后可以运行和观察它们。在这种情况下，您可以使用此标志指定运行哪次迭代。但同样重要的是：默认情况下，算法只保存agent的最新快照，旧快照会被覆盖。

    这个标志的默认值表示“使用最新的快照”。

    要修改算法以便让它生成多个快照，请找到以下行（所有算法中都存在）：

    .. code-block:: python

        if (epoch % save_freq == 0) or (epoch == epochs-1):
            logger.save_state({'env': env}, None)

    将其修改为

    .. code-block:: python

        if (epoch % save_freq == 0) or (epoch == epochs-1):
            logger.save_state({'env': env}, epoch)

    确保还将 ``save_freq`` 设置为合理的值（因为如果它默认为1，那么你的输出目录会被快照迅速填充满，每份快照都会存在一个 ``simple_save`` 文件夹中）。

.. option:: -d, --deterministic

    另一个特例，仅用于SAC。Spinning Up SAC会训练一个随机策略，但使用动作分布的确定性 *均值* 进行评估。 ``test_policy`` 将默认使用SAC训练的随机策略，但您应设置确定性标志以观察确定性均值策略（SAC的正确评估策略）。此标志不用于任何其他算法。


出现Environment Not Found错误
---------------------------------

如果环境未成功保存，您可能会发现 ``test_policy.py`` 报错

.. parsed-literal::

    Traceback (most recent call last):
      File "spinup/utils/test_policy.py", line 88, in <module>
        run_policy(env, get_action, args.len, args.episodes, not(args.norender))
      File "spinup/utils/test_policy.py", line 50, in run_policy
        "page on Experiment Outputs for how to handle this situation."
    AssertionError: Environment not found!

    这看起来像是环境没有被保存，我们无法在其中运行agent。 :( 

    查看实验输出的readthedocs页面，了解如何处理这种情况。

在这种情况下，只要您可以轻松地重建您的环境，观察您的agent执行情况稍微有点痛苦，但并非不可能。在IPython中尝试以下内容：

>>> from spinup.utils.test_policy import load_policy, run_policy
>>> import your_env
>>> _, get_action = load_policy('/path/to/output_directory')
>>> env = your_env.make()
>>> run_policy(env, get_action)
Logging data to /tmp/experiments/1536150702/progress.txt
Episode 0    EpRet -163.830      EpLen 93
Episode 1    EpRet -346.164      EpLen 99
...

使用已经训练好的值函数
-----------------------------

``test_policy.py`` 工具不会帮助你查看已经训练好的值函数，如果你想使用它们，你必须手动处理。查看 `restore_tf_graph`_ 函数的文档，了解具体方法。

.. _`restore_tf_graph`: ../utils/logger.html#spinup.utils.logx.restore_tf_graph
