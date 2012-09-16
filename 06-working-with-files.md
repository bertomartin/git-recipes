# 6 -- Working with files

## Ignore files across all your projects

### Problem

When working on a Mac, your Git repositories tend to get littered with `.DS_Store` files, while Windows throws around `Thumbs.db` and `Desktop.ini` everywhere. If you are a Vim user, you might want to get rid of `.swp` files. You want to let Git ignore these files in all your projects.

### Solution

Git allows you to set a global `.gitignore` file. This files behaves exactly the same as a project-specific `.gitignore` file, but it is always used in every project. In this file you can define pattens for generic files you never want to include in version control:

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

This will tell Git to record the removal, but keep the actual file around, so you don't have to re-create it. The `--cached` option might seem cryptically named, until you learn that Git's _index_ (staging area) used to be called the _cache_. You can read `git rm --cached file` as 'git, remove file from index'.

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

## Removing a file from the repository entirely 

### Problem

You have accidentally committed a file with sensitive information, such as API
keys or passwords, to the repository. This file needs to be removed and in no
way be recoverable in the repository.

### Solution

Github offers a simple solution on its website to achieve the desired effect:

    $ git filter-branch --index-filter\
    'git rm --cached --ignore-unmatch passwords.yml' \
    --prune-empty --tag-name-filter cat -- --all

### Discussion

How does this work? `git-filter-branch` is a low-level Git tool that allows you
to rewrite the entire Git history using one or more filters. This example uses
the `--index-filter` option, which allows you to rewrite the the index for
every commit. The operation used to filter each commit index is the `git-rm`
command to remove the offending file.

The `--pruny-empty` ensures empty commits will be removed. An commit would be
empty, if it only touched the `passwords.yml` file. Since that file will be
removed, any commits affecting only that file become meaningless and can safely
be removed.

Finally, the `--all` option indicates to rewrite every branch and tag, while
`--tag-name-filter cat` recreates every tag with the same name (the argument to
`--tag-name-filter` should take a tag name as input and output a new tag name).

Beware when using operations like this. Re-read the first paragraph: this
command rewrites the entire Git history. That means that as far as Git is
concerned, you end up with an entirely new repository, completely unrelated to
one you started with. Every object in the object database is touched and
changed. Keep that in mind when working in a team. Your rewritten repository
and your team mate's repository will no longer share any commits, meaning you
will no longer be able to push and pull from each other repositories.
Sometimes, operations like this are necessary, but make sure to plan and
coordinate a team-wide switch to the rewritten repo.

## Untracking a file without removing it

### Problem

You have accidentally committed a large log file to your repository. You do not
want to track this file with Git, but it contains relevant information you do
want to keep around.

### Solution

Git allows you to remove a file without actually deleting it from disk:

    $ git rm --cached log/development.log

Note that the Git index used to be called the "cache", hence the `cached`
option. This will record the removal of the file in the index, but leave the
file itself alone.

## Searching files across the repository

### Problem

You want to search your entire project for a string or regular expression.

### Solution

You could use tools like `grep` or `ack` to search across all the files on
disk. `ack` is even smart enough to not search inside the meaningless `.git`
directory. But Git also offers us `git-grep`, which can be used to search in
files Git knows about. This automatically skips any irrelevant files.

Use it as you would regular `grep`:

    $ git grep foo

### Discussion

Using `git-grep` instead of other tools has a couple of advantages:

1. You can search only files Git knows about and ignore any untracked or
   ignored files.
2. You can search in files in the index using `git grep --cached`, which others
   cannot.
3. You can search in the current working tree, or in any other treeish object.
   For example, to search files that exist in the second-to-last commit on the
   branch "feature-branch", use `git grep feature-branch^2 foo`.

## Removing untracked files from the working tree

### Problem

Your working tree contains a lot of untracked files you do not need to keep around nor want to track with Git. For example, running a Ruby project with Rubinius will leave your project littered with a `.rbc` file for every `.rb` file.

### Solution

Rather than removing each file by hand or shell scripting your way out of this, you can use the `git-clean` program to remove files Git does not track:

    $ git status -s
    ?? lib/waterbug/version.rbc
    $ git clean
    Removing lib/waterbug/version.rbc

### Discussion

`git-clean` comes with a few interesting options:

* you can specify wether it should only remove files Git knows nothing about,
  or also remove files that Git ignores (based on the `.gitignore` file) using
  `-x`. To _only_ remove file Git ignores, use `-X`.
* You can test what Git is going to remove using `--dry-run` (or `-n`). Git
  will give you more or less the same output, but not actually remove anything.
  You can make this behaviour the default behaviour using `git config
  clean.requireForce true`. To _force_ a clean, use `--force` (or `-f`).
* Git will, by default, not remove directories. Instruct it to do so using
  `-d`.
* You can limit where files will be removed by supplying a path as an extra
  option. Normally, `git-clean` will operate on the current directory and any
  below it, but you can only affect the `lib` directory using `git clean lib`.

[Kramdown]: http://kramdown.rubyforge.org/
