命令和组
===================

.. currentmodule:: click

Click最重要的特点是任意嵌套命令行工具的概念。这是通过 :class:`Command`
和 :class:`Group` (实际上是 :class:`MultiCommand`) 实现的。

回调调用
-------------------

对于一个普通的命令，只要命令运行就执行回调。如果脚本是唯一的命令，它总是会被触发（除非参数回调阻止了它）。
如果有人传递 ``--help`` 给脚本，就会发生这种情况。

对于组和多个命令，情况看起来不同。在这种情况下，只要子命令触发，就会触发回调（除非这种行为被改变）。
这实际上意味着当内部命令运行时外部命令运行：

.. click:example::

    @click.group()
    @click.option('--debug/--no-debug', default=False)
    def cli(debug):
        click.echo('Debug mode is %s' % ('on' if debug else 'off'))

    @cli.command()
    def sync():
        click.echo('Synching')

效果是这样的：

.. click:run::

    invoke(cli, prog_name='tool.py')
    println()
    invoke(cli, prog_name='tool.py', args=['--debug', 'sync'])

传递参数
------------------

Click 点击严格分隔命令和子命令之间的参数。这意味着该选项和参数特定命令必须指定 *之后* 的命令名称本身，但 *之前* 任何其他命令的名字。

这个行为已经被预定义的 ``--help`` 选项观察到了。
假设我们有一个叫做的程序 ``tool.py``, 其中包含一个叫做子命令 ``sub``

- ``tool.py --help`` 将返回整个程序的帮助（列出子命令）。

- ``tool.py sub --help`` 将返回 ``sub`` 子命令的帮助。

- 但 ``tool.py --help sub`` 将把 ``--help`` 视作为主要方案的论据。 这时 Click 调用回调 ``--help``, 打印帮助和中止程序，再点击可以处理子命令。

嵌套处理和上下文
----------------------------

从前面的示例中可以看到，基本命令组接受一个调试参数，该参数传递给其回调函数，但不传递给sync命令本身。sync命令只接受自己的参数。

这允许工具完全相互独立，但是一个命令如何与嵌套命令进行对话呢？答案是 :class:`Context`

每次调用一个命令时，都会创建一个新的上下文并与父上下文链接。通常情况下，你看不到这些情况，
但他们在那里。上下文自动传递给参数回调和值。 命令还可以通过使用 :func:`pass_context` 装饰器标记自己来请求传递上下文。在这种情况下，上下文作为第一个参数传递。

上下文还可以携带一个程序指定的对象，可以用于程序的目的。这意味着你可以像这样构建一个脚本：

.. click:example::

    @click.group()
    @click.option('--debug/--no-debug', default=False)
    @click.pass_context
    def cli(ctx, debug):
        ctx.obj['DEBUG'] = debug

    @cli.command()
    @click.pass_context
    def sync(ctx):
        click.echo('Debug is %s' % (ctx.obj['DEBUG'] and 'on' or 'off'))

    if __name__ == '__main__':
        cli(obj={})

如果提供了对象，那么每个上下文都将把对象传递给它的子对象，但是在任何级别都可以覆盖上下文的对象。
使用 ``context.parent`` 可以要达到父级。

除此之外，并不是传递一个对象，而是阻止应用程序修改全局状态。例如，你可以只是翻转一个全局 ``DEBUG`` 变量，并完成它。

装饰命令
-------------------

正如你在前面的例子中看到的那样，装饰器可以改变命令的调用方式。幕后实际发生的事情是回调总是通过 :meth:`Context.invoke` 自动调用命令的方法来调用（通过传递上下文）。

当你想编写自定义装饰器时，这是非常有用的。例如，一个常见的模式是配置一个表示状态的对象，然后将其存储在上下文中，然后使用自定义装饰器来查找该类型的最新对象并将其作为第一个参数传递。

例如, the :func:`pass_obj` 装饰器可以像这样实现：

