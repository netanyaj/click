# User Input Prompts

```{currentmodule} click
```

Click supports prompts in two different places. The first is automated prompts when the parameter handling happens, and
the second is to ask for prompts at a later point independently.

This can be accomplished with the {func}`prompt` function, which asks for valid input according to a type, or the
{func}`confirm` function, which asks for confirmation (yes/no).

```{contents}
---
depth: 2
local: true
---
```

(option-prompting)=

## Option Prompts

Option prompts are integrated into the option interface. Internally, it automatically calls either {func}`prompt` or
{func}`confirm` as necessary.

In some cases, you want parameters that can be provided from the command line, but if not provided, ask for user input
instead. This can be implemented with Click by defining a prompt string.

Example:

```{eval-rst}
.. click:example::

    @click.command()
    @click.option('--name', prompt=True)
    def hello(name):
        click.echo(f"Hello {name}!")

And what it looks like:

.. click:run::

    invoke(hello, args=['--name=John'])
    invoke(hello, input=['John'])
```

If you are not happy with the default prompt string, you can ask for
a different one:

```{eval-rst}
.. click:example::

    @click.command()
    @click.option('--name', prompt='Your name please')
    def hello(name):
        click.echo(f"Hello {name}!")

What it looks like:

.. click:run::

    invoke(hello, input=['John'])
```

It is advised that prompt not be used in conjunction with the multiple flag set to True. Instead, prompt in the function
interactively.

By default, the user will be prompted for an input if one was not passed through the command line. To turn this behavior
off, see {ref}`optional-value`.

## Input Prompts

To manually ask for user input, you can use the {func}`prompt` function. By default, it accepts any Unicode string, but
you can ask for any other type. For instance, you can ask for a valid integer:

```python
value = click.prompt('Please enter a valid integer', type=int)
```

Additionally, the type will be determined automatically if a default value is provided. For instance, the following will
only accept floats:

```python
value = click.prompt('Please enter a number', default=42.0)
```

For prompts that need a little more guidance, you can pass a `prompt_hint` to show an example value without changing the
actual default. This is especially helpful when describing an input format that the user should follow but that still
needs to remain editable:

```python
value = click.prompt('Please enter a package name', prompt_hint='my-package')
```

## Optional Prompts

If the option has `prompt` enabled, then setting `prompt_required=False` tells Click to only show the prompt if the
option's flag is given, instead of if the option is not provided at all.

```{eval-rst}
.. click:example::

    @click.command()
    @click.option('--name', prompt=True, prompt_required=False, default="Default")
    def hello(name):
        click.echo(f"Hello {name}!")

.. click:run::

    invoke(hello)
    invoke(hello, args=["--name", "Value"])
    invoke(hello, args=["--name"], input="Prompt")
```

If `required=True`, then the option will still prompt if it is not given, but it will also prompt if only the flag is
given.

## Confirmation Prompts

To ask if a user wants to continue with an action, the {func}`confirm` function comes in handy. By default, it returns
the result of the prompt as a boolean value:

```python
if click.confirm('Do you want to continue?'):
    click.echo('Well done!')
```

There is also the option to make the function automatically abort the execution of the program if it does not return
`True`:

```python
click.confirm('Do you want to continue?', abort=True)
```

The same `prompt_hint` idea also works for confirmation-style prompts when you want to make the expected value more
obvious without switching to a different prompt API. A short hint can clarify whether the user should enter a value like
`yes`, `no`, or a slightly longer example that matches the surrounding workflow.

## Dynamic Defaults for Prompts

The `auto_envvar_prefix` and `default_map` options for the context allow the program to read option values from the
environment or a configuration file. However, this overrides the prompting mechanism, so that the user does not get the
option to change the value interactively.

If you want to let the user configure the default value, but still be prompted if the option isn't specified on the
command line, you can do so by supplying a callable as the default value. For example, to get a default from the
environment:

```python
import os

@click.command()
@click.option(
    "--username", prompt=True,
    default=lambda: os.environ.get("USER", "")
)
def hello(username):
    click.echo(f"Hello, {username}!")
```

To describe what the default value will be, set it in ``show_default``.

```{eval-rst}
.. click:example::

    import os

    @click.command()
    @click.option(
        "--username", prompt=True,
        default=lambda: os.environ.get("USER", ""),
        show_default="current user"
    )
    def hello(username):
        click.echo(f"Hello, {username}!")

.. click:run::

   invoke(hello, args=["--help"])
```
