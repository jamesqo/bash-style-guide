# bash-style-guide

My personal style guide for Bash.

## Context

This will basically be a bunch of notes/bullet points on Bash until I get the energy to organize it into something coherent; this is *not* meant to be a definitive style guide (at least not yet). I'm mostly going to be rambling here because I just want to get these thoughts out of my head as fast as I can, and clean it up later.

## Philosophy

- Don't throw away POSIX compliance for frivolous features, e.g. `function` or hyphens in function names.

- Do use Bashisms when they're useful and you need them, e.g. arrays.

## Notes

- Use `${BASH_SOURCE[0]}` if the script is intended to be sourced and you don't need POSIX compatibility; otherwise, use `$0`.

- Shebang: if your script uses *anything* other than `sh` (like bash, python3, or node) then prefix it with `env`. For example, `#!/usr/bin/env bash`. If you're writing a Bourne-compatible script, use `#!/bin/sh`.

- Functions should be declared like this:

    ```bash
    my_func()
    {
        # code
    }
    ```

- Use `[` instead of `[[`. Only use `[[` when you need it, e.g. `[[ a =~ b ]]` for regexp matching.

    - When you're testing a condition within an if statement, use `[`. Otherwise, use `test`. Example:

        ```bash
        # good
        if [ -z "$var" ]
        # bad
        if test -z "$var"
        
        # good
        test -z "$var" && echo '$var is empty.'
        # bad
        [ -z "$var" ] && echo '$var is empty.'
        ```
    
    - When using `test`, always negate the condition *inside* the brackets. For example:
    
        ```bash
        # good
        [ ! -d "$var" ]
        # bad
        ! [ -d "$var" ]
        ```
    
    - Use `&&` and `||` instead of `test`'s `-a` and `-o` operators; they can cause problems for certain inputs (see [here](http://pubs.opengroup.org/onlinepubs/9699919799/utilities/test.html#tag_20_128_16) for more info) and are deprecated by POSIX.
    
    - Use `-eq` and `-ne` as opposed to `=` and `!=` for comparing numbers. The latters perform *string* comparisons, so for example `test 00 -eq 0` will return true, while `test 00 = 0` will not.
    
        - Also, prefer `-gt` and `-lt` to `>` and `<`. Of course, if you are using `[`, this should not be a choice since `>` will be interpreted as file redirection.

- Include a space after the hash in your comments. For example:

    ```bash
    # good
    #bad
    ```

    This not only looks better, it prevents shebang problems if your comment starts with a `!`.

- Unless necessary, do not use quotes when assigning something to a variable. For example:

    ```bash
    # good
    command=help
    # bad
    command='help'
    command="help"
    ```

    This prevents ugliness when you have many nested commands. For example, this:
    
    ```bash
    scriptpath="$(cd "$(dirname "$0")" && pwd -P)"
    ```
    
    could be rewritten as this:
    
    ```bash
    scriptdir=$(dirname "$0")
    scriptpath=$(cd "$scriptdir" && pwd -P)
    ```
    
    This prevents problems in some code editors which don't have overly sophisticated Bash support. (This also works for inputs with spaces, by the way.)
    
    - In general, try to avoid quoting things when possible. Only quote literals when they contain whitespace, dollar signs, or quotes themselves.
        
        - It is also acceptable to quote them if they represent some kind of message to be shown to the user, e.g. `echo 'Hello!'`

- When redirecting to a file descriptor, omit the space and drop the `1`:

    ```bash
    # good
    do-something >&2
    # bad
    do-something 1>&2
    do-something > &2 # also syntactically incorrect
    ```
    
    When redirecting to a file, add a space after the `>` and also drop the `1`:
    
    ```bash
    # good
    do-something > /dev/null
    do-something 2> /dev/null
    # bad
    do-something 1> /dev/null
    do-something >/dev/null
    ```
    
    - Dilemma: Should `&>` be used? It is a bashism and [deprecated](http://wiki.bash-hackers.org/scripting/obsolete) according to the Bash Hackers wiki, but I personally prefer it.

- Variable names should be all-lowercase, and mashed together where possible (e.g. `scriptpath`). Use underscores to separate them if they start to get uncomfortably long, for example `really_long_name` instead of `reallylongname`.

    - Variables exported to child processes should be all uppercase, for example `export DO_SOMETHING=1` instead of `export do_something=1` or `export dosomething=1`.

- Prefer short options to long options, e.g. `sed -i` as opposed to `--in-place`. Also mush them together where possible, e.g. `ls -1d` instead of `ls -1 -d`.

- Booleans can be declared and used in the following way:

    ```bash
    mybool=true
    
    case "$input" in
        5) mybool=false ;;
    esac
    
    if $mybool; then
        # code
    fi
    ```

- Prefer using `&&` and `||` to full-fledged if statements. For example:

    ```bash
    # good
    type foo &> /dev/null && foo 'Hello, world!'
    # bad
    if type foo &> /dev/null; then
        foo 'Hello, world!'
    fi
    ```

- `case` statements should be formatted the following way:

    ```bash
    case "$input" in
        123)
            something
            ;;
        *)
            something_else
            ;;
    esac
    ```
    
    If the code executed for a case is a single statement, you are permitted to leave it on the same line as the case like this:
    
    ```bash
    case "$input" in
        123) something ;;
        *) something_else ;;
    esac
    ```

- `if` statements should have the `then` on the same line, but loops like `while` and `for` should have `do` on the next line. For example:

    ```bash
    # good
    if foo; then
    fi
    for arg in "$@"
    do
    done
    
    # bad
    if foo
    then
    fi
    for arg in "$@"; do
    done
    ```

- Functions should be self-contained; if a function relies on a global variable, it should either be separated out as a new function or supplied as an argument. For example:

    ```bash
    # good
    my_func()
    {
        do_something "$1"
    }
    
    # or...
    get_something()
    {
        # code...
    }
    
    my_func()
    {
        do_something "$(get_something)"
    }
    
    # bad
    global_var=$(get_something)
    
    my_func()
    {
        do_something "$global_var"
    }
    ```
    
    - Dilemma: Should `local` be used in functions? I've never used the keyword much, and it doesn't seem to be defined by POSIX. However, most shells (even 'lean' ones like `dash` and `ash`) seem to support it, so I'm leaning towards yes.

- Prefer single quotes to double quotes, unless you 1) need string interpolation or 2) have double quotes in your literal. For example:

    ```bash
    # good
    echo 'blah blah blah'
    echo "you're so amazing"
    echo "The value is: $var"
    ```

- If you're writing a script that basically forwards all of its arguments to another one, then consider using `exec` to avoid spawning a subshell. For example, in some systems `egrep` is just a wrapper script for `grep` that looks like this:

    ```bash
    #!/bin/sh
    
    exec grep -E "$@"
    ```

- Prefer `$(` to backticks. The latter is legacy syntax and is hard to nest properly.

- When doing arithmetic with `$((`, do not prefix variable names with a dollar sign. For example:

    ```bash
    # good
    end=$((start + time))
    # bad
    end=$(($start + $time))
    ```
    
    Also, preserve spaces between the arithmetic operators if possible. For example:
    
    ```bash
    # good
    three=$((five - two))
    # bad
    three=$((five-two))
    three=$(( five - two )) # not too many
    ```

- Omit unnecessary whitespace when using `(` or `$(` to spawn subshells. For example:

    ```bash
    # good
    (cd .. && pwd)
    contents=$(cat meow)
    # bad
    ( cd .. && pwd )
    contents=$( cat meow )
    ```
