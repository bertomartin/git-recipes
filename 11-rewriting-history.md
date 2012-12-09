# 11 -- Rewriting history

## Amending the last commit

### Problem

You have just committed some changes to your local repository, only to find you
made a mistake: using `git commit -a` you included all changed files, but Git
did not pick up some new files you added. Those files should really have been
part of that commit, so you want to change it.

### Solution

`git-commit` accepts the `--amend` option to rewrite the last commit. Using
`--amend`, you make a commit as usual, but it will be merged with the last
commit you made.

First, let's prove our latest commit only changed a single file:

    $ git show --stat --oneline
    f9a5cff Apply markdown to README file
     README    | 1 -
     README.md | 1 +
     2 files changed, 1 insertion(+), 1 deletion(-)

Now, let's change that commit:

    $ echo "test" > test
    $ git add test
    $ git commit --amend

Git will fire our `$EDITOR` as usual, but instead of entering a new commit it
pre-fills it with the last commit message we made. We can now change that
message (or not) and finish the commit as usual.

To prove we have changed the last commit, look at `HEAD`:

    $ git show --stat --oneline
    fca32e7 Apply markdown to README file
     README    | 1 -
     README.md | 1 +
     test      | 1 +
     3 files changed, 2 insertions(+), 1 deletion(-)

We have succesfully changed the last commit we made! We have added a single
file, but we could have changed the commit message, removed files, changed
files --- basically anything we want.

### Discussion

Ammending the last commit is a great way quickly fix a mistake, even if only a
typo in the commit message. Do note that we have _changed a commit_ (see for
yourself: the hash of our commit object changed), so all the usual caveats
about rewriting history apply (see earlier section).

How does rewriting work? The documentation for `git-commit` explains it
quite nicely: it is basically a convenience option around `git-reset`: when
using `--amend`, Git first resets the repository to the state of the last commit,
and only then continues with creating the new commit.

You could simulate the `--amend` option yourself:

    $ git reset --soft HEAD^
    $ git commit -c ORIG_HEAD

Using the `-c` option tells Git you want to re-use the commit message of another
commit object, like `ORIG_HEAD`, which is set by the `git-reset` command. For more
information on the `git-reset` command, see section x.y.z.

## Changing commit messages

### Problem

You have developed a new feature in the project in a local feature branch. You
are ready to merge your work into the `master` branch, so you take one last
look at the output from `git-log` to make sure everything is in order. You spot
a few uninspiring commit messages you want to reword to clarify their contents
and their intent.

### Solution

When using interactive rebasing, you can selectively _reword_ commit messages.
The principle is the same as when using `git-commit` with the `--amend` option,
but it works on any commit from your history, not just the latest commit.

We have made three commits on the new feature branch:

    $ git log --oneline master..HEAD
    7775089 layout
    fc86f84 Added show tempalte
    8575dcd Added route to show a link

Let's improve the commit messages for `fc86f84` and `7775089` by rebasing the entire
branch:

    $ git rebase -i master

This will prompt our `$EDITOR` with the following contents:

    pick 8575dcd Added route to show a link
    pick fc86f84 Added show tempalte
    pick 7775089 layout

    # Rebase f9a5cff..7775089 onto f9a5cff
    #
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #
    # Note that empty commits are commented out

There's a line for each commit in our branch. We can edit this file and then
quit our editor to tell Git what we want to do. Taking a cue from the comments
below, let's change `pick` into `reword` on the lines for the commits we want
to edit. Our (abbreviated) content now looks like this:

    pick 8575dcd Added route to show a link
    reword fc86f84 Added show tempalte
    reword 7775089 layout

When we save this file and exit our editor, Git will start replaying all the
commits listed in the file. It will open our `$EDITOR` twice --- once for
commit `fc86f84` and once for `7775089` --- with the commit messages preloaded.
We can now edit these messages, save the temporary file and exit the editor
to redo that commit with the new commit message.

Git informs us that the rebase operation was successful:

    $ git rebase -i master
    [detached HEAD e935b07] Added show template
    1 file changed, 3 insertions(+)
    [detached HEAD dbfb524] Added a sensible HTML layout
    1 file changed, 10 insertions(+)
    Successfully rebased and updated refs/heads/show-link.