.. click:example::

    from functools import update_wrapper

    def pass_obj(f):
        @click.pass_context
        def new_func(ctx, *args, **kwargs):
            return ctx.invoke(f, ctx.obj, *args, **kwargs)
        return update_wrapper(new_func, f) 

:meth:`Context.invoke` 命令将以正确的方式自动调用该函数，因此该函数将被 ``f(ctx, obj)`` 或 ``f(obj)`` 调用，取决于它本身是否被 :func:`pass_context`

这是一个非常强大的概念，可以用来构建非常复杂的嵌套应用程序; 有关更多信息，:ref:`complex-guide`

无命令下的组调用
--------------------------------

默认情况下，除非传递子命令，否则不会调用组命令或多命令。实际上，``--help``默认情况下不会自动提供一个命令。
这种行为可以通过传递 ``invoke_without_command=True`` 给一个组来改变 。在这种情况下，回调总是被调用，而不是显示帮助页面。
上下文对象还包含有关调用是否转到子命令的信息。

举例：

.. click:example::

    @click.group(invoke_without_command=True)
    @click.pass_context
    def cli(ctx):
        if ctx.invoked_subcommand is None:
            click.echo('I was invoked without subcommand')
        else:
             click.echo('I am about to invoke %s' % ctx.invoked_subcommand)

    @cli.command()
    def sync():
         click.echo('The subcommand')

它在实践中如何运作：

.. click:run::

    invoke(cli, prog_name='tool', args=[])
    invoke(cli, prog_name='tool', args=['sync'])

.. _custom-multi-commands:

自定义多命令
---------------------

除了使用 :func:`click.group`, 您还可以建立自己的自定义多命令。当你想要支持从插件中延迟加载的命令时，这很有用。
一个自定义的多命令只需要实现一个列表和加载方法：

.. click:example::

    import click
    import os

    plugin_folder = os.path.join(os.path.dirname(__file__), 'commands')

    class MyCLI(click.MultiCommand):

        def list_commands(self, ctx):
            rv = []
            for filename in os.listdir(plugin_folder):
                if filename.endswith('.py'):
                    rv.append(filename[:-3])
            rv.sort()
            return rv

        def get_command(self, ctx, name):
            ns = {}
            fn = os.path.join(plugin_folder, name + '.py')
            with open(fn) as f:
                code = compile(f.read(), fn, 'exec')
                eval(code, ns, ns)
            return ns['cli']

    cli = MyCLI(help='This tool\'s subcommands are loaded from a '
                'plugin folder dynamically.')

    if __name__ == '__main__':
        cli()

这些自定义类也可以用于装饰器：

.. click:example::

    @click.command(cls=MyCLI)
    def cli():
        pass

合并多个指令
----------------------

除了实现自定义多重命令之外，将多个多重命令合并为一个脚本也是有趣的。虽然这个建议通常不如一个嵌套在另一个下面，
但是在某些情况下，合并方法对于更好的外壳体验是有用的。

这种合并系统的默认实现是　:class:`CommandCollection` 类。它接受其他多个命令的列表，并使命令在同一级别上可用。

用法示例：

.. click:example::

    import click

    @click.group()
    def cli1():
        pass

    @cli1.command()
    def cmd1():
        """Command on cli1"""

    @click.group()
    def cli2():
        pass

    @cli2.command()
    def cmd2():
        """Command on cli2"""

    cli = click.CommandCollection(sources=[cli1, cli2])

    if __name__ == '__main__':
        cli()

它看起来像：

.. click:run::

    invoke(cli, prog_name='cli', args=['--help'])

如果一个命令存在于多个源中，则第一个源有效。


.. _multi-command-chaining:

多命令链接
----------------------

3.0 新版功能.

有时被允许一次调用多个子命令是有用的。例如，如果您已经安装了setuptools的包，你可能熟悉的前 一个 ``setup.py sdist bdist_wheel upload`` 命令链式调用来实现，调用 ``dist`` 在 ``bdist_wheel`` 之前，在　``upload``之前。从Click 3.0开始，这很容易实现。

