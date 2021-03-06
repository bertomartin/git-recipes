# 7 -- Working with the index

## Staging only tracked files

### Problem

You have numerous changes in your working tree. You want to stage them all and
commit them to the repo, but there are some untracked files you wish to exclude
-- perhaps to add them later in a separate commit.

### Solution

You can use the `--update` (or `-u`) option of the `git-add` program to only
stage changes in already tracked files:

    $ git status
    # On branch master
    #
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   README
    #   deleted:    doc/index.html
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #	.rspec
    no changes added to commit (use "git add" and/or "git commit -a")
    $ git add -u
    $ git status
    # On branch master
    #
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #	modified:   README
    #   deleted:    doc/index.html
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #	.rspec

Note that the `--update` option has the side effect that file deletions will
also be staged, while the plain old `git add` only stages changes to files.

### Discussion

The difference between a normal `git add <filepattern>` and `git add 
-u <filepattern>` is how Git matches `filepattern`. This is a basic glob pattern
that defaults to `.` or the current directory and everything below it. Every
file matched by this pattern will be staged.

But what files are matched against it? When using the regular `git add` Git
will match all the files in the current working tree to the pattern. When a
match is found, the file is copied to the index. When using `git add -u`, Git
instead matches all the files from the index to the pattern. Untracked files
are, by definition, not in the index yet, and will therefore be ignored by `git
add -u`. And because the index still contains files that were deleted in the
working tree, `git add -u` will actually find those and stage their deletion.

There is a third option: `git add -A`. You will not be surprised to find that
it matches `filepattern` against both the working tree and the index, ensuring
that the index will be identical to the working tree -- with all untracked
files added, all deleted files removed and all changes staged.

## Unstaging files

### Problem

You have used `git add` to add a file to the index, staging it to be commited
to the repository. This was a mistake and you wish to unstage it.

### Solution

Git tells you the solution when you run `git status`: use `git reset
HEAD <file>` to unstage it:

    $ git reset HEAD README.md

This command is used frequently enough (and typing it is awkwardly enough) to
warrant an alias. You could use the following:

    git config -g alias.unstage reset HEAD

This allows you to do:

    $ git unstage README.md

See another recipe for more information on defining aliases. See another recipe
for more information on using the `git-reset` program.

## Staging or unstaging parts of a file

### Problem

You have made several unrelated changes in a single file. You wish to stage
some for your next commit, but leave some others out.

### Solution

Git allows you to edit the patches that get applied to the index. This allows
you to determine exactly what lines do get staged, and what lines don't. If
this sounds scary to you, you are partially right. This is a powerful tool, and
powerful tools tend to require careful use. But remember, Git always makes it
hard for you to lose data; the worst that can happen is you get frustrated.

To partially stage a file, use the `--patch` or `-p` option for the `git-add`
program:

    $ git add -p app.rb

Git will now loop over all the files matching your `filepattern` -- in this case
only `app.rb` -- and show you a diff between your working tree and the index:

    diff --git a/app.rb b/app.rb
    index 4cbff34..6b68ac2 100644
    --- a/app.rb
    +++ b/app.rb
    @@ -1,6 +1,10 @@
     require 'sinatra'
     require 'store'
     
    -get '/:id' do
    +get '/url/:id' do
       redirect Store.fetch(params[:id].to_i)
     end
    +
    +post '/secret' do
    +  'You found the secret URL'
    +end
    Stage this hunk [y,n,q,a,d,/,s,e,?]?

You will recognise the output from the regular `git-diff` program. Git asks us
if we want to stage this hunk. We can answer `y` to stage the diff as it is
shown, or `n` to skip it. We can use `q` to quit the process and return to
the shell prompt.

Git will try to show us "hunks" of changes, but it might sometimes group too
many changes together in a single hunk, as is the case in our example. We can
use the `s` option to split the hunk into smaller hunks, giving us the
following:

    Stage this hunk [y,n,q,a,d,/,s,e,?]? s
    Split into 2 hunks.
    @@ -1,6 +1,6 @@
     require 'sinatra'
     require 'store'
     
    -get '/:id' do
    +get '/url/:id' do
       redirect Store.fetch(params[:id].to_i)
     end
    Stage this hunk [y,n,q,a,d,/,j,J,g,e,?]? 

