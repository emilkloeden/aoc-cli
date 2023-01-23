# Contributing

`aoc` is a tool to support automation of the automatable components in Advent of Code. It is written in python and can be extended by writing plugins for different languages/interpreters.

The `aoc` command exposes a number of subcommands that perform the following actions:

| command      | action                                                                                                                                                                                                       |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `aoc init`   | Scaffolds out a directory and file structure, populating files from template text                                                                                                                            |
| `aoc get`    | Downloads that day's input (if it exists) based on the current directory when the command is called. Requires a session key to perform                                                                       |
| `aoc run`    | Executes code for either part 1 or part 2 for a particular day, depending on the 'part' parameter value passed and based on the current directory, it derives the year, day and language to use in execution |
| `aoc submit` | Executes the code as per `aoc run` and then submits it to https://adventofcode.com for evaluation                                                                                                            |
| `aoc open`   | Opens the current day's page on https://adventofcode.com. Determines day and year based on current directory.                                                                                                |

One nice thing about `aoc` is that each command excluding `init` is context-aware based on the directory in which it is called. To maintain this, all plugins must scaffold a project in such a way as so the year, language and day can be derived from the folder structure.

By default `aoc` scaffolds out a directory structure python project with the structure

```
<base_dir>\
└──<year>
    └──python
        └──01
            └──day.py
        └──02
            └──day.py
        └──...
```

However a plugin should not need to follow this structure.

# Creating a plugin

To create a plugin create a new python project with a pyproject.toml file
ensure it has an entry like so:

```toml
[project.entry-points."aoc.plugins"]
something = "<your_plugin_name>"
```

e.g here's one for a deno plugin

```toml
[project.entry-points."aoc.plugins"]
deno = "aoc_deno"
```

All plugins are required to implement and expose the following variables, classes and methods as the `aoc` core application will call each.

## variables

**language**
e.g.

```py
# aoc_deno
# └──__init__.py
__language__ = "deno"
```

As you can see the term "language" isn't the best as sometimes "interpreter", or "runtime" might be more accurate. Either way, this variable is required to compose a list of valid values for the `--language` option of the `init` subcommand. Additionally it must denote the "language" component of your scaffolding folder structure

## classes and methods

| class       | variable/method | receiving                                                                                                                   | performs                                                                   |
| ----------- | --------------- | --------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Initialiser | **init**        | year:int, location: Path                                                                                                    |                                                                            |
| Initialiser | initialise      |                                                                                                                             | creates project directories and files                                      |
| Getter      | **init**        | p: Path                                                                                                                     |
| Getter      | get_input       |                                                                                                                             | downloads daily input to location set in **init**                          |
| Runner      | **init**        | part: int                                                                                                                   | imports current day's file (derived current working directory) as a module |
| Runner      | run             | executes the current day's file, specifically the part_1 or part_2 method depending on the part variable passed to **init** |
| Submitter   | **init**        | part: int                                                                                                                   | imports current day's file (derived current working directory) as a module | submit | executes the method for the day derived from the cwd and part passed in to **init** and submits the answer to the adventofcode.com site for verification |
| Opener      | **init**        |                                                                                                                             | derives language, year and day from cwd                                    |
| Opener      | open            | opens the page for the given year and day in a webbrowser                                                                   |
