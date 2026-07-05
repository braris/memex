
# [5 git commands that feel like magic](https://www.howtogeek.com/git-commands-that-feel-like-magic/)

The git program supports so many sub-commands that it’s almost impossible to keep on top of them all. Most of them are quite low-level and carry out simple tasks like committing code changes or switching branch. But some of the most obscure offering surprising functionality that does way beyond the norm.

## git blame: find out who changed what

Personally, I think git blame could use a more positive name: git credit, maybe? Anyway, however you choose to use it, this command will tell you exactly who’s responsible for each line of code in a file:

![A terminal showing the result of git blame which includes each line of a file with commit id, author, and time of change alongside.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/11/git-blame.png?q=70&fit=crop&w=825&dpr=1)

What you’ll see by default is every line in your file, prefixed with details of the last commit that changed it. Those details include the commit ID, the date and time of that commit, and the author who made that change.

If you’re working on a larger project with many contributors, this can be a valuable way of following up on specific code changes. You can use it to find out why somebody made them, or consult on a possible update.

The [git-who command shows stats for authors](https://www.howtogeek.com/discover-who-works-on-what-with-this-helpful-git-tool/) across a whole repository.

Blame is a feature that GitHub supports too: navigate to any individual file, and you’ll see a “Blame” tab next to the default, “Code:”

![The GitHub interface showing a file in "blame" mode, with age, commit, and author avatar alongside each line of code.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/11/git-blame-github.png?q=70&fit=crop&w=825&dpr=1)

Note that GitHub helpfully color-codes the commits by age so you can quickly scan the list and find recent changes.

## git archive: package from any point in history

Working with a Git project typically involves cloning repositories, checking out branches, and committing changes. But, sometimes, you want to work with just the project files: when you’re running a demo or carrying out casual testing, for example.

The git archive command gives you a clean collection of project files, without anything Git-related, at any commit point in your repository:

```bash
git archive HEAD
```

This command outputs the contents of all files in the current branch, in tar format. You’ll probably want to save that output, and there are two ways to do so. You can use shell file redirection to send the output to a file:

```bash
git archive HEAD > project.tar
```

Alternatively, you can use the --output option to send output to a named file:

```bash
git archive --output=project.tar HEAD
```

The main advantage of this approach is that Git will intelligently select the format based on your filename. By default, it will output in tar format, but it supports others, like.tar.gz and zip:

```bash
git archive --output=project.zip 428d235
```

Remember to specify the commit you want using a pointer like HEAD, [a tag ID](https://www.howtogeek.com/devops/what-are-release-tags-in-git-and-how-do-you-use-them/), or a commit ID. You can even pass a subdirectory to archive just part of your project:

```bash
git archive --output=project.tar.gz v1.1 docs
```

In some senses, the git archive command is anti-magic: it removes all the magic Git data from your repository, converting it to a plain set of files. But creating an instant archive of your project from any point in its entire history is a pretty special trick in my book.

## git stash: save changes without committing them

Time and time again, I’ve run into this situation: you’re busy [working on a branch](https://www.howtogeek.com/devops/how-do-git-branches-work/), then something else needs attention. Maybe it’s a feature that needs tweaking or a branch that requires a quick bug fix. You’ve saved your files, but switching branches now will remove those changes.

When I didn’t know any better, I would typically check in my changes, hoping to clean up the commit history later (and probably failing). But this requires a lot of admin and leaves the repository in a messy state, even if just for a brief time. But now I know about git stash:

![Git responds to the stash command with a message indicating that it has saved the working directory and index state on the current branch.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/11/git-stash.png?q=70&fit=crop&w=825&dpr=1)

Once you’ve run git stash, you’ll be left with a clean working tree, so you can safely switch branches and work on something else. When you’re ready to return, you can restore your files:

```bash
git stash pop
```

As if by magic, your files are back, with all the changes you made ready to be checked in. The name of this command reflects the stack-based nature of the stash. If you need to stash multiple times, your changes will be saved on a stack, so the most recent stash will be returned when you ask for one.

## git grep: search your codebase across branches

The git grep sub-command searches through files and prints lines that match a pattern. That might not sound too impressive at first; after all, isn’t that [exactly what grep does](https://www.howtogeek.com/496056/how-to-use-the-grep-command-on-linux/)? The magical bit is how well Git integrates this alternative grep.

By default, git grep searches in all tracked files in your working tree. This is the key difference between the standard grep command and the Git version: grep searches files, while git grep searches the contents of files that have been committed to your repository.

By default, git grep searches the current version of the repository in your working tree. But you can also search the contents of other branches or the repository at a specific commit:

```bash
git grep TODO 4415c19
```

Without git grep, you’d have to check out the specific revision before grepping and account for ignored files, untracked files, and recursion. You can even search through multiple revisions, including the entire history of your repo, if you’re feeling brave:

```bash
git grep strcat $(git rev-list --all)
```

## git worktree: work on the same repository more than once

If you thought git stash was good, here’s something even better: git worktree. While stashing is fine for short-term work, it can get awkward if you’re working on files over a longer period or from many different branches at once.

The worktree feature provides a more permanent way to work on multiple branches simultaneously. It does so by giving you an extra working folder: i.e., another collection of files from the repository.

As with most Git features, you can adapt worktrees to fit your process, but here’s a simple example of how to get started:

```bash
git worktree add <path>
```

In an existing repository, use the worktree sub-command to create a new directory attached to a specific branch. The last part of your path will be used for the branch name, which can be an existing branch:

![Adding a new git worktree with a message confirming its creation.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/11/git-worktree-add.png?q=70&fit=crop&w=825&dpr=1)

Any branches checked out in linked worktrees will be highlighted in cyan and marked with a plus sign:

![The git branch command showing a worktree branch with a plus symbol, colored in cyan.](https://static0.howtogeekimages.com/wordpress/wp-content/uploads/2025/11/git-branch-worktrees.png?q=70&fit=crop&w=825&dpr=1)

You can then work in each directory, on specific branches, without having to keep switching between them, which makes worktrees perfect for multitaskers.