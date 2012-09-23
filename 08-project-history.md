# 8 -- Browsing the project history

## Finding who added or changed a line in a file

### Problem

You have discovered a bug in your project and pinpointed it to a method in a
specific source file. You want to know who added the faulty line and in what
commit.

### Solution

The `git-blame` program can annotate every line in a file with information on when that line was last touched. This tells you the commit hash, author and date. For brevity's sake, this example uses `-s` to omit commit author and date:

    $ git blame -s app.rb
    ^09bca75  1) require 'sinatra'
    ^09bca75  2) require 'store'
    ^09bca75  3) 
    ^09bca75  4) get '/:id' do
    ^09bca75  5)   redirect Store.fetch(params[:id].to_i)
    ^09bca75  6) end
    0d965a4c  7) 
    0d965a4c  8) get '/help' do
    0d965a4c  9)   erb :help
    0d965a4c 10) end

Now you know the commit hash, you can inspect the problem further. 

### Discussion

Blaming a file is very useful when you are trying to trace your (or your
team's) steps throughout the project. You might run into two problems though.

First, dumping a long file without syntax highlighting into a terminal window
is not an optimal experience. Viewers, pagers and integrated tools might help,
but what you really want is to limit your blaming to a specific range of lines.
You can do this using the `-L` option to `git-blame`:

    $ git blame -L 4,6 app.rb

This will output only the `get '/:id'` route from our aplication. Since a piece
of code might wander around in a file over time, line numbers do not suffice.
Instead of a number, you can also enter a regular expression to determine where
the blaming should start and end:

    $ git blame -L "/^get /",+3 app.rb

Note how we indicate the start of the range with a regular expression, and the
end of the range with a relative line number -- `+3` stands for three lines
below the start of the range. We could, of course, also have used a regular
expression to define the end of the range.

Second, `git-blame` will only tell you about lines added or changed, because it
annotates every line of the file in its current state. To find when a line was
removed for your file, we need an entirely different approach.

## Finding who removed a line from a file

### Problem

Someday you might wonder why a particular validation rule was removed from your
model, or why a guard clause was removed from an important method. You have
tried using `git-blame`, but it can only tell you about lines added or changed
-- not lines removed.

### Solution

To search deletions in the project, you will have to inspect the diffs for each
commit. This might sound laborious, but luckily Git has a tool to help us. Slightly
surprisingly, it's an option to `git-log`:

    $ git log -S"get '/:id'" -- app.rb
    commit 39dc77d4791c520b856b5c52a29ff18bd509f718
    Author: Arjan van der Gaag <arjan@kabisa.nl>
    Date:   Sun Sep 23 14:29:46 2012 +0200

        Remove get route

    commit 09bca756d335370122589e443ec7056f5de73115
    Author: Arjan van der Gaag <arjan@kabisa.nl>
    Date:   Fri Dec 16 23:11:46 2011 +0100

        Initial commit

The `-S` option looks for a string that was added or removed. Alternatively,
the `-G` option takes a regular expression as argument. To have the log output
show the entire diff of the commit (so you can actually see the match) add the
`-p` option to `git-log`.

### Discussion

The `-S` and `-G` options are called the pickaxe. Using the pickaxe does more
than just searching for a string in the diff. A commit is considered to match
the search query, when the diff introduces a change in the number of string
matches. For example, removing a line reduces the number of matches from one to
zero. But adding a line, changing the count from zero to one, also counts as a
match.

Note that this is different from the search query appearing in the diff.
Imagine in the example we had indented the line containing `get '/:id'`. The
commit with that change would surely include that line in its diff, but the
pickaxe would not match it. It only matches when the number of occurences
changes.

## Searching through commit messages

### Problem

Last week you know a fix was committed for a bug from your team's issue
tracker. Since your team always includes references to the issue tracker in its
commit messages, you know you should be able to find it by its issue number --
but `git log | grep 1234` is not quite giving you the expected results.

### Solution

To search through commit messages, Git offers the `--grep` option for the
`git-log` program. It acts as a filter for the commits to show, meaning it will
show you the commits whose commit messages match the query, rather than just
show you the lines in the entire output matching your query, as piping it into
`grep` gives you.

    $ git log --grep Remove
    commit 39dc77d4791c520b856b5c52a29ff18bd509f718
    Author: Arjan van der Gaag <arjan@kabisa.nl>
    Date:   Sun Sep 23 14:29:46 2012 +0200

        Remove get route

### Discussion

The query used with `--grep` is a basic regular expression. You could choose to either match it as a simple string using `-F`, an extended regular expression using `-E` and have it ignore case using `-i`. Furthermore, you can specify multiple search queries by adding `--grep` multiple times. Git will match any commits matching either of the queries:

    $ git log --grep Remove --grep help
    commit 39dc77d4791c520b856b5c52a29ff18bd509f718
    Author: Arjan van der Gaag <arjan@kabisa.nl>
    Date:   Sun Sep 23 14:29:46 2012 +0200

        Remove get route

    commit 0d965a4ca4269c1f6f3389f3342c16ffe0b1d9e8
    Author: Arjan van der Gaag <arjan@kabisa.nl>
    Date:   Sun Sep 23 14:01:47 2012 +0200

        Added route to help page

To change the default "or" behaviour, use `--all-match` to have Git show only
commits matching _all_ the queries.
