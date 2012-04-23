# 6 -- Working with files

## Ignore files across all your projects

### Problem

When working on a Mac, your Git repositories tend to get littered with `.DS_Store` files, while Windows throws around `Thumbs.db` and `Desktop.ini` everywhere. If you are a Vim user, you might want to get rid of `.swp` files. You want to let Git ignore these files in all your projects.

### Solution

Git allows you to set a global `.gitignore` file. This files behaves exactly the same as a project-specific `.gitignore` file, but it is always used in every project. In this file you can define pattens for generci files you never want to include in version control:

    .DS_Store
    .svn
    .bundle
    .yardoc

To let Git know to take this file into account, you have to add it to the configuration:

    $ git config --global excludesfile ~/.gitignore

We might have given it any other name, as long as Git can read the file. But it's nice to keep a consistent name.

## Ignoring a file already tracked by Git

### Problem

By accident, you have comitted a log file to your repository. There is no sense to track such a file, so you want to remove it from your repository. However, you do not want to remove the file from disk, as you are still using it -- you only want to tell Git to no longer track it.

### Solution

Telling Git to ignore a file is easy enough: simply add a new line to the `.gitignore` file:

    $ echo "tmp/development.log" >> .gitignore
    $ git add .gitignore
    $ git commit -m "Ignore development log"

This will, however, not remove the file from your index. You could do a `git rm tmp/development.log`, but this will also remove the file from disk. To _only_ remove the file from the Git index, you can use the `--cached` option:

    $ git rm --cached tmp/development.log

This will tell Git to record the removal, but keep the actual file around, so you don't have to re-create it.

## Ignoring all files in a directory but one

### Problem

You have a directory with a lot of generated files, which you do not want to keep under version control. You would like to just ignore the entire directory in your `.gitignore` file, but there's this one file you _do_ want to keep around. You want to create an exception for that file.

### Solution

The `.gitignore` file allows you to add exceptions by prefixing a line with an exclamation mark (`!`). To ignore all the files in the `./bin` directory, except for a single file, you can add:

    bin/*
    !bin/foo

## Quickly show a file from another branch

### Problem

You want to get the contents of a file from one branch while curently having checked out another branch, without having to resort to switching branches, copying the file to a temporary location, switching back to the first branch and reading the temporary file.

This tip is a great way to update the public website of your open source project on Github. Github makes it easy to generate a website on a new `gh-pages` branch using predefined templates and you project README file. Once generated, you need to update the site yourself.
{: .tip }

### Solution

The `git-show` command can be used to show various types of Git objects, like tags, commits or blobs. Given no arguments, it will show you the log output for the latest commit on the current branch. But you can specify which object to show. We can get the contents of the `README.md` file on the `master` branch by calling:

    $ git show master:README.md

This will dump the contents to standard output, so you can use regular pipes and redirects to process it. For example, you might to use the [Kramdown][] Ruby gem to convert this markdown file to an HTML page in the working directory:

    $ git show master:README.md | kramdown --template document > readme.html

Or, you might even load the file output straight into your editor. For example, using Vim, you can edit an existing page and insert the contents of your `README.md` file as HTML in one go:

    :r ! git show master:README.md | kramdown

### Discussion

The `master:README.md` notation looks simple enough, but you can actually pass many different things to `git-show`, some more useful than others. You can pass in hashes or tag names and Git will print out sensible information, taking many of the same options as `git-log` would. But here are a few extra tricks you might want to use when trying to get to a specifc file somewhere in your project history:

`git show master~10:README.md`
: Show the contents of `README.md` as it was 10 commits ago in the `master` branch.

`git show master^{/docs}:README.md`
: Show the contents of `README.md` as it was in the latest commit on the `master` branch whose commit message matches `docs`.

`git show master:README.md master:LICENSE`
: Show the concatenated contents of both `README.md` and `LICENSE` as they are currently on the `master` branch.

For more information, see the documentation on specifying Git revisions: `man gitrevisions`.

[Kramdown]: http://kramdown.rubyforge.org/