We can see our new commit message in all their glory in the logs:

    $ git log --oneline master..HEAD
    dbfb524 Added a sensible HTML layout
    e935b07 Added show template
    8575dcd Added route to show a link

### Discussion

Taking some time to reflect upon the commits you are about to commit to the
shared repository is a good practice to get into. When you are in the zone, go
ahead and commit away. Write your commit messages quickly and get on with the
real work. But once you are done, take a minute to fix spelling mistakes and
improve descriptions.

Note that even making minor adjustments to a commit message, triggers the
creation of an entirely new commit object (remember, the commit SHA hash is
based on, among others, the commit message). Therefore, all te usual caveats
about history rewriting apply.

## Reordering commits using interactive rebasing

### Problem

As you are reviewing the changes in your feature branch before you merge it to
`master` and share it with your team members, you notice you were kind of all
over the place as you implemented the feature. It would make sense to group a
couple of commits together, so your changes are easier to code review.

### Solution

Interactive rebasing allows you to re-order commits. Rebase your feature branch
on the `master` branch:

    $ git rebase -i master

This will trigger your `$EDITOR` with a list of commits:

    pick 8575dcd Added route to show a link
    pick fc86f84 Added show tempalte
    pick 7775089 layout

    # Rebase f9a5cff..7775089 onto f9a5cff
    #
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #
    # Note that empty commits are commented out

Git already points out in the comments in this file, that you can move these
lines around to your liking. Let's swap two lines:

    pick fc86f84 Added show tempalte
    pick 8575dcd Added route to show a link
    pick 7775089 layout

Saving this file and exiting your editor, will cause Git to re-do all these
commits in the order you specified.

### Discussion

Reordering commits is useful to make your changes easier to understand, but be
aware of its implications:

1. Swapping two commits changed both of them: they get new parents and thus new
   hashes. Therefore, any subsquent commit is also rewritten.
2. By re-ordering commits, you may be trying to apply patches to pieces of
   files that were not there before. Prepare to deal with merge conflicts as
   Git tries to apply these changes for you.

In practice, re-ordering commits is not worth the trouble.

## Rewriting history by squashing commits

### Problem

Over the course of building a new feature, you have made several commits
cleaning up the usage of whitespace in your project. You have ended up with
several commits titled "Clean up whitespace", which is kind of silly. You want
to merge them into a single clean-up commit.

### Solution

You can squash multiple commits together using interactive rebasing. This will
cause Git to apply to patches to the same working tree, and only then record a
new commit object. Although unlikely, this might trigger merge conclicts.

First, use interactive rebasing to rewrite your feature branch:

    $ git rebase -i master

This will trigger your `$EDITOR` with a list of commits:

    pick 8575dcd Added route to show a link
    pick fc86f84 Whitespace
    pick 7775089 Clean up whitespace

    # Rebase f9a5cff..7775089 onto f9a5cff
    #
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #
    # Note that empty commits are commented out

By changing `pick` into `squash`, you can tell Git to merge that commit into
the one that came before it. Therefore, to merge our two whitespace commits, we
only need to change the second:

    pick 8575dcd Added route to show a link
    pick fc86f84 Whitespace
    squash 7775089 Clean up whitespace

When we save and quit, Git will prompt us to enter a commit message for our new
combined commit:

    # This is a combination of 2 commits.
    # The first commit's message is:
    
    Added show template
    
    # This is the 2nd commit message:
    
    Added a sensible HTML layout

Helpfully, Git is telling us the original commit messages for each commit we
are merging (so this might easily contain a handful of commit message instead
of just two). It is up to you to come up with a new message that accurately
describes both of your commits. When you're done, you can save and quit your
editor as usual to record the commit.

Notice how the comments in the rebase overview also mention the `fixup` option.
This is similar to using `squash`, but it does not prompt you to enter a new
commit message -- it simply uses the first and discards the second.

## Splitting a single commit into multiple commits

### Problem

