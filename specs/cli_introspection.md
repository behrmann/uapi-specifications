---
title: UAPI.X CLI-Introspection Specification
category: Interfaces
layout: default
version: 0.0
SPDX-License-Identifier: CC-BY-4.0
weight: 1
aliases:
- /UAPI.18
- /18
---

# UAPI.X The CLI-Introspection Specification

| Version | Changes |
|---------|---------|
| 0.0     | WIP     |

This document defines a JSON schema defining a machine-readable equivalent of
what programs commonly output when they are called with the `--help` or an
equivalent option.

This output can be used,
among other things,
to check whether a program has a given option or to automatically generate
command-line completion for different shells.

## Target Audience

The target audience for this specification are:

* People writing runnable programs that have command-line interfaces,
* authors of tab-completion helpers, and
* people interested in automatic testing of documentation and self-documenting
  interfaces.

## The Schema

The schema is defined as

```json
{
    "mediaType" : "application/vnd.uapi-group.cli-introspection",
    "commands": [<command object>, …]
}
```

where `mediaType` is the fixed string
`application/vnd.uapi-group.cli-introspection-0` and `commands` is a non-empty
JSON array of *command objects*.

Further additions to this specification will only be additive and non-backward
compatible changes may only be introduced by defining a new `mediaType`.

The `commands` array defines the actual CLI and a single binary can define
arbitrarily many CLIs,
but must define at least one.
Defining multiple CLIs is relevant for multicall binaries that behave
differently depending on with what name they are called,
e.g the basename of `argv[0]`,
`program_invocation_short_name` in the GNU system
context,
or in the context of scripting languages the name under which they are imported

### Command ojects

The command object is recursively defined as follows.

```json
{
    "names" : ["name1", "name2", …],
    "argspec": ["value1", "value2", …]
    "version": "myversion",
    "features": ["feature1", "feature2", …]
    "abstract": ["paragraph1", "paragraph2", …],
    "postscript": ["paragraph1", "paragraph2", …],
    "help": "short help"
    "options": [<option object>, …],
    "verbs": [<command >, …],
    "documentation": ["doc1", …],
    "project": "myproject",
    "isDeprecated": false
}
```

All keys except `names` are optional and are treated as empty string or empty
array when missing.

`names` is a non-empty array of string names for that command.
The first element of that array is the primary name of that command.
Further names can be added as aliases,
e.g. for backward compatibility.

`argspec` is an array of argspec objects describing the arguments to the command
after the name of the command,
i.e. C language `argv` excluding the element with index 0.

`version` defines the version string of the program.
This string should be string compatible with UAPI.10 Version Format
Specification-compatible.

`features` defines an array of strings that define builtin-in features of this
program.

`abstract` and `postscript` define arrays of strings that are shown respectively
before and after the description of options in the program's help output.
Each element of the array represents a paragraph of text.
Both are meant for standalone help output, e.g. for the top-level help output of
a program or specific help output of a verb in cases where they have their own
help output.

`help` defines the help text of the command. This is useful for single-line
usage information as well as for short descriptions of verbs in the list of
options.

`options` define a CLI's positional arguments and options and are described in a
section below.

`verbs` are command objects that describe a program's verbs,
also called subcommands.
Verbs may recursively define further verbs up to a maximum depth that should not
be larger than 16.

`documentation` is an array of string values describing URIs referencing
documentation for this command,
see `man:uri(7)`for a description of valid URIs.
Preferably the URIs should be URLs starting with `https://` as these are widely
supported in modern terminals.

`project` is a string describing what this command belongs to.
This may the package that installed it or the project that produced it.

`isDeprecated` is a boolean describing whether the command has been

### Argspec objects

Argspec objects describe the arguments passed to the program,
specifically in what order verbs, arguments, and options are passed to the
program.

```json
{
    "name": "myname",
    "value_name": "VALUE_NAME",
    "type": "argument",
    "optional": true,
    "default": null
}
```

All keys except `name` and `type` are required and are treated as empty string,
false or null if missing.

`name` is a non-empty string describing the internal name of a value.

`value_name` is a string describing the name that is shown in the argspec of the
command.
If it is empty, it defaults to the value of `name`.
This concept is sometimes also called metavar.

`type` is a string describing what the argument is. It is either of
- `argument`, a positional argument to the command,
- `option`, which is command line switch from the `options` array of a command, or
- `verb`, which is a verb from the `verbs` array of a command.

`optional` is a boolean describing whether an argspec object of type `argument`
or `verb` must be passed or not.

`default` is a string or null,
describing what the default value for this argument is.
`null`, the default, means there is no default.
`default` is ignored for argspec objects of type `option`.
For argspec objects of type `verb` a non-`null` value must be a `name` of a
commands object in the verbs array.

### Option objects

Option objects arguments define both positional arguments as well as options,
commonly prefixed with `-` or `--`,
of a program.
Both positional arguments and options will be referred to as options in this
section,
when a distinction is not explicitly made.

```json
{
    "names": ["-h","--help"],
    "argument": "no_argument",
    "help": "Show this help",
    "group": [""],
    "values": [<value object>],
    "section": "option block 1",
    "isDeprecated": false
}
```

All keys except `names` and `argument` are optional and are treated as empty
when missing.

`names` is a non-empty array of strings defining the name of an option.
Names starting with dashes define options and names not starting with dashes
define positional arguments.
An option object may define only options or arguments,
but not both for the same object,
i.e. an option object may not have names both with and without dashes.
Options prefixed with a single dash (`-`) are called short options and options
prefixed with two dashes (`--`) are called long options.
Short options are usually followed by a single chracters,
whereas long options can be a longer string.
Short otions may be followed by multiple characters,
but this is discouraged.

`argument` defines the argument type, which is one of the strings:
-`no_argument`,
-`required_argument`, or
-`optional_argument`.

`help` is a string that defines the help text of the option.

`group` is an array of string that defines sections in which this option should
be shown.
This is only for display-purposes.

`values` is an array of *value objects* describing the values this option may
take.
Value objects are described in a section below.

`section` is a string to visually group options into separate lists.

`isDeprecated` is a boolean describing whether this option has been deprecated.

### Value objects

Value objects describe a value that is passed as an argument,
either to a positional argument or an option.

```json
{
    "name": "myname",
    "help": "help text",
    "default": true,
    "isDeprecated": false
}
```

All keys except `name` are optional and are treated as empty string or false
when missing.

`name` is a string describing the name of a value.

`help` is a string describing the help text that should be shown for the value.

`default` is a boolean describing whether this value is the default value for
the option this is a value for.
