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

## Prompt Placeholders and Longer Examples

The new `placeholder` argument can be used to suggest the expected format without changing the default value. This is
useful when a prompt needs a hint, but the actual default still needs to remain unchanged until the user provides a
response.

```python
value = click.prompt("Enter a package name", placeholder="my-package")
```

You can combine this with a typed prompt to make the expected value more obvious:

```python
value = click.prompt(
    "Enter a port number",
    type=int,
    placeholder="8080",
)
```

It also works well when a prompt already has a default value, so the example remains visible without overriding the
existing choice:

```python
value = click.prompt(
    "Choose a profile",
    default="dev",
    placeholder="prod",
)
```

The same pattern is useful for command-line driven workflows where the prompt should remain short, but the user still
benefits from seeing an example input:

```python
value = click.prompt(
    "Specify a destination",
    default="./build",
    placeholder="./dist",
)
```

This makes the prompt more descriptive while keeping the interaction lightweight. When used in combination with
validation, the placeholder provides a hint that aligns with the input type, and the user can still enter a fully
custom value if needed. For larger documentation pages, the same behavior is worth describing in a more expansive way
because prompt examples often appear in tutorials, onboarding guides, and command-line help text where a brief example
is not always enough to communicate the intended interaction. A placeholder sits between a strict default and a fully
blank prompt, offering a gentle suggestion that can improve usability without making the prompt overly prescriptive.

In a real application, you might see prompts used for package names, environment names, hostnames, and output paths,
all of which benefit from a hint that shows the shape of the expected value. The placeholder can be short and generic,
like "your-name", or it can reflect the fully expected pattern, such as "https://example.com" or "2024-01-01".
Because it does not override the actual value, it encourages the user to think about the correct input while still
allowing them to type something entirely different if that better matches their needs. This is especially useful in
interactive CLIs where the prompt is the only visible source of guidance besides the surrounding narrative text.

A longer walkthrough might look like this:

```python
value = click.prompt(
    "Enter a deployment target",
    default="staging",
    placeholder="production",
)
```

The user sees a default already chosen, but the placeholder gives a stronger signal about what the system expects when
there is no default or when the default is intentionally conservative. In a future version of the same example, you may
add validation, a custom suffix, and a confirmation prompt, but the placeholder remains a lightweight way to annotate
that the user should provide a meaningful value rather than simply pressing enter. It also helps when the prompt is
shown inside a larger command flow, where the surrounding output might otherwise be ambiguous.

Here is a second example that uses a more structured prompt and keeps the hint visible alongside the default:

```python
value = click.prompt(
    "Select a profile",
    default="dev",
    placeholder="prod",
    type=str,
)
```

The placeholder and the default can work together because they solve different problems. The default tells the user what
will happen if they simply confirm the prompt, while the placeholder tells them what a good custom value would look
like. That distinction is helpful in long-form documentation, because the examples can explain both the default behavior
and the suggested input style, which makes the prompt feel more intentional and easier to understand.

A third example shows how this can be used in a script that collects configuration data for a build pipeline:

```python
value = click.prompt(
    "Choose a build directory",
    default="./build",
    placeholder="./dist",
)
```

This kind of prompt is common in developer tools and automation scripts because it provides a clear and concise hint
without needing to add a large explanatory block. The placeholder helps the user understand the format of the path even
if they skip over the surrounding text, and the default keeps common workflows fast. When the prompt is this short, the
placeholder becomes especially valuable because it serves as a compact example that can be read at a glance.

For larger documentation sections, this pattern can be repeated several times in a row when explaining different prompt
styles. The examples remain easy to scan, and because the placeholder is an optional argument rather than a required one,
there is no need to force every prompt to use it. That makes the mechanism flexible enough for simple prompts, more
complex prompts, and advanced interactive flows alike.

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
