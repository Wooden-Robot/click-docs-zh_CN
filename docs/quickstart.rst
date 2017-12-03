快速上手
==========

.. currentmodule:: click

你可以直接从 PyPI 中进行安装：::

    pip install click

强烈推荐将其安装在 :ref:`virtualenv` 中。

.. _virtualenv:

virtualenv
----------

你可能需要用到 Virtualenv 来开发 Click 应用程序。

virtualenv 解决了什么问题呢? 有可能是你想将它应用于除了 Click 脚本以外的其他项目。
你负责的项目越多，你越有可能会应用到不同版本的 Python,或者不同版本的 Python 库, 我们可能面临一个问题：很多时候库会出现兼容性问题，任何应用程序都有可能发生版本冲突。那么如果两个或两个以上的项目具有相互冲突的依赖关系，你会怎么做？

这个时候 Virtualenv 派上用场了! Virtualenv 支持多个并行的 Python 安装，每个项目一个。
它实际上并不安装单独的 Python 副本，但它提供了一种巧妙的方法来隔离不同的项目环境。
让我们看看virtualenv是如何工作的。

如果您使用的是 Mac OS X 或 Linux，则有可能是以下两个命令之一适用于您：::

    $ sudo easy_install virtualenv

或者更好的方式::

    $ sudo pip install virtualenv

以上任意一种方法都可以安装 virtualenv。甚至它已经在你的包管理器中。 
如果你是 Ubuntu 用户, 尝试下列命令进行安装::

    $ sudo apt-get install python-virtualenv

如果你在Windows上（或者上述方法都不行），你必须先安装 
``pip`` .  获取更多信息, 详见 `installing pip`_.
安装完成后，运行 上面的``pip`` 指令, 但没有
 `sudo` 前缀.

.. _installing pip: http://pip.readthedocs.org/en/latest/installing.html

一旦你安装了 virtualenv, 只需启动一个 shell 并创建你自己的环境。
我通常在以下地方创建一个项目文件夹和一个 `venv` 文件夹::

    $ mkdir myproject
    $ cd myproject
    $ virtualenv venv
    New python executable in venv/bin/python
    Installing setuptools, pip............done.

现在，只要你想在一个项目上工作，你只需要激活相应的环境。在 OS X 和 Linux 上，执行以下操作：::

    $ . venv/bin/activate

如果你是 Windows 用户, 执行下列操作::

    $ venv\scripts\activate

无论哪种方式，你现在应该使用你的 virtualenv（注意你的shell的提示已经改变，以显示新生成的环境）。

如果你想返回初始环境, 使用下列命令::

    $ deactivate

这样做以后你的 shell 的提示应该变得和以前一样.

现在，我们继续。输入以下命令获取在 virtualenv 中 Click 的运行环境：::

    $ pip install Click

只需几秒的等待即可。

视频和例子
-----------------------

有一个视频可以展示 Click 的基本 API 以及如何使用它创建简单的应用程序。
它还探讨了如何使用子命令构建命令。

*   `使用 Click 构建命令行应用程序
    <https://www.youtube.com/watch?v=kNke39OZ2k0>`_

Click 应用程序的例子可以在文档中找到，也可以在 GitHub 的 readme 文件中找到：:

*   ``inout``: `文件输入和输出
    <https://github.com/mitsuhiko/click/tree/master/examples/inout>`_
*   ``naval``: `docopt naval 端口例子
    <https://github.com/mitsuhiko/click/tree/master/examples/naval>`_
*   ``aliases``: `命令别名例子
    <https://github.com/mitsuhiko/click/tree/master/examples/aliases>`_
*   ``repo``: `Git-/Mercurial-like 命令行接口
    <https://github.com/mitsuhiko/click/tree/master/examples/repo>`_
*   ``complex``: `插件加载的复杂例子
    <https://github.com/mitsuhiko/click/tree/master/examples/complex>`_
*   ``validation``: `自定义参数验证例子
    <https://github.com/mitsuhiko/click/tree/master/examples/validation>`_
*   ``colors``: `Colorama ANSI 字体颜色支持
    <https://github.com/mitsuhiko/click/tree/master/examples/colors>`_
