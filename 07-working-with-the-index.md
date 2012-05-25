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

The difference between a normal `git add <filepattern>` and `git add -u
<filepattern>` is how Git matches `filepattern`. This is a basic glob pattern
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

Git tells you the solution when you run `git status`: use `git reset HEAD
<file>` to unstage it:

    $ git reset HEAD README.md

This command is used frequently enough (and typing it is awkwardly enough) to
warrant an alias. You could use the following:

    git config -g alias.unstage reset HEAD

This allows you to do:

    $ git unstage README.md

See another recipe for more information on defining aliases. See another recipe
for more information on using the `git-reset` program.

## Staging or unstaging parts of a file

## Problem

You have made several unrelated changes in a single file. You wish to stage
some for your next commit, but leave some others out.

## Solution

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
it. We can use the same `--path` option for the `git-reset` program (see another recipe
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

