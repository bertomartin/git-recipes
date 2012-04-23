# 5 -- General workflow

## Creating your own Git aliases

### Problem

You find yourself typing the same long combination of Git commands over and over again, which is error prone and time consuming.

### Solution

You can define your own Git aliases, which you can invoke using `git your_alias`. A Git alias can be both an alias for a Git subcommand with some preset options, or a full shell command.

Edit your global Git configuration file at `~/.gitconfig` and add a new section `[alias]`:

    [alias]
        s = status
        l = log --decorate --oneline -a --graph
        sp = !git svn fetch && git svn rebase

You can try out your new alias by invoking `git s`. You will see the exact same output as if you had types `git status`. `git l` will give you a nicely fomatted log output, while `git sp` will do a consecutive `git svn fetch` and `git svn rebase`.

You can also create a new alias using `git-config`. For example, you could create the same alias for `git-status` like so: `git config --global alias.s status`. But typing a long command on the command line, and remembering to quote and escape special characters, is so awkward it's usually easier to just open the configuration file itself.
{: .tip }

### Discussion

Git has many, many subcommands and even more options, formats and configurations. Remembering them all is a hopeless and useless endeavour. Spend the time to figure out what you want once, and then create an alias for it to save it for future use.

Git aliases still require you to type `git` every time, so you might be tempted to create these aliases as regular shell aliases instead, for example in `~/.bashrc`. This would allow you to create an alias like `gl`, which is better than `git l`. There is, however, a neatness factor in having all your Git-specific aliases in a single place. You may get the best of both worlds by simply aliasing `git` to `g` in your shell, so you can use `g s`, `g l` and `g sp`.

Note that although we have created these aliases in the global `~/.gitconfig` file, you could just as well add them to your local Git repository in `./git/config`. This might be helpful in setting up project-specific aliases.

## Generating a list of changes

### Problem

You want to maintain a list of changes to your project. This list will help other developers keep up to date with changes introduced in new versions of the project. Such lists are usually kept in a `HISTORY` or `CHANGES` file, and can be time-consuming to create.

### Solution

We can use `git-log` to list what has changed in a given range. To generate something resembling a changelog, we want to do three things:

* List all the changes from the last tag up to now
* List only the commit subject
* Omit merge commits

This will give us a list good enough to manually tweak in an editor and put into our changelog file.

First, limit the range of commits:

    $ git log v1.0.13..HEAD

Showing just the commits since the `v1.0.13` tag is easy enough, so let's challenge ourselves a little and make it more generic. We can use the `git-describe` program to let Git tell us the latest tag in our current branch:

    $ git log `git describe --abbrev=0`..HEAD

Next, let's format the commits in a sensible way. We would like to use an asterisk as a bullet point and then include the subject and abbreviated hash. Use the `--pretty` option:

    $ git log `git describe --abbrev=0`..HEAD --pretty="* %s (%h)"

This output something similar to the following:

    * Bugfix: remove spaces around post title (a94ce9b)
    * Bugfix: prevent posts from removing feed (a94ce9b)
    * Merge branch "feeds" (6ed9ca2)
    * Added post scheduling (3309ea6)

Finally, let's make sure no merges are listed in this list, since they are not relevant in this context:

    $ git log `git describe --abbrev=0`..HEAD --pretty='* %s (%h)' --no-merges

That will give us:

    * Bugfix: remove spaces around post title (a94ce9b)
    * Bugfix: prevent posts from removing feed (a94ce9b)
    * Added post scheduling (3309ea6)

### Discussion

The `git-log` program has many options to limit the range of commits to show, based on time, hash, author and so forth. Using the `--pretty` option you can specify your own custom format string, so could even have Git generate HTML if you wanted it to.

For a full list of options and format tokens, refer to the `git-log` manual using `man git-log`.

If you find yourself using this awkwardly long comand regularly, make sure to define your own Git alias.