你所要做的就是传递　``chain=True`` 给你的多态:

.. click:example::

    @click.group(chain=True) 
    def cli():
        pass


    @cli.command('sdist')
    def sdist():
        click.echo('sdist called')


    @cli.command('bdist_wheel')
    def bdist_wheel():
        click.echo('bdist_wheel called')

现在你可以像这样调用它：

.. click:run::

    invoke(cli, prog_name='setup.py', args=['sdist', 'bdist_wheel'])

当使用多命令链接时，只能在一个参数上使用一个　``nargs=-1``　命令（在最后）。在链式多命令
下面嵌套多个命令也是不可能的。除此之外，他们的工作方式没有限制。他们可以正常接受选项和论点。

另外需要注意的是：这个 :attr:`Context.invoked_subcommand` 属性对于多命令来说是没有用处的，它将给
``'*'``
赋值，如果多个命令调用。这是必要的，因为子命令的处理是一个接一个发生的，所以当回调触发时，将要处理的子命令不可用。

.. note::

    It is currently not possible for chain commands to be nested.  This
    will be fixed in future versions of Click.


多命令管道
-----------------------

3.0新版功能。

多命令链接的一个非常常见的用例是让一个命令处理前一个命令的结果。有多种方式可以促进这一点。最显而易见的方法是在上下文对象上存储一个值，并将其从一个函数处理到另一个函数。这可以通过装饰一个 :func:`pass_context` 函数来完成，在这个函数之后提供上下文对象，子命令可以在那里存储它的数据。

另一种方法是通过返回处理函数来设置管线。可以这样想：当一个子命令被调用时，它处理所有的参数，并提出一个如何处理的计划。在那时，它返回一个处理函数并返回。

返回的函数在哪里？　链式多命令可以通过 :meth:`MultiCommand.resultcallback` 遍历所有这些函数来注册回调，然后调用它们。

为了使这个更具体一些考虑这个例子：

.. click:example::

    @click.group(chain=True, invoke_without_command=True)
    @click.option('-i', '--input', type=click.File('r'))
    def cli(input):
        pass

    @cli.resultcallback()
    def process_pipeline(processors, input):
        iterator = (x.rstrip('\r\n') for x in input)
        for processor in processors:
            iterator = processor(iterator)
        for item in iterator:
            click.echo(item)

    @cli.command('uppercase')
    def make_uppercase():
        def processor(iterator):
            for line in iterator:
                yield line.upper()
        return processor

    @cli.command('lowercase')
    def make_lowercase():
        def processor(iterator):
            for line in iterator:
                yield line.lower()
        return processor

    @cli.command('strip')
    def make_strip():
        def processor(iterator):
            for line in iterator:
                yield line.strip()
        return processor

这是一个很大的问题，所以让我们一步一步来解决。

1.  首先要做一个可链接的 :func:`group` 。 除此之外，即使没有定义子命令，我们也指示单击来调用。如果不这样做，那么调用一个空管道将产生帮助页面，而不是运行结果回调。
2.  我们接下来要做的就是在我们的小组上注册结果做回调。这个回调将被调用一个参数，该参数是所有子命令的所有返回值的列表，并且是与我们组本身相同的关键字参数。这意味着我们可以轻松访问输入文件，而无需使用上下文对象。
3.  在这个结果回调中，我们创建了输入文件中所有行的迭代器，然后通过所有从子命令返回的所有回调传入此迭代器，最后我们将所有行打印到stdout。


之后，我们可以根据需要注册许多子命令，每个子命令可以返回一个处理器函数来修改线路流。

需要注意的一个重要的事情是，在每次回调运行之后，Click都会关闭上下文。　这意味着例如文件类型不能在 `processor` 函数中被访问，因为文件将在那里被关闭。这个限制不太可能改变，因为这会使资源处理变得更加复杂。为此，建议不要使用文件类型，并通过　:func:`open_file`　函数打开文件。