Git now presents us with the same choices but for only the first change. Let's
answer `y` here, and Git will continue to the next hunk:

    Stage this hunk [y,n,q,a,d,/,j,J,g,e,?]? y
    @@ -5,2 +5,6 @@
      redirect Store.fetch(params[:id].to_i)
    end
    +
    +post '/secret' do
    +  'You found the secret URL'
    +end
    Stage this hunk [y,n,q,a,d,/,K,g,e,?]? 

Pick `n` in this case, and Git will return us to the shell prompt. When you run
`git status` now, you will see that `app.rb` is both staged and unstaged: it
contains changes to be comitted, and changes not to be comitted:

    $ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #   modified:   app.rb
    #
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #   modified:   app.rb
    #

Now suppose we had `app.rb` staged completely, and we wanted to unstage part of
it. We can use the same `--patch` option for the `git-reset` program (see another recipe
for more information about unstaging stages using `git-reset`):

    $ git add app.rb
    $ git reset -p HEAD app.rb
    diff --git a/app.rb b/app.rb
    index 4cbff34..6b68ac2 100644
    --- a/app.rb
    +++ b/app.rb
    @@ -1,6 +1,10 @@
     require 'sinatra'
     require 'store'
     
    -get '/:id' do
    +get '/url/:id' do
       redirect Store.fetch(params[:id].to_i)
     end
    +
    +post '/secret' do
    +  'You found the secret URL'
    +end
    Unstage this hunk [y,n,q,a,d,/,s,e,?]? 

Instead of asking us to _stage_ a hunk, Git now asks us whether to _unstage_ a
hunk. We can use the same options to split the hunk and accept and reject as we
see fit.

### Discussion

The list of options you are presented with when using `--patch` might be
overwhelming, but luckily Git offers some help with the `?` option:

    y - unstage this hunk
    n - do not unstage this hunk
    q - quit; do not unstage this hunk nor any of the remaining ones
    a - unstage this hunk and all later hunks in the file
    d - do not unstage this hunk nor any of the later hunks in the file
    g - select a hunk to go to
    / - search for a hunk matching the given regex
    j - leave this hunk undecided, see next undecided hunk
    J - leave this hunk undecided, see next hunk
    k - leave this hunk undecided, see previous undecided hunk
    K - leave this hunk undecided, see previous hunk
    s - split the current hunk into smaller hunks
    e - manually edit the current hunk
    ? - print help

We have seen the `y`, `n` and `s` options, which are the most commonly used.
It is also good to know you can stop the entire process with `q`. The most
interesting option is `e`, that allows you to manually edit a hunk. See the
next recipe for a more in-depth explanation of this option.

## Editing hunks to be staged or unstaged

### Problem

You want to stage only some changes in a file for the next commit. However, Git
does not automatically create the correct hunks for you when using `git add
--patch`. You want to manually edit exactly which lines will be staged, at
which won't.

### Solution

Apart from accepting or rejecting entire hunks, Git's patch mode also offers you
the option to _edit_ a hunk. Run `git add --patch <yourfile>` and cycle to the hunk
you want to edit. Git prompts you for an action:

    diff --git a/app.rb b/app.rb
    index 4cbff34..a70cdca 100644
    --- a/app.rb
    +++ b/app.rb
    @@ -1,6 +1,7 @@
     require 'sinatra'
     require 'store'
     
    -get '/:id' do
    +get '/id/:id' do
    +  # Redirect to URL
       redirect Store.fetch(params[:id].to_i)
     end
    Stage this hunk [y,n,q,a,d,/,e,?]? 