*   ``termui``: `终端 UI 功能演示
    <https://github.com/mitsuhiko/click/tree/master/examples/termui>`_
*   ``imagepipe``: `多命令链接演示
    <https://github.com/mitsuhiko/click/tree/master/examples/imagepipe>`_

基本概念
--------------

Click 是通过装饰器声明命令的。在内部，高级用例有一个非装饰器接口，但不鼓励高级用法。

一个函数通过装饰器成为一个 Click 命令行工具
:func:`click.command`。就是这么简单，用这个装饰器来装饰一个函数使它成为一个可调用的脚本：:

.. click:example::

    import click

    @click.command()
    def hello():
        click.echo('Hello World!')

接着装饰器将函数转化成
:class:`Command` 可以被调用的函数::

    if __name__ == '__main__':
        hello()

它看起来像:

.. click:run::

    invoke(hello, args=[], prog_name='python hello.py')

相应的帮助页面:

.. click:run::

    invoke(hello, args=['--help'], prog_name='python hello.py')

Echoing
-------

为什么这个例子使用 :func:`echo` 而不是常规的
:func:`print` 功能?  原因很简单， Click 支持不同版本的 Python，并且即使在环境配置错误的情况下也是非常易用的。即使一切都完全被破坏，Click 在基础层面上起作用。

这意味着 :func:`echo` 函数应用了一些错误修正，以防终端配置错误而不是因为
:exc:`UnicodeError` 退出程序。

另一个好处是，从 Click 2.0开始，echo 函数也对 ANSI 字体颜色有很好的支持。如果输出流是一个文件，它会自动去除 ANSI 代码空格，并且支持色彩，ANSI 色彩也可以在Windows上使用。有关更多信息，请参阅:ref:`ansi-colors`

如果你不需要这个，你也可以使用 `print()` 构造/运行.

嵌套命令
----------------

命令可以附加到其他 :class:`Group` 类型的命令。这允许任意嵌套脚本。下面例子中的脚本实现了两个管理数据库的命令：

.. click:example::

    @click.group()
    def cli():
        pass

    @click.command()
    def initdb():
        click.echo('Initialized the database')

    @click.command()
    def dropdb():
        click.echo('Dropped the database')

    cli.add_command(initdb)
    cli.add_command(dropdb)

正如你所看到的那样, :func:`group` 装饰器就像 :func:`command`
装饰器一样工作, 但创建一个 :class:`Group` 对象，可以通过 :meth:`Group.add_command` 赋予多个可以附加的子命令。

对于简单的脚本，也可以使用 :meth:`Group.command` 装饰器自动附加和创建命令。上面的脚本可以写成这样：

.. click:example::

    @click.group()
    def cli():
        pass

    @cli.command()
    def initdb():
        click.echo('Initialized the database')

    @cli.command()
    def dropdb():
        click.echo('Dropped the database')

添加参数
-----------------

要添加参数，请使用 :func:`option` 和 :func:`argument` 装饰器:

.. click:example::

    @click.command()
    @click.option('--count', default=1, help='number of greetings')
    @click.argument('name')
    def hello(count, name):
        for x in range(count):
            click.echo('Hello %s!' % name)

它看起来像：

.. click:run::

    invoke(hello, args=['--help'], prog_name='python hello.py')

.. _switching-to-setuptools:

生成 Setuptools
-----------------------

就目前你写的代码，文件末尾有一个类似这种的代码块: ``if __name__ == '__main__':``。这是一个传统独立的 Python 文件格式。使用 Click 你可以继续这样做, 但使用 setuptools 是一个更好的办法。

有两个主要的原因（还有更多其他的原因）：

第一个原因是 setuptools 自动为 Windows 生成可执行的包装器，所以你的命令行工具也可以在Windows上工作。

第二个原因是 setuptools 脚本和 Unix 上的 virtualenv 一起工作，而不需要激活 virtualenv 。这是一个非常有用的概念，它允许你将你的脚本和所有依赖绑定到一个虚拟环境中。

Click 和它搭配起来简直天衣无缝。 接下来的文档将假设你正在通过 setuptools 写应用程序。

假如你将使用setuptools，强烈建议去阅读 :ref:`setuptools-integration`
章节，然后再阅读其余的例子。