对于一个更复杂的例子，在处理管道时也有所改进，请看Click资源库中的 `imagepipe multi命令链接演示
<https://github.com/mitsuhiko/click/tree/master/examples/imagepipe>` 它实现了一个基于管道的图像编辑工具，具有良好的管道内部结构。


覆盖默认值
-------------------

默认情况下，参数的默认值是从 ``default`` 定义时提供的标志中提取的，但这不是唯一可以从中加载的默认值。另一个地方是　:attr:`Context.default_map` 上下文（字典）。这允许从配置文件加载默认值来覆盖常规的默认值。

如果您插入来自其他软件包的某些命令，但是您对默认值不满意，这很有用。

默认映射可以针对每个子命令任意嵌套，并在脚本被调用时提供。另外，它也可以在任何时候被命令覆盖。例如，顶级命令可以从配置文件加载默认值。

用法示例：

.. click:example::

    import click

    @click.group()
    def cli():
        pass

    @cli.command()
    @click.option('--port', default=8000)
    def runserver(port):
        click.echo('Serving on http://127.0.0.1:%d/' % port)

    if __name__ == '__main__':
        cli(default_map={
            'runserver': {
                'port': 5000
            }
        })

具体运行如下：

.. click:run::

    invoke(cli, prog_name='cli', args=['runserver'], default_map={
        'runserver': {
            'port': 5000
        }
    })

上下文默认值
----------------

2.0版本新功能。

从Click 2.0开始，不仅可以在调用脚本时覆盖上下文的默认值，还可以在声明命令的装饰器中覆盖默认值。举个例子，前面定义了一个自定义的例子，　``default_map``　现在也可以在装饰器中完成。

这个例子和前面的例子一样：

.. click:example::

    import click

    CONTEXT_SETTINGS = dict(
        default_map={'runserver': {'port': 5000}}
    )

    @click.group(context_settings=CONTEXT_SETTINGS)
    def cli():
        pass

    @cli.command()
    @click.option('--port', default=8000)
    def runserver(port):
        click.echo('Serving on http://127.0.0.1:%d/' % port)

    if __name__ == '__main__':
        cli()

再次举例说明：

.. click:run::

    invoke(cli, prog_name='cli', args=['runserver'])


命令返回值
---------------------

3.0版本新功能

Click 3.0中的新增功能之一就是完全支持命令回调的返回值。这实现了以前难以实现的全部功能。

实质上，任何命令回调现在都可以返回一个值。这个返回值冒泡给某些接收者。其中一个用例已经在 
:ref:`multi-command-chaining` 的例子中显示出来，在这个例子中，链接的多命令可以有处理所有返回值的回调。

在Click中使用命令返回值时，这是你需要知道的：

-   命令回调的返回值通常从
    :meth:`BaseCommand.invoke` 方法返回 。 这个规则的例外与　:class:`Group`\s:

    *   在一个组中，返回值通常是被调用的子命令的返回值。这个规则的唯一例外是返回值是组的回调的返回值，如果它被调用没有参数和　`invoke_without_command` 被启用。
    *   如果一个组被设置为链接，则返回值是所有子命令结果的列表。
    *   组的返回值可以通过　:attr:`MultiCommand.result_callback` 来处理。 这是通过链表模式中的所有返回值的列表来调用的，或者是在非链接命令的情况下的单个返回值。

-   返回值通过 :meth:`Context.invoke`　方法和 :meth:`Context.forward` 方法以冒泡的形式返回。这在您内部需要调用另一个命令的情况下非常有用。

-   点击对返回值没有任何硬性要求，并且不使用它们本身。这允许返回值用于自定义装饰器或工作流（如在多命令链接示例中）。

-   当Click脚本作为命令行应用程序(通过　:meth:`BaseCommand.main`　) 被调用时， 除非　`standalone_mode` 被禁用，否则返回值被忽略 。
