异常处理
==================

.. currentmodule:: click

Click内部使用异常提示来表示应用程序的用户设置的各种错误条件。这是使用不当所造成的。

错误在哪里处理？
-------------------------

Click的主要错误处理发生在 :meth:`BaseCommand.main`.  在那里它处理所有的子类
:exc:`ClickException` 以及
:exc:`EOFError` 和 :exc:`KeyboardInterrupt` 标准。后者在内部翻译成一个 :exc:`Abort`.

应用的逻辑如下：

1.  如果 :exc:`EOFError` 或者 :exc:`KeyboardInterrupt` 发生,，将其重新评估为 :exc:`Abort`.
2.  如果出现一个 :exc:`ClickException` , 调用
    :meth:`ClickException.show` 方法来显示它，然后退出程序 :attr:`ClickException.exit_code`.
3.  如果发生 :exc:`Abort` 异常，则将字符串打印Aborted! 到标准错误，并使用退出代码退出程序1。
4.  如果顺利通过，退出程序退出代码0。

如果我不想要呢？
--------------------------

一般来说，你总是可以选择自己调用 :meth:`invoke` 方法。例如，如果你有一个
 :class:`Command` 你可以像这样手动调用它::

    ctx = command.make_context('command-name', ['args', 'go', 'here'])
    with ctx:
        result = command.invoke(ctx)

在这种情况下，异常将不会被完全处理，并且会像你期望的那样冒出来。

从Click 3.0开始，你也可以使用该 :meth:`Command.main` 方法，但禁用独立模式将执行两项操作：禁用异常处理并在最后通过 :meth:`sys.exit` 禁用隐式模式。

所以你可以这样做：::

    command.main(['command-name', 'args', 'go', 'here'],
                 standalone_mode=False)

存在哪些例外？
-----------------------

Click 具有两个异常基础: :exc:`ClickException` 针对所有例外情况引发的Click，这些异常是要向用户发出信号，而 :exc:`Abort`
用于指示Click的中止执行。

 :exc:`ClickException` 有一个 :meth:`~ClickException.show` 方法可以将错误消息呈现给stderr或给定的文件对象。如果你想自己使用这个异常做一些检查他们提供的API文档。

以下常见存在的子类：

*   :exc:`UsageError` 通知用户出了问题。
*   :exc:`BadParameter` 通知用户特定参数出现问题。这些通常在Click处理内部处理，如果可能的话，增加额外的信息。例如，如果这些是从回调引发Click,将自动增加它的参数名称。
*   :exc:`FileError` 这是一个错误，
    :exc:`FileType` Click遇到打开文件问题时会出现
