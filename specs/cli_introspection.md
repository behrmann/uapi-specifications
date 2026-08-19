---
title: UAPI.X CLI-Introspection Specification
category: Booting
layout: default
version: 0.0
SPDX-License-Identifier: CC-BY-4.0
weight: 1
aliases:
- /UAPI.X
- /X
---

# UAPI.X The CLI-Introspection Specification

| Version | Changes |
|---------|---------|
| 0.0     | WIP     |

This document defines a JSON schema defining a machine-readable equivalent of
what programs commonly output when they are called with the `--help` or an
equivalent option.

This output can be used, among other things, to check whether a program has a
given option or to automatically generate command-line completion for different
shells.

## Target Audience

The target audience for this specification are:

* People writing runnable programs that have command-line interfaces.

## The Schema

The schema is defines as

```json
{
    "mediaType" : "application/vnd.uapi-group.cli-introspection-0",
    "commands": [<command object>, …]
}
```

where `mediaType` is the fixed string
`application/vnd.uapi-group.cli-introspection-0` and `commands` is a non-empty
JSON array of *command objects*.

Further additions to this specification will only be additive and non-backwards
compatible changes may only be introduced by defining a new `mediaType`.

The `commands` array defines the actual CLI and a single binary can define
arbitrarily many CLIs, but must define at least one. Defining multiple CLIs is
relevant for multicall binaries that behave differently depending on with what
name they are called, e.g `argv[0]` or in the context of scripting languages
under what name they are imported

### Command ojects

The command object is recursively defined as follows.

```json
{
    "names" : ["name1", "name2", …],
    "version": "myversion",
    "features": ["feature1", "feature2", …]
    "abstract": ["paragraph1", "paragraph2", …],
    "postscript": ["paragraph1", "paragraph2", …],
    "options": [<option object>, …],
    "verbs" : [<command >, …]
}
```

All keys except `names` are optional and are treated as empty string or empty
array when missing.

`names` is a non-empty array of string names for that command. The first element
of that array is the primary name of that command. Further names can be added as
aliases, e.g. for backwards-compatibility.

`version` defines the version string of the program. This string should be a
UAPI.10 Version Format Specification-compatible.

`features` defines an array of strings, that can be empty, that define
builtin-in features of this program.

`abstract` and `postscript` define arrays of strings that are shown respectively
before and after the description of options in the program's help output. These
arrays may be empty.

`options` define a program's arguments and options and are described in a section below.

`verbs` are command objects that describe a program's verbs, also called
subcommands. Verbs may recursively define further verbs up to a maximum depth
that should not be larger than 16.

### Option objects

Option objects arguments define both (positional)arguments as well as options
(commonly prefixed with `-` or `--`) of a program. Both arguments and options
will be referred to as options in this section, when a distinction is not
explicitly made.

```json
{
    "names": ["-h","--help"],
    "argument": "no_argument",
    "help": "Show this help",
    "group": [""],
    "conflicts": [],
    "values": [<value object>]
}
```

All keys except `names` and `argument` are optional and are treated as empty
string or empty array when missing.

`names` is a non-empty array of strings defining the name of an option. Names
starting with dashes define options and names not starting with dashes define
(positional) arguments. An option object may define only options or arguments,
but not both for the same object, i.e. an option object may not have names both
with and without dashes. Options prefixed with a single dash (`-`) are called
short options and options prefixed with two dashes (`--`) are called
long-options. Short options are usually followed by a single chracters, whereas
long options can be a longer string. Short otions may be followed by multiple
characters, but this is discouraged.

`argument` defines the argument type TODO

`help` is a string that defines the help text of the option.

`group` is an array of string that defines sections in which this option should
be shown. This is only for display-purposes.

`conflicts` defines an array of strings referring to the names of other options
this option may not be called together with.

`values` is an array of *value objects* describing the values this option may
take. Value objects are described in a section below.

### Value objects

Value objects describe a value that is passed as an argument, either to a
(positional) argument or an option.

```json
{
    "name": "myname",
    "help": "help text",
    "default": True
}

All keys except `name` are optional and are treated as empty string or false
when missing.

`name` is a string describing the name of a value.

`help` is a string describing the help text that should be shown for an option.

`default` is a boolean describing whether this value is the default value for
the option this is a value for.
