记录脚本
===================

.. currentmodule:: click

Click 可以很容易地记录你的命令行工具。首先，它会自动为您生成帮助页面。虽然这些目前还不能根据其布局进行自定义，但所有文本都可以更改。

帮助文本
----------

命令和选项接受帮助参数。在命令的情况下，功能的文档字符串是自动使用

简单举例：

.. click:example::

    @click.command()
    @click.option('--count', default=1, help='number of greetings')
    @click.argument('name')
    def hello(count, name):
        """This script prints hello NAME COUNT times."""
        for x in range(count):
            click.echo('Hello %s!' % name)

它看起来像：

.. click:run::

    invoke(hello, args=['--help'])

参数不能用这种方式记录。这是遵循Unix工具的一般惯例，只使用最必要的参数来使用参数，并通过名称来引用它们。

防止重新装包
---------------------

Click 的默认行为是根据终端的宽度重新包装文本。在某些情况下，这可能会成为一个问题。主要的问题是在显示代码示例时，换行符很重要。

重新包装可以通过添加一个只有 ``\b`` 转义标记的行来禁用。此行将从帮助文本中删除，重新包装将被禁用。

例：

.. click:example::

    @click.command()
    def cli():
        """First paragraph.

        This is a very long second paragraph and as you
        can see wrapped very early in the source text
        but will be rewrapped to the terminal width in
        the final output.

        \b
        This is
        a paragraph
        without rewrapping.

        And this is a paragraph
        that will be rewrapped again.
        """

它看起来像：

.. click:run::

    invoke(cli, args=['--help'])

元变量
--------------

选项和参数接受 ``metavar`` 参数，可以更改在帮助页面中的元变量。默认版本是带有下划线的大写参数名称，但如果需要，可以按不同的方式进行注释。这可以在各个层面进行定制：

.. click:example::

    @click.command(options_metavar='<options>')
    @click.option('--count', default=1, help='number of greetings',
                  metavar='<int>')
    @click.argument('name', metavar='<name>')
    def hello(count, name):
        """This script prints hello <name> <int> times."""
        for x in range(count):
            click.echo('Hello %s!' % name)

举例：

.. click:run::

    invoke(hello, args=['--help'])


命令的短帮助
------------------

对于命令，会生成一个简短的帮助代码片断。默认情况下，它是该命令的帮助消息的第一个句子，除非它太长。这也可以被覆盖：

.. click:example::

    @click.group()
    def cli():
        """A simple command line tool."""

    @cli.command('init', short_help='init the repo')
    def init():
        """Initializes the repository."""

    @cli.command('delete', short_help='delete the repo')
    def delete():
        """Deletes the repository."""

它看起来像：

.. click:run::

    invoke(cli, prog_name='repo.py')

帮助参数订制
----------------------------

2.0版本新功能

帮助参数在 Click 中以一种非常特殊的方式实现。与常规参数不同的是，它自动通过点击添加任何命令，并执行自动冲突解决。默认情况下它被调用 ``--help`` ，但是这个可以改变。如果一个命令本身实现了一个名字相同的参数，那么默认的帮助参数将停止接受它。有一个上下文设置可以用来覆盖所调用的帮助参数的名称 `help_option_names` 。

这个例子改变了默认的参数，用 ``-h--`` 和 ``--help`` 代替 ``--help`` ：

.. click:example::

    CONTEXT_SETTINGS = dict(help_option_names=['-h', '--help'])

    @click.command(context_settings=CONTEXT_SETTINGS)
    def cli():
        pass

它看起来像：

.. click:run::

    invoke(cli, ['-h'])
