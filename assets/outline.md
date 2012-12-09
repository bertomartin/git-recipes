* Introduction
    * Why another book on Git
    * Context of this book: experience from colleagues
    - A typical Git workflow
- Git object model
    - Example application: waterbug, a simple URL shortener
    - Peeking inside the Git object database
    - Adding another commit and understanding Git history
* Branching and merging
    - What makes branching cheap?
    - Of branches and references
    - The abandoned commit
    - Merging branches
    - Dealing with merge conflicts
    - Working with remote branches
* Git workflow
    - Using colors in Git output
    - Using a GUI
    - Using external diff programs
    - Creating your own git aliases
    ? Editor integration (textmate, sublimetext, fugitive)
    - Generating a list of changes
    M Showing a shorter status list
    - Finding help for Git scenarios and commands
* Working with files
    - Ignoring files across all projects
    - Ignoring a file already tracked by Git
    - Ignoring all files in a directory but one
    - Quickly show a file from another branch
    - Remove a file entirely from the repo
    - Untrack a file but keep it around
    - Searching files across the repository
    - Removing untracked files from the working tree
* Working with the Git index
    - Staging only tracked files
    - Unstaging files
    - Staging or unstaging partial files
    - Editing a hunk
    - Continue working on the last commit
    - Create a new branch from stashed changes
    - Summarizing working copy changes
    ? Reviewing changes in your working tree
* Browsing the project history
    - Find out who made a change in a file
    - Find out who removed a line
    - Searching commit messages
    - Listing the history of a file
    - Listing project committers
    * Reviewing recent actions on your local repository
* Dealing with branches
    * Find out changes that are in one branch but not another
    * Seeing all branches in a graph
    * Find out which branches a commit is in
    * Moving a single commit from one branch to another
    * Moving a range of commits from one branch to another
    * Squashing entire feature branches
* Merging and conflicts
    * Undo a merge
    * Undo a merge commit
    * Creating an explicit merge commit
    * Not creating needless merge commits
    * Re-using recorded merge conflict resolutions
    * using an external diff tool
* Working with remote repositories and teams
    * Setting up a branch to track a remote branch
    * Listing all remote repositories
    * Using multiple remotes
    * Pushing a local branch to a remote repository
    * Deleting a branch from the remote repository
    * Pushing one local branch to another remote branch
    * Export a repository to a compressed archive
    * Push changes to a locale file instead of a remote repository
    * Using rebase instead of merge with pull
* Rebasing
    * The dangers of rewriting history
    - Amending the last commit
    - Changing commit messages
    - Reordering commits using interactive rebasing
    - Rewriting history by squashing commits
    - Splitting a single commit into multiple commits
    - Automatically squashing commits
* Using Git hooks
    * Using commit message templates
    * Notifying others of new commits
    * Reject commits with trailing whitespace
