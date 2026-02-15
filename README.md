# GDScript Toolkit (Custom Fork)

> **Note**: This is a custom fork of [Scony/godot-gdscript-toolkit](https://github.com/Scony/godot-gdscript-toolkit) with additional formatting options and behavior changes.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Custom Features

This fork includes the following custom modifications to the formatter:

1. **`--single-blank-lines` flag**: Use single blank lines between class methods instead of the default double blank lines
2. **Always-multiline enums**: Enum definitions are always formatted with one element per line, regardless of length
3. **Always-multiline dictionaries**: Dictionary literals are always formatted with one key-value pair per line, regardless of length

### Example of custom formatting:

```gdscript
# Enums are always multiline (one element per line)
enum Status {
	IDLE,
	RUNNING,
	COMPLETED
}

# Dictionaries are always multiline (one pair per line)
func get_config() -> Dictionary:
	return {
		"name": "example",
		"value": 42
	}

# Using --single-blank-lines flag
class MyClass:
	func foo():
		pass

	func bar():  # Single blank line instead of double
		pass
```

## About

This project provides a set of tools for daily work with `GDScript`. At the moment it provides:

- A parser that produces a parse tree for debugging and educational purposes.
- A linter that performs a static analysis according to some predefined configuration.
- A formatter that formats the code according to some predefined rules.
- A code metrics calculator which calculates cyclomatic complexity of functions and classes.

## Installation

To install this custom fork, you need `python3` and `pip`.

### Installing from this fork

```bash
# Clone this repository
git clone https://github.com/YOUR-USERNAME/godot-gdscript-toolkit.git
cd godot-gdscript-toolkit

# Install in editable mode
pip install -e .
# or
pipx install .
```

### Installing the original version

For the original upstream version without custom modifications, see [Scony/godot-gdscript-toolkit](https://github.com/Scony/godot-gdscript-toolkit):

```bash
pip3 install "gdtoolkit==4.*"
# or
pipx install "gdtoolkit==4.*"
```

## Linting with gdlint [(more)](https://github.com/Scony/godot-gdscript-toolkit/wiki/3.-Linter)

To run a linter you need to execute `gdlint` command like:

```
$ gdlint misc/MarkovianPCG.gd
```

Which outputs messages like:

```
misc/MarkovianPCG.gd:96: Error: Function argument name "aOrigin" is not valid (function-argument-name)
misc/MarkovianPCG.gd:96: Error: Function argument name "aPos" is not valid (function-argument-name)
```

## Formatting with gdformat [(more)](https://github.com/Scony/godot-gdscript-toolkit/wiki/4.-Formatter)

**Formatting may lead to data loss, so it's highly recommended to use it along with Version Control System (VCS) e.g. `git`**

To run a formatter you need to execute `gdformat` on the file you want to format.

### Custom formatting options

This fork adds the `--single-blank-lines` flag:
```bash
gdformat --single-blank-lines your_file.gd
```

This uses single blank lines between class methods instead of the default double blank lines.

### Example

Given a `test.gd` file:

```
class X:
	var x=[1,2,{'a':1}]
	var y=[1,2,3,]     # trailing comma
	func foo(a:int,b,c=[1,2,3]):
		if a in c and \
		   b > 100:
			print('foo')
func bar():
	print('bar')
```

when you execute `gdformat test.gd` command, the `test.gd` file will be reformatted as follows:

```
class X:
	var x = [1, 2, {
		'a': 1
	}]  # Note: dicts always use multiple lines in this fork
	var y = [
		1,
		2,
		3,
	]  # trailing comma

	func foo(a: int, b, c = [1, 2, 3]):
		if a in c and b > 100:
			print('foo')


func bar():
	print('bar')
```

**Note**: In this fork, enums and dictionaries are always formatted on multiple lines with one element/pair per line, regardless of their length or content.

## Parsing with gdparse [(more)](https://github.com/Scony/godot-gdscript-toolkit/wiki/2.-Parser)

To run a parser you need to execute the `gdparse` command like:

```
gdparse tests/valid-gd-scripts/recursive_tool.gd -p
```

The parser outputs a tree that represents your code's structure:

```
start
  class_def
    X
    class_body
      tool_stmt
      signal_stmt	sss
  class_def
    Y
    class_body
      tool_stmt
      signal_stmt	sss
  tool_stmt
```

## Calculating cyclomatic complexity with gdradon

To run cyclomatic complexity calculator you need to execute the `gdradon` command like:

```
gdradon cc tests/formatter/input-output-pairs/simple-function-statements.in.gd tests/gd2py/input-output-pairs/
```

The command outputs calculated metrics just like [Radon cc command](https://radon.readthedocs.io/en/latest/commandline.html#the-cc-command) does for Python code:
```
tests/formatter/input-output-pairs/simple-function-statements.in.gd
    C 1:0 X - A (2)
    F 2:1 foo - A (1)
tests/gd2py/input-output-pairs/class-level-statements.in.gd
    F 22:0 foo - A (1)
    F 24:0 bar - A (1)
    C 18:0 C - A (1)
tests/gd2py/input-output-pairs/func-level-statements.in.gd
    F 1:0 foo - B (8)
```

## Using gdtoolkit's GitHub action

In order to setup a simple action with gdtoolkit's static checks, the base action from this repo can be used:

```
name: Static checks

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  static-checks:
    name: 'Static checks'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: Scony/godot-gdscript-toolkit@master
    - run: gdformat --check source/
    - run: gdlint source/
```

See the discussion in https://github.com/Scony/godot-gdscript-toolkit/issues/239 for more details.

## Using gdtookit in pre-commit

To add gdtookit as a pre-commit hook check the latest GitHub version (eg `4.2.2`) and add the followingto your `pre-commit-config.yaml` with the latest version.

```Yaml
repos:
  # GDScript Toolkit
  - repo: https://github.com/Scony/godot-gdscript-toolkit
    rev: 4.2.2
    hooks:
      - id: gdlint
        name: gdlint
        description: "gdlint - linter for GDScript"
        entry: gdlint
        language: python
        language_version: python3
        require_serial: true
        types: [gdscript]
      - id: gdformat
        name: gdformat
        description: "gdformat - formatter for GDScript"
        entry: gdformat
        language: python
        language_version: python3
        require_serial: true
        types: [gdscript]
```

## Development [(more)](https://github.com/Scony/godot-gdscript-toolkit/wiki/5.-Development)

Everyone is free to fix bugs or introduce new features. For that, however, please refer to existing issue or create one before starting implementation.

## Contributing to Upstream

This fork contains opinionated formatting changes that may not align with the upstream project's goals. If you want to contribute general bug fixes or features that would benefit the wider community, please consider contributing directly to the [original repository](https://github.com/Scony/godot-gdscript-toolkit).

## License

This fork maintains the same MIT license as the original project. See the LICENSE file for details.

## Credits

Original project by [Paweł Lampe](https://github.com/Scony) and contributors. Custom modifications for personal/team preferences.
