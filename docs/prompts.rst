用户输入提示
==================

.. currentmodule:: click

Click 支持两个不同地方的提示。第一种是参数处理发生时自动提示，第二种是后来单独要求提示。

这可以通过 :func:`prompt` 功能实现, 该功能要求根据类型进行有效输入，或者使用 :func:`confirm` 功能，并在使用时要求确认的功能（是/否）。

选项提示
--------------

选项提示集成到选项界面中。有关更多信息，请参阅 :ref:`option-prompting` 。在内部，它会自动调用 :func:`prompt` 或 :func:`confirm` 根据需要。

输入提示
-------------

要手动要求用户输入，您可以使用该 :func:`prompt` 功能。
默认情况下，它接受任何Unicode字符串，但您可以要求任何其他类型。例如，你可以要求一个有效的整数::

    value = click.prompt('Please enter a valid integer', type=int)

此外，如果提供默认值，则会自动确定类型。例如，以下将只接受浮点数::

    value = click.prompt('Please enter a number', default=42.0)

确认提示
--------------------

要问一个用户是否想要继续一个动作，这个 :func:`confirm`
函数就派上用场了。默认情况下，它以布尔值的形式返回提示的结果::

    if click.confirm('Do you want to continue?'): 
        click.echo('Well done!')

如果程序没有返回 ``True`` ，也可以自动中止程序的执行::

    click.confirm('Do you want to continue?', abort=True)