As you can see, the patch contains two changes directly next to each other. We
can no longer use `s` to split up this hunk, as Git would not know where to
split it.  We shall therefore use `e` to open this patch in our default editor
and manually edit it.  Git pre-loads the following document for us:

    # Manual hunk edit mode -- see bottom for a quick guide
    @@ -1,6 +1,7 @@
     require 'sinatra'
     require 'store'
       
    -get '/:id' do
    +get '/id/:id' do
    +  # Redirect to URL
       redirect Store.fetch(params[:id].to_i)
     end
    # ---
    # To remove '-' lines, make them ' ' lines (context).
    # To remove '+' lines, delete them.
    # Lines starting with # will be removed.
    #
    # If the patch applies cleanly, the edited hunk will immediately be
    # marked for staging. If it does not apply cleanly, you will be given
    # an opportunity to edit again. If all lines of the hunk are removed,
    # then the edit is aborted and the hunk is left unchanged.

There are three important parts: the indication that we are in manual hunk edit mode,
the diff we are editing, and a help section. We can now take several steps:

If we want to abort the edit, we can simply remove every line and quit our
editor while saving our changes. Git will bring us back to the patch process.

If we want to record the change on the fourth line, we edit the document so
that it looks like this:

    # Manual hunk edit mode -- see bottom for a quick guide
    @@ -1,6 +1,7 @@
     require 'sinatra'
     require 'store'
       
    -get '/:id' do
    +get '/id/:id' do
       redirect Store.fetch(params[:id].to_i)
     end

We have removed the line that read `+  # Redirect to URL`. Since this line is left
out of the patch, the change will be not be staged.

If we want to record the adding of the commit, we edit the document so that it
looks like this:

    # Manual hunk edit mode -- see bottom for a quick guide
    @@ -1,6 +1,7 @@
     require 'sinatra'
     require 'store'
       
     get '/:id' do
    +  # Redirect to URL
       redirect Store.fetch(params[:id].to_i)
     end

Note how the patch now only has a `+` line for the new comment. We have removed
the other `+` line, since that is a change we do not wish to stage, and we have
removed the dash from the `-` line, since we do not want that line to be
deleted.

When the necessary changes have been made, we can save our changes and quit our
editor. Git will apply the patch to the index and prompt your for the next step
in the patch process.

### Discussion

Git stages and unstages changes by applying patches to the index. You can edit
those patches if you want, allowing you to really hand-craft your commits.
There are however a couple of things to keep in mind:

* Note how every line in the example diffs start with either a `+`, `-` or a
  space. Lines starting with a space are "unchanged". Not starting a line with
  a space will result in a bad patch and failure for Git to apply the patch.  Take
  care that you editor does not remove all trailing whitespace when editing Git
  patches!
* You could add and remove more lines in the patch than you did in your
  working tree. This may theoretically work, but is not a good idea,
  since you are adding content to the index and then the repository
  that was never tested in any way.
* Note the `@@ -1,6 +1,7 @@` line at the top of the diff. These contain the
  input and output ranges in the file, expressed in line numbers. When your
  diff contains changes on the edges of the context Git shows you -- which
  would happen if the very first line in the file has changed -- you might have
  to edit these ranges in order to get the patch to apply. This is, admittedly,
  an edge case.

## Continue work on your last commit

### Problem

You have committed changes to your local repository, but you just realised your
work was not yet complete. You could use the `--amend` option of `git-commit`
to add new changes to your last commit, but you would like a little more
control over the changes in the final commit.

### Solution

Using `git-reset` you can undo your last commit, and optionally leave the
changes in the index and your working tree. You have probably used `git reset
--hard` before to completely undo a commit and restore your working tree to the
state it was in one or more commits back in time. But you could also leave your
cumulative changes staged, ready to be committed again, using `git reset
--soft` or only leave them in your working tree so you can stage them again
using `git reset --mixed` (`--mixed` is the default option).

Take a look at the following example:

    $ echo "Hello, world" > README
    # On branch master
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    # modified:   README
    #
    $ git add README
    $ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    # modified:   README
    #
    $ git commit -m "Example commit"

We have changed a file and committed that change to the repository. If we want
to continue working on this commit, we could pretend as if we had never staged
or committed this change at all:

    $ git reset HEAD^
    $ git status
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    # modified:   README
    #

We are now back in the state before we added the changes to `README` to the
index. This is great if you have a lot of changes, and you want to create a new
commit containing only a few of them. On the other hand, if you have a lot
changes and you want to omit only a few of them, you could use the `--soft`
option to leave the index intact:

    $ git reset --soft HEAD^
    $ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    # modified:   README
    #

