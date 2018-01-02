Python 3 支持
================

.. currentmodule:: click

Click 支持Python 3，但像所有其他命令行实用程序库一样，它受到 Python 3 中的 Unicode 文本模型的影响。文档中的所有示例都是为了能够在Python 2.x和Python 3.3或更高版本上运行而编写的。

目前，强烈建议使用 Python 2 作为 Click 实用程序，除非 Python 3 是一个硬性要求。

.. _python3-limitations:

Python 3 的限制
--------------------

目前, Clik Python 3中的几个问题：

*   Unix中的命令行传统上是以字节为单位的，而不是Unicode。虽然有编码暗示所有这一切，通常有一些情况下，可以打破。最常见的是SSH连接到不同语言环境的机器。

由于缺少对双向代用转义的支持，错误配置的环境目前可能导致Python 3中的各种Unicode问题。这不会在Click中修复！

有关更多信息，请参阅 :ref:`python3-surrogates`.

*   Python 3中的标准输入和输出默认以Unicode模式打开。在某些情况下，Click 必须以二进制模式重新打开流。因为没有标准化的方式来做到这一点，所以这可能并不总是奏效。主要是在测试命令行应用程序时，这可能会成为问题。

    这不支持::

         sys.stdin = io.S tringIO('Input here')
        sys.stdout = io.StringIO()

    你需要这样做::

         input = 'Input  here'
        in_stream = io.BytesIO(input.encode('utf-8'))
        sys.stdin = io.TextIOWrapper(in_stream, encoding='utf-8')
        out_stream = io.BytesIO()
        sys.stdout = io.TextIOWrapper(out_stream, encoding='utf-8')

请记住，在这种情况下，你需要使用 ``out_stream.getvalue()``  而不是 ``sys.stdout.getvalue()`` 如果你想访问缓冲区的内容，因为包装将不会转发该方法。

Python 2 和 3 的差异
--------------------------

通过遵循两种版本的最佳做法，Click 尝试最小化缩减Python 2和Python 3之间的差异。

在Python 2中，以下是正确的：

*   ``sys.stdin``, ``sys.stdout``, 和 ``sys.stderr`` 以二进制模式打开，但在某些情况下，它们支持Unicode输出。单击尝试不能颠覆这一点，但提供支持强制流是基于Unicode的。

*   ``sys.argv`` 总是以字节为基础的。Click将字节传递给所有输入类型，并根据需要进行转换。 The :class:`STRING` 类型自动将正确解码输入值插入尝试最合适的编码字符串。

*   处理文件时，Click将永远不会通过Unicode API，而是使用操作系统的字节API来打开文件。 

在Python 3中，以下是正确的：

*   ``sys.stdin``, ``sys.stdout`` 和 ``sys.stderr`` 默认情况下是基于文本的。当Click需要二进制流时，它会尝试发现基础二进制流。请参阅 :ref:`python3-limitations` 以了解其工作原理。

*   ``sys.argv`` 始终是基于Unicode的。这也意味着Click中类型的输入值的本机类型是Unicode，而不是字节。

    这会导致问题，如果终端设置不正确，Python不会计算出编码。在这种情况下，Unicode字符串将包含编码为代理转义的错误字节。

*   处理文件时，Click将始终使用Unicode文件系统API调用，方法是使用操作系统的报告或猜测的文件系统编码。代理支持文件名，因此应该可以通过 :class:`File` 类型打开文件，即使环境配置错误也是如此。

.. _python3-surrogates:

Python 3 代理处理
---------------------------

在Python 3中单击将执行标准库中的所有Unicode处理，并受其行为影响。在Python 2中，Click单独处理所有的Unicode，这意味着在错误行为上存在差异。

最明显的区别是，在Python 2中，Unicode将“正常工作”，而在Python 3中，它需要特别小心。原因是在Python 3中，编码检测是在解释器中完成的，在Linux和某些其他操作系统上，其编码处理是有问题的。

最大的难题来源是init系统（sysvinit，upstart，systemd等），部署工具（salt，puppet）或cron作业（cron）调用的Click脚本将拒绝工作，除非导出Unicode区域设置。

如果Click遇到这样的环境，它将阻止进一步的执行，迫使你设置一个语言环境。这样做是因为一旦被调用，Click不知道系统的状态，并在Python的Unicode处理加入之前恢复值。



如果你在Python 3中看到这样的错误::

    Traceback (most recent call last):
      ...
    RuntimeError: Click will abort further execution because Python 3 was
      configured to use ASCII as encoding for the environment. Either switch
      to Python 2 or consult http://click.pocoo.org/python3/ for
      mitigation steps.

您正在处理Python 3认为您仅限于ASCII数据的环境。这些问题的解决方案取决于您的计算机运行在哪个区域。

例如，如果你有一台德语的Linux机器，你可以通过导出语言环境来解决这个问题 ``de_DE.utf-8``::

    export LC_ALL=de_DE. utf-8
    export LANG=de_DE.utf-8

如果你在US机器上, 选择的是 ``en_US.utf-8`` 编码。在一些较新的Linux系统上，你也可以尝试一下 ``C.UTF-8`` 作为本地编码::

    export LC_ALL=C.UTF-8
    export LANG=C.UTF-8

在某些系统上，据报道， `UTF-8` 必须写成 `UTF8` ，反之亦然。要查看哪些语言环境支持，您可以调用 
``locale -a``::

    locale -a

您需要在调用Python脚本之前执行此操作。如果你对这个原因感到好奇，你可以参加Python 3 bug跟踪器的讨论:

*   `ASCII 是报错的文件系统默认编码
    <http://bugs.python.org/issue13643#msg149941>`_
*   `使用surrogateescape作为默认报错处理程序
    <http://bugs.python.org/issue19977>`_
*   `Python 3在C语言环境中引发Unicode报错
    <http://bugs.python.org/issue19846>`_
*   `LC_CTYPE = C：pydoc使终端处于不可用状态
    <http://bugs.python.org/issue21398>`_ (这与Click相关，因为寻呼机支持由stdlib pydoc模块提供)

Unicode 常量
----------------

从Click 5.0开始，将会在Python 2中使用未来导入 ``unicode_literals`` 警告。This has been done due to
这是由于导入导致的负面影响而导致的，这是由于将Unicode数据引入到无法处理的API而导致的无意中导致的错误他们。对于这个问题的一些例子，请参阅关于这个github问题的讨论： `python-future#22
<https://github.com/PythonCharmers/python-future/issues/22>`_.

如果您 ``unicode_literals`` 在任何定义Click命令的文件中使用或者调用了单击命令，将会给您一个警告。强烈建议您不要使用 ``Unicode`` 字符串的 ``unicode_literals`` 明确 ``u`` 前缀。

如果您想忽略警告并继续使用 ``unicode_literals`` 自己的危险，可以按如下所示禁用警告::

    import click 
    click.disable_unicode_literals_warning = True
