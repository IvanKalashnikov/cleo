# Cleo

[![Tests](https://github.com/python-poetry/cleo/actions/workflows/tests.yml/badge.svg)](https://github.com/python-poetry/cleo/actions/workflows/tests.yml)
[![PyPI version](https://img.shields.io/pypi/v/cleo)](https://pypi.org/project/cleo/)

Create beautiful and testable command-line interfaces.

## Resources

- [Documentation](http://cleo.readthedocs.io)
- [Issue Tracker](https://github.com/python-poetry/cleo/issues)

## Usage

To make a command that greets you from the command line, create
`greet_command.py` and add the following to it:

```python
from cleo.commands.command import Command


class GreetCommand(Command):

    # specify command name *require
    name = "great"

    """
    Greets someone

    greet
        {name? : Who do you want to greet?}
        {--y|yell : If set, the task will yell in uppercase letters}
    """

    def handle(self):
        name = self.argument("name")

        if name:
            text = f"Hello {name}"
        else:
            text = "Hello"

        if self.option("yell"):
            text = text.upper()

        self.line(text)
```

You also need to create the file `application.py` to run at the command line which
creates an `Application` and adds commands to it:

```python
#!/usr/bin/env python

from greet_command import GreetCommand

from cleo.application import Application


application = Application()
application.add(GreetCommand())

if __name__ == "__main__":
    application.run()
```

Test the new command by running the following

```bash
$ python application.py greet John
```

This will print the following to the command line:

```text
Hello John
```

You can also use the `--yell` option to make everything uppercase:

```bash
$ python application.py greet John --yell
```

This prints:

```text
HELLO JOHN
```

As you may have already seen, Cleo uses the command docstring to
determine the command definition. The docstring must be in the following
form :

```python
"""
Command description

Command signature
"""
```

The signature being in the following form:

```python
"""
command:name {argument : Argument description} {--option : Option description}
"""
```

The signature can span multiple lines.

```python
"""
command:name
    {argument : Argument description}
    {--option : Option description}
"""
```

### Coloring the Output

Whenever you output text, you can surround the text with tags to color
its output. For example:

```python
# green text
self.line("<info>foo</info>")

# yellow text
self.line("<comment>foo</comment>")

# black text on a cyan background
self.line("<question>foo</question>")

# white text on a red background
self.line("<error>foo</error>")
```

The closing tag can be replaced by `</>`, which revokes all formatting
options established by the last opened tag.

It is possible to define your own styles using the `add_style()` method:

```python
self.add_style("fire", fg="red", bg="yellow", options=["bold", "blink"])
self.line("<fire>foo</fire>")
```

Available foreground and background colors are: `black`, `red`, `green`,
`yellow`, `blue`, `magenta`, `cyan` and `white`.

And available options are: `bold`, `underscore`, `blink`, `reverse` and
`conceal`.

You can also set these colors and options inside the tag name:

```python
# green text
self.line("<fg=green>foo</>")

# black text on a cyan background
self.line("<fg=black;bg=cyan>foo</>")

# bold text on a yellow background
self.line("<bg=yellow;options=bold>foo</>")
```

### Verbosity Levels

Cleo has four verbosity levels. These are defined in the `Output` class:

| Mode                     | Meaning                            | Console option    |
| ------------------------ | ---------------------------------- | ----------------- |
| `Verbosity.QUIET`        | Do not output any messages         | `-q` or `--quiet` |
| `Verbosity.NORMAL`       | The default verbosity level        | (none)            |
| `Verbosity.VERBOSE`      | Increased verbosity of messages    | `-v`              |
| `Verbosity.VERY_VERBOSE` | Informative non essential messages | `-vv`             |
| `Verbosity.DEBUG`        | Debug messages                     | `-vvv`            |

It is possible to print a message in a command for only a specific
verbosity level. For example:

```python
if Verbosity.VERBOSE <= self.io.verbosity:
    self.line(...)
```

There are also more semantic methods you can use to test for each of the
verbosity levels:

```python
if self.output.is_quiet():
    # ...

if self.output.is_verbose():
    # ...
```

You can also pass the verbosity flag directly to `line()`.

```python
self.line("", verbosity=Verbosity.VERBOSE)
```

When the quiet level is used, all output is suppressed.

### Using Arguments

The most interesting part of the commands are the arguments and options
that you can make available. Arguments are the strings - separated by
spaces - that come after the command name itself. They are ordered, and
can be optional or required. For example, add an optional `last_name`
argument to the command and make the `name` argument required:

```python
class GreetCommand(Command):
    """
    Greets someone

    greet
        {name : Who do you want to greet?}
        {last_name? : Your last name?}
        {--y|yell : If set, the task will yell in uppercase letters}
    """
```

You now have access to a `last_name` argument in your command:

```python
last_name = self.argument("last_name")
if last_name:
    text += f" {last_name}"
```

The command can now be used in either of the following ways:

```bash
$ python application.py greet John
$ python application.py greet John Doe
```

It is also possible to let an argument take a list of values (imagine
you want to greet all your friends). For this it must be specified at
the end of the argument list:

```python
class GreetCommand(Command):
    """
    Greets someone

    greet
        {names* : Who do you want to greet?}
        {--y|yell : If set, the task will yell in uppercase letters}
    """
```

To use this, just specify as many names as you want:

```bash
$ python application.py greet John Jane
```

You can access the `names` argument as a list:

```python
names = self.argument("names")
if names:
    text = "Hello " + ", ".join(names)
```

There are 3 argument variants you can use:

| Mode     | Notation                            | Value                                                                                                       |
| -------- | ----------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Required | none (just write the argument name) | The argument is required                                                                                    |
| Optional | `argument?`                         | The argument is optional and therefore can be omitted                                                       |
| List     | `argument*`                         | The argument can contain an indefinite number of arguments and must be used at the end of the argument list |

You can combine them like this:

```python
class GreetCommand(Command):
    """
    Greets someone

    greet
        {names?* : Who do you want to greet?}
        {--y|yell : If set, the task will yell in uppercase letters}
    """
```

If you want to set a default value, you can it like so:

```text
argument=default
```

The argument will then be considered optional.

### Using Options

Unlike arguments, options are not ordered (meaning you can specify them
in any order) and are specified with two dashes (e.g. `--yell` - you can
also declare a one-letter shortcut that you can call with a single dash
like `-y`). Options are _always_ optional, and can be setup to accept a
value (e.g. `--dir=src`) or simply as a boolean flag without a value
(e.g. `--yell`).

> _Tip_: It is also possible to make an option _optionally_ accept a value (so
> that `--yell` or `--yell=loud` work). Options can also be configured to
> accept a list of values.

For example, add a new option to the command that can be used to specify
how many times in a row the message should be printed:

```python
class GreetCommand(Command):
    """
    Greets someone

    greet
        {name? : Who do you want to greet?}
        {--y|yell : If set, the task will yell in uppercase letters}
        {--iterations=1 : How many times should the message be printed?}
    """
```

Next, use this in the command to print the message multiple times:

```python
for _ in range(0, int(self.option("iterations"))):
    self.line(text)
```

Now, when you run the task, you can optionally specify a `--iterations`
flag:

```bash
$ python application.py greet John
$ python application.py greet John --iterations=5
```

The first example will only print once, since `iterations` is empty and
defaults to `1`. The second example will print five times.

Recall that options don\'t care about their order. So, either of the
following will work:

```bash
$ python application.py greet John --iterations=5 --yell
$ python application.py greet John --yell --iterations=5
```

There are 4 option variants you can use:

| Option         | Notation     | Value                                                                               |
| -------------- | ------------ | ----------------------------------------------------------------------------------- |
| List           | `--option=*` | This option accepts multiple values (e.g. `--dir=/foo --dir=/bar`)                  |
| Flag           | `--option`   | Do not accept input for this option (e.g. `--yell`)                                 |
| Requires value | `--option=`  | This value is required (e.g. `--iterations=5`), the option itself is still optional |
| Optional value | `--option=?` | This option may or may not have a value (e.g. `--yell` or `--yell=loud`)            |

You can combine them like this:

```python
class GreetCommand(Command):
    """
    Greets someone

    greet
        {name? : Who do you want to greet?}
        {--y|yell : If set, the task will yell in uppercase letters}
        {--iterations=?*1 : How many times should the message be printed?}
    """
```

### Testing Commands

Cleo provides several tools to help you test your commands. The most
useful one is the `CommandTester` class. It uses a special IO class to
ease testing without a real console:

```python
from greet_command import GreetCommand

from cleo.application import Application
from cleo.testers.command_tester import CommandTester


def test_execute():
    application = Application()
    application.add(GreetCommand())

    command = application.find("greet")
    command_tester = CommandTester(command)
    command_tester.execute()

    assert "..." == command_tester.io.fetch_output()
```

The `CommandTester.io.fetch_output()` method returns what would have
been displayed during a normal call from the console.
`CommandTester.io.fetch_error()` is also available to get what you have
been written to the stderr.

You can test sending arguments and options to the command by passing
them as a string to the `CommandTester.execute()` method:

```python
from greet_command import GreetCommand

from cleo.application import Application
from cleo.testers.command_tester import CommandTester


def test_execute():
    application = Application()
    application.add(GreetCommand())

    command = application.find("greet")
    command_tester = CommandTester(command)
    command_tester.execute("John")

    assert "John" in command_tester.io.fetch_output()
```

You can also test a whole console application by using the
`ApplicationTester` class.

### Calling an existing Command

If a command depends on another one being run before it, instead of
asking the user to remember the order of execution, you can call it
directly yourself. This is also useful if you want to create a \"meta\"
command that just runs a bunch of other commands.

Calling a command from another one is straightforward:

```python
def handle(self):
    return_code = self.call("greet", "John --yell")
    return return_code
```

If you want to suppress the output of the executed command, you can use
the `call_silent()` method instead.

### Autocompletion

Cleo supports automatic (tab) completion in `bash`, `zsh` and `fish`.

By default, your application will have a `completions` command. To register these completions for your application, run one of the following in a terminal (replacing `[program]` with the command you use to run your application):

```bash
# Bash
[program] completions bash | sudo tee /etc/bash_completion.d/[program].bash-completion

# Bash - macOS/Homebrew (requires `brew install bash-completion`)
[program] completions bash > $(brew --prefix)/etc/bash_completion.d/[program].bash-completion

# Zsh
mkdir ~/.zfunc
echo "fpath+=~/.zfunc" >> ~/.zshrc
[program] completions zsh > ~/.zfunc/_[program]

# Zsh - macOS/Homebrew
[program] completions zsh > $(brew --prefix)/share/zsh/site-functions/_[program]

# Fish
[program] completions fish > ~/.config/fish/completions/[program].fish
```
