# 2 -- A quick introduction of Git

## A typical Git workflow

### Creating and cloning repositories

Create a new Git repository is easy with the `git-init` program:

    $ git init .
    Initialized empty Git repository in /path/to/repo/.git/

This will initialize a new, empty repository in the current directory. You could run `git init subdir` to have Git first create a subdirectory "subdir" and create a new repository in there.

Oftentimes, you will want to continue working on an existing project. The `git-clone` program can copy an entire _other_ repository to a specified location. For example, you could clone a repistory hosted on Github to your local machine:

    $ git clone git@git.github.com:username/project.git

By default, this will create a `project` directory in your current working directory and copy all the files there. You could supply a path as a third argument to tell Git where the repository should be stored:

    $ git clone git@git.github.com:username/project.git /code/projects/cloned/

### Staging and committing changes

Git uses a two-step process for creating new commits. Assuming you have made a couple of changes to files in your project directory, you first add them to the staging area. the staging area is a sort of preview of your upcoming commit: any change you want to include in your next commit, needs to be added to the staging area first. Then, once you are happy with the contents of the staging area, you can save it in a new commit in the repository.

With other version control systems, such as Subversion, your working copy and the staging area are basically the same thing: any change in your working copy will be part of the next commit. Git allows to pick and choose exactly what changes will be part of the next commit.

Staging a file is easy enough with the `git-add` program. You can add all the changes in this directory and any directory below it:

    $ git add .

Or, you can stage just a single file:

    $ git add README.md

You can inspect the status of the working directory and the index using the `git-status` program:

    $ git status
    ...

Here you can see that one change you made is staged to be committed next, while another is not.

Creating a new commit is done via the `git-commit` program:

    $ git commit -m "Add license to README file"

This saves the state of the index (anything you have staged) as a new commit with the message `Add license to README file`.

When you have staged something you do not want to be part of the next commit, you can _unstage_ it. This will keep the change on disk, but it will not be included in your next commit. Somewhat cryptically, you can do this with the `git-reset` program as Git itself explained in the output from `git-status`:

    $ git reset HEAD README.md

### Browsing project history

Once you have made a couple of commits, you can use `git-log` to list them. Git can show you all kinds of information in all kinds of formats, but by default it lists detailed descriptions of every commit from the current version of your working tree back to the earliest commit it can find:

    TODO git log output

### Creating and merging branches

Git has first-class support for branches. Most branch-related operations are done using the `git-branch` program. You can list branches:

    $ git branch
    master
    docs

You can start a new branch at the currenty checked out commit:

    $ git branch new-feature

To actually switch to a branch and make new commits on it, you have to check it out (literally):

    $ git checkout new-feature

The pattern of creating a new branch and switching to it is common enough to warrant a shortcut:

    $ git checkout -b new-feature

Once you have made a couple of commits on another branch and you are ready to merge them back to the `master` branch, you can use the `git-merge` program:

    $ git checkout master
    $ git merge new-feature

The `new-feature` branch still exists, but its contents is now identical to the `master` branch.

### Resolving merge conflicts

When you work with branches, it is not rare to encounter merge conflicts. Usually, Git is rather smart about solving multiple changes in the same file. Sometimes, it is impossible for Git how to merge two changes. When that happens, Git will inform you that a merge failed and that it is up to you to solve the problem manually:

    $ git merge new-feature
    ... conflicts

What has happened, is that the merge is actually still in process. Your project directory now contains all the changes from both `master` and `new-feature` as far as Git could merge them, but the merge has not yet been recorded as a new commit. That final step is up to you. To inspect what files are in conflict, use `git-status`:

    $ git status

Note how all successfully merged files are staged, while conflicted files are not. Use your favourite editor to edit the conflicted files manually. Git has wrapped every conflict in the following wrapper lines:

    <<<<<<< master
    line in master branch
    =======
    line in feature branch
    >>>>>>> new-feature

Edit these lines so that file is correct again. To resolve the conflict, simply stage the file:

    $ git add README.md

Repeat the process for every conflicted file and then finish the commit:

    $ git commit

### Working with remote repositories

Git is a decentralized version control system, which means there is no one central repository that contains "the truth". Instead, every team member has its own full copy of the repository on its local machine. Usually, a team sets up a single canonical repository on a shared server, even though there is nothing about Git itself that says you should do it like that.

With every team member having its own repository and making commits locally, you will need a way to share new changes with each other. For this, Git supports remote repositories: a reference to another Git repository. Using `git fetch` and `git push` it allows you to copy changes to and from that repository.

When you have completed a feature branch, you can copy it and all its changes to a remote repository, such as a colleague's or a shared canonical repository:

    $ git remote add origin git://shared.server/repo.git
    $ git push origin new-feature

To copy any changes from the other repository to your local repository, you use `git-fetch`:

    $ git fetch origin

Usually, you will want to merge any changes from the other repository's `master` branch to your copy of the `master` branch. You can merge a remote branch into your own:

    $ git fetch origin
    $ git merge origin/master

This is a common enough pattern, to have its own subcommand: `git-pull`:

    $ git pull origin master

This will download any new changes from the `origin` remote repository, and merge any changes from its `master` branch into your `master` branch.

### Stashing changes

The final highlight in this brief overview of every day Git features is the stash: a convenient temporary location to store changes in without making them full-blown commits on feature branches. Consider the following scenario: you are working on a new feature. You are not yet done, but your boss tells you to immediately fix a critical bug. You need to get your working tree back to a pristine state, but your new feature is not yet done. Just stash the changes away for now:

    $ git stash

Your working tree is now exactly as it was in your latest commit. You can start work on something else, finish and commit that, and then return to your original new, unfinished feature:

    $ git stash pop

All changes are re-applied to your current working tree and you can continue where you left off. The stash is not limited to one set of changes; you can run `git stash` multiple times and it will remember every set of changes. You can see a full list of changes you have stashed away:

    $ git stash list

## Further reading

This concludes a very brief overview of the most common Git operations and programs. For more information, refer to the following excellent resources:

* The Git website ([http://git-scm.com](http://git-scm.com))
* Pro Git by Scott Chacon ([http://git-scm.com/book](http://git-scm.com/book))
