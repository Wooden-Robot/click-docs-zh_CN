.. _complex-guide:

复杂的应用程序
====================

.. currentmodule:: click

Click 旨在帮助创建复杂和简单的 CLI 工具。然而，它设计初衷是将任意系统嵌套在一起。例如，如果你曾经使用 Django，你会意识到它提供了一个命令行实用程序，但是 Celery 也有此功能。在 Django 中使用 Celery 时，这两个工具需要互相交互并进行交叉配置。

在两个独立的 Click 命令行实用程序的理论上，它们可以通过嵌套在另一个内部来解决这个问题。例如，Web 框架也可以加载消息队列框架的命令。

基本概念
--------------

要理解这是如何工作的，你需要理解两个概念：上下文和调用约定。

上下文(Contexts)
````````

无论何时执行 Click 命令，:class:`Context` 都会创建一个对象，用于保存此特定调用的状态。它会记住解析的参数，创建它的命令，需要在函数结束时清理哪些资源，等等。它也可以选择性地保存应用程序定义的对象。

上下文对象构建一个链表，直到它们碰到最上面的一个。每个上下文链接到父上下文。这允许命令在另一个命令之下工作，并在那里存储它自己的信息，而不必担心改变父命令的状态。

因为父数据是可用的，但是，如果需要，可以导航到它。

大多数情况下，你没有看到上下文对象，但是当编写更复杂的应用程序时，它会派上用场。下面我们来看下一个概念。

调用约定(Calling Convention)
``````````````````
当执行 Click 命令回调时，它将所有非隐藏参数作为关键字参数传递。Contexts 显然不存在。但是，回调可以选择通过标记自己来传递给上下文对象 :func:`pass_context`

那么如果你不知道它是否应该接收上下文(contexts)你怎么调用一个命令回调？
答案是，上下文本身提供了一个辅助函数(:meth:`Context.invoke`)它可以为你做到这一点。
它接受回调作为第一个参数，然后正确调用该函数。

建立一个 Git 克隆
--------------------

在这个例子中，我们想要构建一个类似于版本控制系统的命令行工具。像 Git 这样的系统通常会提供一个重载的命令，它已经接受了一些参数和配置，然后有额外的子命令来做其他事情。

根命令
````````````````

在顶层，我们需要一个可以容纳所有命令的组。在这种情况下，我们使用 :func:`click.group` 允许我们在其下面注册其他 Click 命令。

对于这个命令，我们也想接受一些配置我们工具状态的参数：

.. click:example::

    import os
    import click


    class Repo(object):
        def __init__(self, home=None, debug=False):
            self.home = os.path.abspath(home or '.')
            self.debug = debug


    @click.group()
    @click.option('--repo-home', envvar='REPO_HOME', default='.repo')
    @click.option('--debug/--no-debug', default=False,
                  envvar='REPO_DEBUG')
    @click.pass_context
    def cli(ctx, repo_home, debug):
        ctx.obj = Repo(repo_home, debug)


让我们来看看其中含义，我们创建一个可以有子命令的组命令。当它被调用时，它将创建一个 Repo 类的实例。它将保持我们的命令行工具的状态。在这种情况下，它只是记住一些参数，但在这一点上它也可以加载配置文件等等。

这个状态对象接着被上下文（contexts）作为 :attr:`~Context.obj` 所记住。
这是一个特殊的属性，命令应该记住他们需要传递给他们的孩子。

同时，我们需要标记我们的功能
:func:`pass_context`, 否则，上下文对象将会把我们完全隐藏掉。

第一个子命令
```````````````````````

让我们添加我们的第一个子命令，克隆命令：

.. click:example::

    @cli.command()
    @click.argument('src')
    @click.argument('dest', required=False)
    def clone(src, dest):
        pass

所以现在我们有一个克隆命令，但是我们如何获得 repo?  一种方法是使用 :func:`pass_context` 函数，这个函数也会让我们的回调函数也获得我们之前 repo 掉的上下文(contexts)。
另一种方法，我可以直接使用 :func:`pass_obj` 装饰器，它将只传递存储的对象（在我们的例子中是repo）：


.. click:example::

    @cli.command()
    @click.argument('src')
    @click.argument('dest', required=False)
    @click.pass_obj
    def clone(repo, src, dest):
        pass

交错命令
````````````````````

虽然与我们想要构建的特定程序无关，但交错命令对交错系统也有很好的支持。例如，我们的版本控制系统有一个超级酷的插件需要大量的配置，并希望将自己的配置存储为
:attr:`~Context.obj`。如果我们再附上另一个命令，我们会突然得到插件配置，而不是我们的repo对象。

解决这个问题的一个显而易见的方法就是在插件中存储一个对 repo 的引用，但是这个命令需要知道它是附在这个插件下面的。

有一个更好的系统，可以利用上下文的链接性建立起来。我们知道，插件上下文链接到创建我们的 repo 的上下文。因此，我们可以开始搜索由上下文存储的对象是 repo 的最后一级。

内置支持由 :func:`make_pass_decorator`，它将为我们创建装饰器，以查找对象（内部调用
 :meth:`Context.find_object`）。在我们的例子中，我们知道我们要找到最接近的 Repo 对象，
 所以让我们为此做一个装饰器：

.. click:example::

    pass_repo = click.make_pass_decorator(Repo)

如果我们现在使用  pass_repo 而不是 pass_obj，我们将总是得到一个 repo，而不是别的：

.. click:example::

    @cli.command()
    @click.argument('src')
    @click.argument('dest', required=False)
    @pass_repo
    def clone(repo, src, dest):
        pass

保障对象的创建
````````````````````````

上面的例子只有在有一个外部命令创建一个 Repo 对象并将其存储在上下文中时才起作用。对于一些更高级的用例，这可能会成为一个问题。:func:`make_pass_decorator` 默认的行为是调用 :meth:`Context.find_object`
（帮助定位对象）。如果找不到对象，则会引发错误。另一种行为是使用 :meth:`Context.ensure_object`
（帮助定位对象）, 如果找不到它，将会创建一个并将其存储在最内层的上下文中。这种行为也可以利用
:func:`make_pass_decorator` 传递 ``ensure=True`` 来启用:

.. click:example::

    pass_repo = click.make_pass_decorator(Repo, ensure=True)

在这种情况下，最里面的上下文获取一个对象，如果它缺少。这可能会取代之前放置的对象。即使外部命令没有运行，命令仍然可执行。为此，对象类型需要有一个不接受参数的构造函数。

因此它单独运行：

.. click:example::

    @click.command()
    @pass_repo
    def cp(repo):
        click.echo(repo)

你将看到：

.. click:run::

    invoke(cp, [])