While you were working on a new feature, you were interrupted by a colleague.
Rather than stashing your changes, you quickly recorded your uncommitted
changes in your working tree using a new commit titled "WIP" (short for "Work
In Progress"). This commit actually contains several changes you now want to
split into multiple commit objects, to make your code easier to review.

### Solution

Interactive rebasing allows you to edit a commit object. When re-doing all the
commits you have selected, Git will stop after applying the changes of the
commit you want to edit to the index, giving you the chance to make changes.
When you are done, you can tell Git to continue applying any remaining commits
from the rebase range.

First, rebase your feature branch:

    $ git rebase -i master

You will be prompted with the familiar list of commits in your editor:

    pick 8575dcd Added route to show a link
    pick fc86f84 Added show tempalte
    pick 7775089 layout

    # Rebase f9a5cff..7775089 onto f9a5cff
    #
    # Commands:
    #  p, pick = use commit
    #  r, reword = use commit, but edit the commit message
    #  e, edit = use commit, but stop for amending
    #  s, squash = use commit, but meld into previous commit
    #  f, fixup = like "squash", but discard this commit's log message
    #  x, exec = run command (the rest of the line) using shell
    #
    # These lines can be re-ordered; they are executed from top to bottom.
    #
    # If you remove a line here THAT COMMIT WILL BE LOST.
    #
    # However, if you remove everything, the rebase will be aborted.
    #
    # Note that empty commits are commented out

Mark a commit for editing by changing its corresponding line to start with
`edit` rather than `pick`. Then save and quit, so Git will proceed with
applying the commits in order.

When it gets to the commit to be edited, Git applies it but stops, informing
you that you can go ahead, do your business, and how to tell Git to continue:

    Stopped at dbfb524... Added a sensible HTML layout
    You can amend the commit now, with

      git commit --amend

    Once you are satisfied with your changes, run

      git rebase --continue

We can use `git commit --amend` to change the last commit, or we can use `git
reset HEAD^` to undo it altogether and leave all its changes unstaged.

We can now use `git-add` and `git-commit` as we see fit to craft several commits
out of the changes of just this one commit. When we're done, we simply tell Git
to apply any remainin commits from the rebase:

    $ git rebase --continue

...and we're done!

## Automatically squashing commits

### Problem

Going back and squashing commits is a tedious job. Sometimes, you know you want
to squash the commit you are about to make, into another commit. You want to
speed up the rebasing process and mark it for squashing right now.

### Solution

`git-rebase` gained the `--autosquash` option with its 1.7 release. This is a
simple tool that reads the commit messages you are about the rebase, and
automatically applies a `squash` or `fixup` strategy to them.

How does Git know what strategy to use? It takes its cue from the first word of
the commit message. Messages starting with `squash!` are squashed, messages
starting with `fixup!` are used to fix their previous commit. What commit is it
merged with? Git merges the commit with the other commit in the rebase range
that has the same commit message as the rest of the message of the commit your
are marking to be squashed. Simple, no?

Let's look at an example. We've got the following feature branch:

    $ git log --oneline master..HEAD
    dbfb524 Added a sensible HTML layout
    e935b07 Added show template
    8575dcd Added route to show a link

Let's make a change to the show template we've created in `e935b07`:

    # do some work on app.rb first...
    $ git add app.rb
    $ git commit -m "fixup! Added show template"

Our history now looks like this:

    f354e0f fixup! Added show template
    dbfb524 Added a sensible HTML layout
    e935b07 Added show template
    8575dcd Added route to show a link

Nothing has happened yet. We have only left a special instruction for Git,
which it will pick up when we run `git rebase` with the `--autosquash` _and_
`-i` options. When we do, Git prompts us in our `$EDITOR` with the usual list
of commits we can rebase, but note how it has automatically changed the
instructions for our latest commit:

    pick 8575dcd Added route to show a link
    pick e935b07 Added show template
    fixup f354e0f fixup! Added show template
    pick dbfb524 Added a sensible HTML layout

    # ...comments omitted for brevity...

Git has changed the list so that our `fixup!` commit will be applied to
`e935b07` even if we do nothing else but save and quit our editor now.

### Discussion

Note that adding special instructions to commit messages does nothing in
itself, other that indicate to the reader that you intended to fix a previous
commit. It is still human-readable, even if you never actually rebase the
commits.

It is kind of odd that Git forces you to identify the target commit by its
commit message. Don't even think about re-entering the commit message; simply
copy the message from Git's log output or use a shell command like `git log -1
--format='%s' HEAD~3` to get the commit message of the third-last commit.
