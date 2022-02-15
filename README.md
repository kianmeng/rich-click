# rich-click

**Format [click](https://click.palletsprojects.com/) help output nicely with [Rich](https://github.com/Textualize/rich).**

- Click is a _"Python package for creating beautiful command line interfaces"_.
- Rich is a _"Python library for rich text and beautiful formatting in the terminal"_.

The intention of `rich-click` is to provide attractive help output from
click, formatted with rich, with minimal customisation required.

## Screenshots

<table>
    <tr>
        <th>Native click help</th>
        <th>With <code>rich-click</code></th>
    </tr>
    <tr>
        <td><img src="docs/images/example_with_just_click.png"></td>
        <td><img src="docs/images/example_with_rich-click.png"></td>
    </tr>
</table>

## Installation

You can install `rich-click` from the Python Package Index (PyPI) with `pip` or equivalent.

```bash
python -m pip install rich-click
```

## Usage

There are two main ways to set up `rich-click` formatting for your tool:
monkey patching or declarative.
Which you choose will depend on your use-case and your personal disposition!

Note that the intention is to maintain most / all of the normal click functionality and arguments.
If you spot something that is missing once you start using the plugin, please
create an issue about it.

### The path of least typing

Monkey patching is [probably bad](https://en.wikipedia.org/wiki/Monkey_patch#Pitfalls)
and you should only use this method if you are a Responsible Developer.
It's also good if you're lazy, as it requires very little typing.

Assuming that you're already using click, you only need to add three lines:

```python
import rich_click
click.Command.format_help = rich_click.rich_format_help
click.Group.format_help = rich_click.rich_format_help
```

_(if you're not click groups, only 2 lines!)_

This _overwrites_ the default `click` methods with those from the `rich-click` package.
As such, no other changes are needed - just continue to use `click` as you would
normally and it will use these custom methods to print your help output.

### The good and proper way

If using monkey-patching in your project makes your palms sweaty and your pulse race,
then you'll be pleased to know that you can also use `rich-click` in a nicely
declarative and verbose manner:

```python
import click
import rich_click

class RichClickGroup(click.Group):
    def format_help(self, ctx, formatter):
        rich_click.rich_format_help(self, ctx, formatter)
class RichClickCommand(click.Command):
    def format_help(self, ctx, formatter):
        rich_click.rich_format_help(self, ctx, formatter)

@click.group(cls=RichClickGroup)
@click.option('--debug/--no-debug', default=False)
def cli(debug):
    click.echo(f"Debug mode is {'on' if debug else 'off'}")

@cli.command(cls=RichClickCommand)
def sync():
    click.echo('Syncing')
```

_(example based on the [click docs](https://click.palletsprojects.com/en/8.0.x/commands/))_

Here we are making new `Group` and `Command` child classes that are based on click.
We define our custom `format_help()` functions and then tell click to to use these classes with the `cls` argument.

## Groups and sorting

`rich-click` gives functionality to list options and subcommands in groups, printed as separate panels.
It accepts a list of options / commands which means you can also choose a custom sorting order.

For example, you can produce something that looks like this:
![command groups](docs/images/command_groups.png)

- To do this with options (flags), set `rich_click.core.OPTION_GROUPS`.
- To do this with subcommands (groups), set `rich_click.core.COMMAND_GROUPS`.

### Options

To group command flags into two sections with custom names, you could do the following:

```python
rich_click.core.OPTION_GROUPS = {
    "mytool": [
        {
            "name": "Simple options",
            "options": ["--name", "--description", "--version", "--help"],
        },
        {
            "name": "Advanced options",
            "options": ["--force", "--yes", "--delete"],
        },
    ]
}
```

If you omit `name` it will use `Commands` (can be configured with `OPTIONS_PANEL_TITLE`, see _Customisation_ below).

### Commands

Here we create two groups of commands for the base command of `mytool`.
Any subcommands not listed will automatically be printed in a panel at the end labelled "Commands" as usual.

```python
rich_click.core.COMMAND_GROUPS = {
    "mytool": [
        {
            "name": "Commands for uploading",
            "commands": ["sync", "upload"],
        },
        {
            "name": "Download data",
            "commands": ["get", "fetch", "download"],
        },
    ]
}
```

If you omit `name` it will use `Commands` (can be configured with `COMMANDS_PANEL_TITLE`, see _Customisation_ below).

### Multiple commands

If you use multiple nested subcommands, you can specify their commands using the top-level dictionary keys:

```python
rich_click.core.COMMAND_GROUPS = {
    "mytool": ["commands": ["sync", "auth"]],
    "mytool sync": [
        {
            "name": "Commands for uploading",
            "commands": ["sync", "upload"],
        },
        {
            "name": "Download data",
            "commands": ["get", "fetch", "download"],
        },
    ],
    "mytool auth":[{"commands": ["login", "logout"]}],
}
```

## Customisation

You can customise most things that are related to formatting.

For example, to limit the maximum width of the help output to 100 characters:

```python
rich_click.core.MAX_WIDTH = 100
```

To print the option flags in a different colour, use:

```python
rich_click.core.STYLE_OPTION = "magenta"
```

<details><summary>Full list of config options</summary>

```python
# Default styles
STYLE_OPTION = "bold cyan"
STYLE_SWITCH = "bold green"
STYLE_METAVAR = "bold yellow"
STYLE_USAGE = "yellow"
STYLE_USAGE_COMMAND = "bold"
STYLE_DEPRECATED = "red"
STYLE_HELPTEXT_FIRST_LINE = ""
STYLE_HELPTEXT = "dim"
STYLE_METAVAR = "bold yellow"
STYLE_OPTION_HELP = ""
STYLE_OPTION_DEFAULT = "dim"
STYLE_REQUIRED_SHORT = "red"
STYLE_REQUIRED_LONG = "dim red"
STYLE_OPTIONS_PANEL_BORDER = "dim"
ALIGN_OPTIONS_PANEL = "left"
STYLE_COMMANDS_PANEL_BORDER = "dim"
ALIGN_COMMANDS_PANEL = "left"
MAX_WIDTH = None  # Set to an int to limit to that many characters

# Fixed strings
DEPRECATED_STRING = "(Deprecated) "
DEFAULT_STRING = " [default: {}]"
REQUIRED_SHORT_STRING = "*"
REQUIRED_LONG_STRING = " [required]"
RANGE_STRING = " [{}]"
OPTIONS_PANEL_TITLE = "Options"
COMMANDS_PANEL_TITLE = "Commands"

# Behaviours
SHOW_ARGUMENTS = False
USE_MARKDOWN = False
USE_RICH_MARKUP = False
COMMAND_GROUPS = {}
OPTION_GROUPS = {}
```

</details>

## Contributing

Contributions and suggestions for new features are welcome, as are bug reports!
Please create a new [issue](https://github.com/ewels/rich-click/issues)
or better still, dive right in with a pull-request.

## Credits

This package was put together hastily by Phil Ewels ([@ewels](http://github.com/ewels/)),
but the hard work was really done by Will McGugan ([@willmcgugan](https://github.com/willmcgugan))
who not only is the author of [Rich](https://github.com/Textualize/rich)
but also wrote the original code that this package is based on.
