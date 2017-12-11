高级模式
=================

.. currentmodule:: click

除了库本身实现的通用功能之外，还有无数的模式可以通过扩展Click来实现。这节内容可以帮你更好完成信息。

.. _aliases:

命令别名
---------------

许多工具支持命令的别名。例如，你可以配置``git`` 来接受 ``git ci`` 取别名为 ``git commit``.  其他工具还支持通过自动缩短别名来自动发现别名。

Click 不支持开箱即用, 但是定制 :class:`Group` 或任何其他 :class:`MultiCommand` 来提供这个功能是非常容易的。

正如 :ref:`custom-multi-commands`解释中那样, 一个multi命令可以提供两种方法: :meth:`~MultiCommand.list_commands` 和:meth:`~MultiCommand.get_command`. 
 在这种特殊情况下，您只需要覆盖后者，因为你通常不希望枚举帮助页面上的别名，以避免混淆。

下面的例子实现了一个接受命令前缀的子类 :class:`Group`。  如果有一个命令被调用 ``push``,
它会接受 ``pus`` 这个别名（只要它是唯一的）:

.. click:example::

    class AliasedGroup(click.Group):

        def get_command(self, ctx, cmd_name):
            rv = click.Group.get_command(self, ctx, cmd_name)
            if rv is not None:
                return rv
            matches = [x for x in self.list_commands(ctx)
                       if x.startswith(cmd_name)]
            if not matches:
                return None
            elif len(matches) == 1:
                return click.Group.get_command(self, ctx, matches[0])
            ctx.fail('Too many matches: %s' % ', '.join(sorted(matches)))

然后可以像这样使用它:

.. click:example::

    @click.command(cls=AliasedGroup)
    def cli():
        pass

    @cli.command()
    def push():
        pass

    @cli.command()
    def pop():
        pass

参数修改
-----------------------

参数（选项和参数）被转发到命令回调，正如你所看到的。防止参数传递给回调的一个常见方法是参数的expose_value参数，该参数完全隐藏参数。这样做的方式是 :class:`Context`
对象具有 :attr:`~Context.params` 属性，它是所有参数的字典。
表示字典中的任何东西正在传递给回调。


这可以用来补充附加参数。通常这种模式是不推荐的，但在某些情况下，它可能是有用的。至少可以知道这个系统是这样工作的。

.. click:example::

    import urllib

    def open_url(ctx, param, value):
        if value is not None:
            ctx.params['fp'] = urllib.urlopen(value)
            return value

    @click.command()
    @click.option('--url', callback=open_url)
    def cli(url, fp=None):
        if fp is not None:
            click.echo('%s: %s' % (url, fp.code))

在这种情况下，回调 ``fp`` 函数会返回不变的URL，但也会传递第二个值给回调函数。
更值得推荐的是将信息传递给包装器:

.. click:example::

    import urllib

    class URL(object):

        def __init__(self, url, fp):
            self.url = url
            self.fp = fp

    def open_url(ctx, param, value):
        if value is not None:
            return URL(value, urllib.urlopen(value))

    @click.command()
    @click.option('--url', callback=open_url)
    def cli(url):
        if url is not None:
            click.echo('%s: %s' % (url.url, url.fp.code))


令牌标准化
-------------------

.. versionadded:: 2.0

从Click 2.0开始，可以提供用于标准化令牌的函数。令牌是选项名称，选项值或命令值。例如，这可以用来实现不区分大小写的选项。

为了使用这个特性，上下文需要传递一个执行标记规范化的函数。例如，你可以有一个将标记转换为小写的函数：

.. click:example::

    CONTEXT_SETTINGS = dict(token_normalize_func=lambda x: x.lower())

    @click.command(context_settings=CONTEXT_SETTINGS)
    @click.option('--name', default='Pete')
    def cli(name):
        click.echo('Name: %s' % name)

它如何在命令行上工作:

.. click:run::

    invoke(cli, prog_name='cli', args=['--NAME=Pete'])

调用其他命令
-----------------------

