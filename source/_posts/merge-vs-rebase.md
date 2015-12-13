title: Merge vs Rebase
tags:
  - git
date: 2015-11-27 23:55:16
---

# Preface
Nowadays lots of teams use git as the VCS, one of most power feature of git is branching, so it's not uncommon that we need to frequently incorporate feasures into master or vise versa. As you know, there are two approches `merge` and `rebase`.
Then you may be wondering shall we use `git merge` or `git rebase`? The debates between these two is a bit controvertial and also there is holy-wars likes `vim vs emacs` or `Tabs vs Spaces` between them, here I have no interest to join the debate, which means I am not talking which one is better, but in a certain scenario whichone is more suitable. 
<!-- more -->

# Concept

## git merge
Merge brings the commits in one branch which are not in the other(current) branch into the current branch with a __merge__ commit.
```
A -> B -> C (master)
      \
       D -> E (feature)
```
Base upon the commit trees above, `git merge master` on feature branch leads to:
```
A -> B  ->   C    (master)
      \        \
       D -> E -> F(feature)
```
The commit `F` is a merge commit with two parents (Note the `Merge:` line below) something likes
``` git
commit 6ff74286c4573adf2e330d59ee1aaa8841f756a9
Merge: 1df0415 70e1786
Author: xavierzhou <xiayezhou#googlemail.com>
Date:   Sun Nov 22 23:25:37 2015 +0800

    Merge branch 'feature'
```

## git rebase
Rebase also brings the commits in one branch which are not in the other(upstream) branch into the upstream branch with a __transplant__ way.
```
A -> B -> C (master)
      \
       D -> E (feature)
```
Base upon the commit trees above, `git rebase master` at feature leads to:
```
A -> B -> C (master)
           \      
            D -> E (feature)
```
Here you can find the hisotry of feature branch is changed.(from `A->B->D->E` to `A->B->C->D->E`), keep this in mind, it's very important!

# Usage Scenario

## Working on master and wanna bring other branch back
* Best practise:
Always use merge please! i.e `git checkout master && git merge feature`.
* Explanation:
You still remember the `rebase` will change the history of branch? Altering the master branch history will makes other collabrator confuse and stuck.

## Sitting in  a feature branch and wanna take in the changeset happened in master
* Best practise:
Both `merge` and `rebase` are appropriate. i.e `git merge master` or `git rebase master`.
* Explanation:
If you prefer linear history and clean log for this feature, you should use `rebase`, if you prefer to keep the historical context then use `merge`.
N.B.
If you have pushed the branch to remote or as a Pull Request, make sure nobody will fetch your branch if you used `rebase` since it changes the history.

## Woking on a branch which hasn't been published(push) and wanna clean up
* Best practise:
`git rebase -i` is what you need.
* Explanation:
It's not rare that you finished some works with a commit, and realized `Damn, forgot to add some file!` or `Sh*t, another typo here` things. Then you makes more commits with
those fixs and your `git log` looks rather ugly as follows, 
```git
121ca50 Fix another typo
b253d57 Fix typo
60bfba2 Add forgot config files
ae6058c Implement feature
```
You can use `git rebase -i HEAD~4` to clean up history, which brings the default editor for git, and you now can `reword`, `squash`, `fixup` or even just remove some commits.


## A pull request was reviewed or approved and about to be merged back to master.
* Best practise:
It's a good timing to use `rebase` to clean up if you donn't want to keep the review/feedback changes in the pull request.
* Explanation:
Althought in this case the pull request is considered as a published one, since it will immediately be merged back and removed,
no one will fetch and care it soon, it's safe to be `rebase`ed.

# Golden Rule for rebase
Remember `rebase` changes the history of branch, **NEVER** do it at the branch which may be fetched by other collabrators who plans to continue works upon it.

# Reference： 
http://stackoverflow.com/questions/804115/when-do-you-use-git-rebase-instead-of-git-merge 
https://www.atlassian.com/git/tutorials/merging-vs-rebasing/conceptual-overview 
https://www.atlassian.com/git/articles/git-team-workflows-merge-or-rebase/ 

