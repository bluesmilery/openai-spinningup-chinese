================
绘制结果
================

Spinning Up有一个简单的绘图工具可以用于解释结果。使用下述代码运行：

.. parsed-literal::

    python -m spinup.run plot [path/to/output_directory ...] [--legend [LEGEND ...]] 
        [--xaxis XAXIS] [--value [VALUE ...]] [--count] [--smooth S]
        [--select [SEL ...]] [--exclude [EXC ...]]


**位置参数：**

.. option:: logdir

    *strings*。你可以使用多个日志目录下的数据来画图（或者只使用日志目录的前缀，绘图仪将在内部自动补全目录名）。Logdirs将被以递归的方式来搜索实验输出。

    .. admonition:: You Should Know

        内置的自动补全功能非常方便！假设您已经运行了多个实验，目的是比较不同算法之间的性能，从而产生以下日志目录结构：

        .. parsed-literal::

            data/
                bench_algo1/
                    bench_algo1-seed0/
                    bench_algo1-seed10/
                bench_algo2/
                    bench_algo2-seed0/
                    bench_algo2-seed10/

        您可以轻松生成比较algo1和algo2的图表：

        .. parsed-literal::

            python spinup/utils/plot.py data/bench_algo

        依靠自动补全来找到 ``data/bench_algo1`` 和 ``data/bench_algo2`` 。

**可选参数：**

.. option:: -l, --legend=[LEGEND ...]

    *strings*。指定图例的可选方法。默认图例将自动使用 ``config.json`` 文件中的 ``exp_name`` ，除非你使用这个参数另外指定。仅当您为将要绘制的每个目录提供名称时，此方法才有效。 （注意：这可能与您提供的logdir参数的数量不同！回想一下绘图仪查找logdir参数时的自动填充：给定的logdir前缀可能有多个匹配，那么您需要为每一个匹配上的logdir提供一个图例字符串——除非您通过选择或排除规则（下方有介绍）删除其中一些作为候选项。）

.. option:: -x, --xaxis=XAXIS, default='TotalEnvInteracts'

    *string*。选择数据中的哪个列用作x轴。

.. option:: -y, --value=[VALUE ...], default='Performance'

    *string*。选择数据中的哪些列画在y轴上。提交多个值将生成多个图。默认为 ``Performance`` ，但是它不是任何算法的实际输出。相反， ``Performance`` 指的是 ``AverageEpRet`` （对于on-policy算法的正确性能度量）或者是 ``AverageTestEpRet`` （对于off-policy算法的正确性能度量）。绘图仪将自动找出为每个logdir画图所需的 ``AverageEpRet`` 或 ``AverageTestEpRet`` 。

.. option:: --count

    可选标志。默认情况下，绘图仪显示的y值是使用同一个 ``exp_name`` 的所有结果的平均值，这通常是一组仅有随机种子变化的相同实验。但是如果你想分别看到所有这些曲线，请使用 ``--count`` 标志。

.. option:: -s, --smooth=S, default=1
    
    *int*。通过在固定窗口内平均来平滑数据。此参数表示窗口的宽度。

.. option:: --select=[SEL ...]

    *strings*。可选的选择规则：绘图仪将仅显示来自包含这些子串的logdir的曲线。

.. option:: --exclude=[EXC ...]

    *strings*。可选的排除规则：绘图仪将仅显示来自不包含这些子串的logdir的曲线。
