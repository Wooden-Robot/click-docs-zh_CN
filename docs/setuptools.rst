.. _setuptools-integration:

Setuptools 集成
======================

编写命令行程序时，建议将它们编写为 setuptools 型分发的模块，而不是使用 Unix shebangs。

你为何要这么做呢?  有如下几个原因:

1.  传统方法的问题之一是Python解释器加载的第一个模块名称不正确。这可能像一个小问题，
    但具有相当重要的意义。

    第一个模块没有被其实际名称调用，但解释器将其重命名为 ``__main__`` 虽然这是一个完全有
    效的名字，但这意味着如果另一段代码想要从该模块导入，它将在其真实名称下第二次触发导入，
    并且突然间您的代码将被导入两次。

2.  不是在所有的平台上，程序都是易于执行的。在Linux和OS X上，您可以将注释添加到文件的开头
    (开头这么写 ``#!/usr/bin/env python``) ，这样您的脚本就像一个可执行文件（假设它具有可执行位设置）。 
    但是，这不适用于Windows系统。在Windows上，您可以将解释器与文件扩展名关联起来()
    (就像任何以 ``.py`` 结尾的文件，通过Python解释器执行) ，如果在要在virtualenv中使用该脚本，
    则会遇到问题。

    实际上，在 virtualenv 中运行脚本也是 OS X 和 Linux 一个问题。使用传统方法，您需要激活整个 virtualenv，
    以便使用正确的Python解释器。这对用户不是非常友好。

3.  主要的技巧只有在脚本是在Python模块时才有效。如果你的应用程序变得太大，你想开始
    使用一个包，你会遇到问题。

介绍
------------

要将脚本与setuptools捆绑在一起，您只需要Python程序包和 ``setup.py`` 文件。

想象一下这个目录结构::

    yourscript.py
    setup.py

``yourscript.py`` 的内容为:

.. click:example::

    import click

    @click.command()
    def cli():
        """Example script."""
        click.echo('Hello World!')

``setup.py`` 的内容为::

    from setuptools import setup

    setup(
        name='yourscript',
        version='0.1',
        py_modules=['yourscript'],
        install_requires=[
            'Click',
        ],
        entry_points='''
            [console_scripts]
            yourscript=yourscript:cli
        ''',
    )

神奇的是 ``entry_points`` 参数.  下面``console_scripts``, 每行标识一个控制台脚本。第一部分是在等号 (``=``) 前面的应该生成的脚本名称。第二部分是在冒号后面(``:``)的导入路径。 

仅此而已。

测试脚本
------------------

为了测试脚本，你可以创建一个新的virtualenv然后安装你的包::

    $ virtualenv venv
    $ . venv/bin/activate
    $ pip install --editable .

之后，您的命令应该可用:

.. click:run::

    invoke(cli, prog_name='yourscript')

包中的脚本
-------------------

如果您的脚本正在增多，并且您希望切换到包含在Python包中的脚本，那么所需的更改很少。我们假设你的目录结构变成这样::

    yourpackage/
        __init__.py
        main.py
        utils.py
        scripts/
            __init__.py
            yourscript.py

在这种情况下，在 ``setup.py`` 文件中，使用 ``pacckages`` 代替 ``py_modules`` ，同时自动包会找到 setuptools的支持。除此之外，还建议它含其他包数据。

如下这些是对 ``setup.py`` 文件内容的修改::

    from setuptools import setup, find_packages

    setup(
        name='yourpackage',
        version='0.1',
        packages=find_packages(),
        include_package_data=True,
        install_requires=[
            'Click',
        ],
        entry_points='''
            [console_scripts]
            yourscript=yourpackage.scripts.yourscript:cli
        ''',
    )
