# bash-style-guide

My personal style guide for Bash.

## Context

This will basically be a bunch of notes/bullet points on Bash until I get the energy to organize it into something coherent; this is *not* meant to be a definitive style guide (at least not yet). I'm mostly going to be rambling here because I just want to get these thoughts out of my head as fast as I can, and clean it up later.

## Notes

- Should `${BASH_SOURCE[0]}` or `$0` be used to get the script's path? The first is and resilient and handles things like sourcing the script, the second is sh-compatible and more concise.

- *Always* use `#!/usr/bin/env bash` instead of `#!/bin/bash` at the top of the script. Always.
    - This is more of an issue than you might think; in systems like FreeBSD, for example, `bash` is in `/usr/bin/local` as opposed to `/bin`.

- Dilemma: should functions be declared like this:

```bash
function my-function {
    # code
}
```

or like this:
 
```bash
my_function()
{
    # code
}
```
 
The first one is (in my opinion) more visually appealing, but the second version is less likely to have problems with older shells.

- Use `[` instead of `[[`. The latter not only ties you down to Bash, it is also less visually appealing and can get you into the habit of forgetting to quote things. Only use `[[` when you need it, e.g. `[[ a =~ b ]]` for regexp matching.
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

- Include a space after the hash in your comments. For example:

```bash
# good
#bad
```

This not only looks better, it prevents shebang problems if your comment starts with a `!`.