有时，从另一个命令调用一个命令可能会很有趣。但我们不鼓励用这样的click模式，不过你可以自行尝试下 :func:`Context.invoke`和 :func:`Context.forward` 方法.

它们的工作方式类似，但区别在于 :func:`Context.invoke` 只是用你提供的参数作为调用者调用另一个命令，而 :func:`Context.forward` 填充当前命令的参数。
两者都接受命令作为第一个参数，其他的一切都按照你所期望的那样向前传递。

例如:

.. click:example::

    cli = click.Group()

    @cli.command()
    @click.option('--count', default=1)
    def test(count):
        click.echo('Count: %d' % count)

    @cli.command()
    @click.option('--count', default=1)
    @click.pass_context
    def dist(ctx, count):
        ctx.forward(test)
        ctx.invoke(test, count=42)

呈现的效果:

.. click:run::

    invoke(cli, prog_name='cli', args=['dist'])


.. _callback-evaluation-order:

回调评估顺序
-------------------------

Click的作用与其他一些命令行解析器有所不同，它试图在调用任何回调函数之前，将程序员定义的参数顺序与用户定义的参数顺序进行协调。

在将复杂模式移植到optparse或其他系统进行点击时，这是一个重要的概念。optparse中的参数回调调用是解析步骤的一部分，而Click中的回调调用是在解析之后发生的。

主要区别在于，在optparse中，回调函数会在原始值被调用的情况下调用，而在Click完成转换后调用Click中的回调函数。

通常，调用的顺序是由用户向脚本提供参数的顺序驱动的; 如果有一个选项被调用，``--foo``
并且一个选项被调用 ``--bar`` 用户调用它 ``--bar--foo``,那么这个回调 ``bar`` 将会在这个选项之前
触发 ``foo``.

这里有三个特例不遵守这个规则:

渴望：
    一个选项可以设置为“渴望”。在所有非渴望参数之前评估所有渴望的参数，但是又按用户在命令行上提供的顺序来评估。

    这对于执行参数输出比如 ``--help``和 ``--version``一样重要
    两者都是渴望的参数，但是无论命令行上的第一个参数是什么，都将赢得并退出程序。

重复参数：
    如果一个选项或者参数在命令行中被分割成多个地方，比如重复 ``--exclude foo --include
    baz --exclude bar`` -- 回调会根据第一个选项的位置触发。在这种情况下，回调将触发
    ``exclude`` ，它将通过这两个选项（``foo`` 和``bar``）,然后回调 ``include`` 只会触发 ``baz``。
   
    请注意，即使一个参数不允许多个版本，Click仍然会接受第一个的位置，但是会忽略除最后一个之外的每个值。其原因是通过设置默认值的shell别名来允许可组合性。

缺少参数：
    如果在命令行中没有定义参数，回调仍然会触发。这与它在optparse中的工作方式不同，未定义的值不会触发回调。缺少参数在最后触发它们的回调，这使得它们可以默认来自之前参数的值。

大多数情况下，你不必关心这些情况，但了解某些高级案例的工作原理非常重要。

.. _forwarding-unknown-options:

转发未知的选项
--------------------------

在某些情况下，能够接受所有未知选项以进一步手动处理是有趣的。点击一般可以做到的点击4.0，但是它仍有一些局限性。对此的支持是通过一个解析器标志调用的In some situations it is interesting to be able 
``ignore_unknown_options`` ，它将指示解析器收集所有未知的选项，并将它们放到剩余参数中，
而不是触发解析错误。


可以通过下面两种方式激活:

1.  更改 :class:`Command` 属性在custom :attr:`~BaseCommand.ignore_unknown_options` 子类上启用它。
2.  更改上下文类I (:attr:`Context.ignore_unknown_options`)上相同名称的属性来启用它。这是通过 ``context_settings`` 命令字典改变。

对于大多数情况下，最简单的解决方案是第二个。一旦行为被改变，需要拿起那些剩余的选项（在这点上被认为是参数）。为此，你有两个选择：

