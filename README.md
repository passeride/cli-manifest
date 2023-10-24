# cli-manifest
A protocol to manifest how a CLI works

## Problem
When using a CLI you have to use `XXX --help` or `man XXX` or `tldr XXXX` to understand it's functions and arguments.
And keeping this information in your interal flesh-memory, you then write commands. 

Or you write arguments out of order, or with the wrong type

But why could not the terminal be more helpful?

We have autocompletion, but this is not shell agnostic, and sometimes annoying to set up.

## Solution: Cli-Manifest.json
A potential solution is a standardized way and format for cli metadata, i suggest that compatible cli's should have a `--export-cli-manifest`[^1], as this is called curing install/update and stored in `/var/cli-man/` as a json with the name of the executable, given multiple executables with the same name, the `hash` field can be used to differentiate.
The `cli-manifest` in a json format, containing the following fields:

```json
{
    "name": "My Todo App",
    "hash": "444ed02fcc0228931fc4df45fe9d5ea9982f07733fe1caf503bb8dbbc378fc46",
    "description": "A simple todo app to append and complete tasks in my Tasks.md",
    "optional_arg_position": "before_positional",
    "idempotent": false,
    "completions": [
        {
            "name": "add",
            "shorthand": "a",
            "desc": "Adds a new task",
            "type": "action",
            "completions": [
                {
                    "name": "Task content",
                    "desc": "The content of the task, this will be pasted into your markdown",
                    "type": "string"
                }
            ]
        },
        {
            "name": "done",
            "shorthand": "d",
            "desc": "Complete a task",
            "type": "action",
            "completions": [
                {
                    "name": "Task index",
                    "desc": "The index of the task you have completed",
                    "type": "interactive",
                    "interactive_command": "list"
                }
            ]
        },
        {
            "name": "list",
            "shorthand": "l",
            "desc": "List tasks",
            "type": "action"
        },
        {
            "name": "target_file",
            "shorthand": "f",
            "desc": "Will target this md file, else it will use default from env $MARKDOWN_TODO_TASK_FILE",
            "type": "argument",
            "optional": true,
            "type": "optional_flag",
            "completions": [
                {
                    "name": "File path",
                    "desc": "Path to file you whish to use",
                    "type": "Path"
                }
            ]
        }
    ]
}

```

## Structure
### Root
 - Name and description
 - `hash` a sha256 hash of the executable, used to validate integrity and differentiate naming collisions
 - `optional_arg_position` Specifies if flags can be set after the main body or should be placed before
 - `idempotent` referes to the idempotency of this program, eg. Will running it multiple times have the same outcome as running it once? Very useful for scripting.
### Completions
This is currently the meat and bone of the cli-manifest, and it's somewhat of a job to get right, so i'm looking forward to input on possible permutations to look out for.
 - `name` Is the text to call this action or flag 
   - If `action`, it should be typed alone with spaces on either side
   - If `argument`, it should be prefixed with two dashes `--` 
 - `shorthand` Is a simpler way to call this action/argument
   - If `action`, it should be typed alone with spaces on either side
   - If `argument`, it should be prefixed with a single dashe `-` 
 - `type`
   - `action` A word that will trigger an action, usually defining the action to be executed
   - `argument` A flag to change some ascpect of the execution with additional parameters
   - `string` User defined input string, defined by a single word or multiple surrounded by single or double qoutes
   - `rest_string` Rest of the words/arguments will be parsed as a single string, this must be the last argument
   - `int` A whole number 
   - `double` A decimal number
   - `interactive`
 - `interactive_command` The arguments to be sendt to the defined executable to get out a newline seperated list of possible options [^2]
 - `optional` Defines if this action/argument can be omitted
 - `completions` A recursive list of sub-completions
Footnotes:
[^1]: There is an argument for this to be something less likely to collide with other sensible use cases, using some special characters could eliminate this.
[^2]: This is of corse a security issue as it's executing code, somewhat hidden from the user, and terminals should have an option to disable interactive autocompletion