The change seems trivial with a small set of changes in a single commit, but
you could just as easily reset to ten or more commits back. This would
accumulate all changes introduced in those commits as changes in your working
tree and index.

### Discussion

The `git-reset` program can be used to manipulate the index and working tree of
your repository. Its options `--soft`, `--mixed` and `--hard` are hardly
intention revealing, but once you have tried them all, you will quickly grasp
what they do.

Consider the usage:

    $ git reset --[soft|mixed|hard] <target>

`<target>` is a commit or reference, like `HEAD^2` or `c7eb45`, defaulting to
`HEAD`.

* `--soft` only moves `HEAD` to the target;
* `--mixed` does the same as `--soft`, but also updates the index to reflect
  the target;
* `--hard` does the same as `--mixed`, but also updates the working tree to
  reflect the target.

It should no longer be a mystery why Git advises you to unstage a file using
`git reset HEAD <path>`. Considering that `--mixed` is `git-reset`'s default
operation mode, what it essentially tells you to do is:

    $ git reset --mixed HEAD <path>

As per the description above, Git will make the index look like the contents of
HEAD (or: the latest commit) -- essentially as if nothing had been staged since
the last commit. Since it is operating on a single file, the usual action
associated with `--soft` makes no sense, so no references are updated and no
commits are abandoned. And since `--hard` is not in effect, the working tree is
left alone. So, your changes are kept on disk, but they will be removed from
the index: they are unstaged.

Note how using `git-reset` can function as an alternative to `git-stash`: you
can quickly commit some stuff, move elsewhere, do some work, and then return to
your temporary commit, reset it and continue working. The main difference is,
of course, that your stash is purely local, while a temporary commit isn't --
it is just a commit in the repository, ready to be shared with colleagues. 

## Creating a branch from stashed changes

### Problem

You were working on a new feature, when you had to stash your changes and
quickly fix a bug. You now want to continue working on your feature using your
stashed changes, but enough has changed with your bug fix that applying your
stash now causes merge conflicts. You want to continue working from the point
where you saved the stash.

### Solution

The `git-stash` program comes with the `branch` subcommand that lets you create
a new branch and apply the stash on that branch in one go. What's neat is that
the branch will be created from the commit that the stash was saved from, so
you know the stash will apply cleanly:

    $ echo "Hello, world" >> README
    $ git stash
    Saved working directory and index state WIP on master: 421002c Initial commit
    HEAD is now at 421002c Initial commit
    # make some commits here
    $ git stash brach improve-readme

This will create a new branch called `improve-readme` starting from `421002c`,
check that out, apply the latest stash and drop it.

## Summarising working tree changes

### Problem

In your daily workflow, the regular output from `git-status` might get too verbose for your taste. You want something shorter.

### Solution

The `git-status` program comes with the handy `--short` (or `-s`) option to
display only the bare minimum of information. Compare:

    $ git status
    # On branch master
    # Changes to be committed:
    #   (use "git reset HEAD <file>..." to unstage)
    #
    #	modified:   README
    #
    # Changes not staged for commit:
    #   (use "git add <file>..." to update what will be committed)
    #   (use "git checkout -- <file>..." to discard changes in working directory)
    #
    #	modified:   LICENSE
    #
    # Untracked files:
    #   (use "git add <file>..." to include in what will be committed)
    #
    #	.DS_Store

With:

    $ git status -s
    M  02-a-quick-introduction-to-git.md
     M 07-working-with-the-index.md
    ?? assets/outline.md
    ?? styles.css

Git uses a single line per file and uses the first two columns of each line to
indicate its status: the first to indicate status in the index, the second in
the working tree.

One thing you might miss from this output is the information on the current branch. You can bring that back using the `--branch` (or `-b`) option, so your short status output will still display the branch name and tracking information:

    $ git status -sb
    ## master
    M  02-a-quick-introduction-to-git.md
     M 07-working-with-the-index.md
    ?? assets/outline.md
    ?? styles.css

You might even consider setting up a shell alias:

    alias gs='git status -sb'