1.  你可以使用 :func:`pass_context` 来获取传递的上下文。这只有在除了 :attr:`~Context.ignore_unknown_options`之外，你也可以设置 :attr:`~Context.allow_extra_args` ，否则命令将会中止，并有一个错误，存在剩余的参数。如果使用这个解决方案，额外的参数将被收集在
    :attr:`Context.args`中。
2.  你可以附上 :func:`argument` 将 ``nargs`` 设置为 `-1` ，这将吃掉所有剩余参数。
在这种情况下，建议将类型设置为 :data:`UNPROCESSED` 以避免对这些参数进行任何字符串处理，
否则它们会被自动强制为unicode字符串，这通常不是您想要的。

最后你会得到这样的结果：

.. click:example::

    import sys
    from subprocess import call

    @click.command(context_settings=dict(
        ignore_unknown_options=True,
    ))
    @click.option('-v', '--verbose', is_flag=True, help='Enables verbose mode')
    @click.argument('timeit_args', nargs=-1, type=click.UNPROCESSED)
    def cli(verbose, timeit_args):
        """A wrapper around Python's timeit."""
        cmdline = ['python', '-mtimeit'] + list(timeit_args)
        if verbose:
            click.echo('Invoking: %s' % ' '.join(cmdline))
        call(cmdline)

它看起来像：

.. click:run::

    invoke(cli, prog_name='cli', args=['--help'])
    println()
    invoke(cli, prog_name='cli', args=['-n', '100', 'a = 1; b = 2; a * b'])
    println()
    invoke(cli, prog_name='cli', args=['-v', 'a = 1; b = 2; a * b'])

正如你所看到的，通过Click来处理详细性标志，其他的一切都会在timeit_args变量中进行进一步处理，然后调用一个子进程。关于如何忽略未处理的标志，有几件重要的事情要知道：

*   未知的长选项通常被忽略，根本不处理。因此，举例来说，无论 ``--foo=bar``还是 ``--foo bar``  传递
他们通常最终会这样。请注意，因为解析器无法知道某个选项是否会接受参数，所以 ``bar`` 部分可能会作为
参数进行处理。
*   未知的短期选项可能会被部分处理，如有必要可重新调整。例如在上面的例子中，有一个选项 ``-v`` 可以启用详细模式。如果该命令将被忽略， ``-va`` 那么 ``-v`` 部分将由Click处理（因为它是已知的）
并且 ``-a`` 将在剩余参数中结束以用于进一步处理。
*   根据你的计划，你可能会通过禁用散布的参数(:attr:`~Context.allow_interspersed_args`) 来取得一些成功，它指示解析器不允许混合参数和选项。根据你的情况，这可能会改善你的结果。

一般来说，虽然从你自己的命令和来自另一个应用程序的命令的选项和参数的组合处理是不鼓励的，请尽可能避免这种情况。将子命令下的所有内容都转发给另一个应用程序比自己处理一些参数更好。


全局上下文访问
---------------------

.. versionadded:: 5.0

从Click 5.0开始，可以通过使用:func:`get_current_context` 函数从同一个线程中的任何地方访问当前上下
文，这主要用于访问上下文绑定对象以及存储在其上的一些标志，以定制运行时行为。例如
:func:`echo` 函数可以用来推断颜色标志的默认值。

用法示例::

    def get_current_command_name():
        return click.get_current_context().info_name

应该指出的是，这只适用于当前线程。如果你产生了额外的线程，那么这些线程将无法引用当前上下文。如果你想给另一个线程引用这个上下文的能力，你需要使用线程内的上下文作为上下文管理器::

    def spawn_thread(ctx, func):
        def wrapper():
            with ctx:
                func()
        t = threading.Thread(target=wrapper)
        t.start()
        return t

现在线程函数可以像主线程那样访问上下文。但是，如果你使用线程的话，你需要非常小心，因为绝大多数的上下文不是线程安全的！你只能从上下文中读取，而不能对其进行任何修改。
