# test-branch-merge

Comparing strategies for merging multiple branches

# Note on branch naming conventions

In this guide I use the branch name `main` for the production release branch and `develop` for the development branch. These branches may have different names in your organisation, such as `master` or `production` or `dev`. Call them what you like, it can all be set up in the `git flow init` command.

# HOWTO -- 2 Feature Branches in Parallel

## Initialise Gitflow

```
git flow init
git push --set-upstream origin develop
```

You can now run this to confirm that we are on the `develop` branch:

```
git status
```

## Create Feature Branches

```
git flow feature start f1
git checkout develop
git flow feature start f2
```

Note at this point I have created 2 feature branches with the same root point at the `develop` branch.  We will explore how to fix issues created when `f1` has already been merged when you start to create `f2` later.

## Start Working on Feature f1

```
git checkout feature/f1
cat <<EOF1 >FEATURE_f1.txt
This file belongs to feature f1
EOF1
git add FEATURE_f1.txt 
git commit -am 'Added some code in f1' 
git push --set-upstream origin feature/f1
```

Note we are not going to finish feature `f1` yet.

## Start Working on Feature f2

```
git checkout feature/f2
cat <<EOF2 >FEATURE_f2.txt
This file belongs to feature f2
EOF2
git add FEATURE_f2.txt 
git commit -am 'Added some code in f2' 
git push --set-upstream origin feature/f2
```

Note we are not going to finish feature `f2` yet either.  We will handle merging later.  To see the list of branches we have created so far, you can use this command:

```
git branch
```

## Merge both branches to develop

As always we merge both feature branches to develop, but again we don't finish the branches because we're not ready to merge to main yet.

```
git checkout develop
git merge feature/f1
git merge feature/f2
git push
```

Note that we are now on the develop branch with both features `f1` and `f2` merged in. You can verify that using these commands:

```
git status
ls -l
```

You can see that both `FEATURE_f1.txt` and `FEATURE_f2.txt` files are present.

## Merge f2 but not f1 to main

At this point it's easy to merge `f2` to `main` but not merge `f1`, because we have branched them at the same root point:

```
git checkout main
git merge feature/f2
git push
```

We are now on the main branch and with only feature `f2` merged in, but not feature `f1`.  You can check your local repository by doing this:

```
git status
ls -l
```

You will see the `FEATURE_f2.txt` file but not the `FEATURE_f1.txt` file.

# What If I Have Already Merged f1 Before Branching f2?

In the case where feature `f1` has already been merged to develop before work is started on `f2`, but you now need to merge `f2` to master but not `f1`, there are a number of possible solutions:

## If you have not branched f2 yet

If the features are not inter-dependent and you have not yet branched feature `f2` yet, then do not branch it from the head of the `develop` branch, but instead branch it from the the point where `f1` as branched from. You need to use a repository browser to find the SHA of the commit, as described here: https://stackoverflow.com/questions/2816715/branch-from-a-previous-commit-using-git

Let's try this with 2 new feature branches, `f3` and `f4`:

```
git checkout develop
git flow feature start f3
cat <<EOF3 >FEATURE_f3.txt
This file belongs to feature f3
EOF3
git add FEATURE_f3.txt 
git commit -am 'Added some code in f3' 
git push --set-upstream origin feature/f3
```

We will merge `f3` immediately before creating `f4`

```
git checkout develop
git merge feature/f3
git push
```

Now we create the `f4` branch, not from the HEAD but from the same point where `f3` got branched:

```
git log
# I see that my previous commit hash is deb88fdb591a1cd8509cb62d5fc2d0c583ef1ebf
# I could also use HEAD-1 here.
# Note that I have to manually use the git commands, not the gitflow shortcut:
git branch feature/f4 deb88fdb591a1cd8509cb62d5fc2d0c583ef1ebf
git checkout feature/f4
cat <<EOF4 >FEATURE_f4.txt
This file belongs to feature f4
EOF4
git add FEATURE_f4.txt 
git commit -am 'Added some code in f4'
git push --set-upstream origin feature/f4
```

I can now repeat the same process I went through above -- I can merge the `f4` branch into `develop`, and then also merge it to main but not merge `f3` into `main`:

```
git checkout develop
git merge feature/f4
git push

git checkout main
git merge feature/f4
git push
```

Note that this has also merged feature `f1` into `main`! This is because we created the `f4` branch in `develop` after we merged the `f1` branch to `develop`. So the merge of f4 into main picked up all of the commits that had been made to develop prior to `f4` being created, which included `f1` (but not `f3`).  To resolve this we can use branch rebasing or use a release branch, which we will discuss next.

## What if you have already created f2 after f1 was merged, but don't want to merge the f1?

If the feature `f2` has already been branched after feature `f1` has been merged (e.g. to `develop`), but you need to merge it to `main` without bringing across the commits from feature `f1`, then you need to rebase feature `f2` back to the point where `f1` was branched from. Once again you need to use a repository browser (or `git log`) to find the SHA of the commit at the root of the feature `f1` branch as described above. There are a few different strategies for doing this, such as using the `--onto` parameter of `git rebase`, or using a temporary branch at the point where you want to rebase to and then rebase to the temporary branch. They are described in this posting: https://stackoverflow.com/questions/7744049/git-how-to-rebase-to-a-specific-commit

As a quick guide, the following answer gives a description:

* Find a previous branching point of the branch to be rebased (moved) - call it old parent. In the example that's A
* Find commit on top of which you want to move the branch to - call it new parent. In the example that's B
* You need to be on your branch (the one you move):
* Apply your rebase: `git rebase --onto <new parent> <old parent>`

In the example above that's as simple as:

```
git checkout feature/f2
git rebase --onto B A
```

Let's try this now with two new branches called f5 and f6:

```
git checkout develop
git flow feature start f5
cat <<EOF5 >FEATURE_f5.txt
This file belongs to feature f5
EOF5
git add FEATURE_f5.txt
git commit -am 'Added some code in f5' 
git push --set-upstream origin feature/f5
```

We will merge f5 immediately before creating f6

```
git checkout develop
git merge feature/f5
git push
```

Now we create f6:

```
git checkout develop
git flow feature start f6
cat <<EOF6 >FEATURE_f6.txt
This file belongs to feature f6
EOF6
git add FEATURE_f6.txt
git commit -am 'Added some code in f6'
git push --set-upstream origin feature/f6
```

Now if we merge `f6` we will also bring in the changes on branch `f5` which we don't want to do, so we wil rebase before merge:

```
git checkout develop
git log
```

We can see that the commit where feature `f5` was merged was `a2318a96c43a5af2b6f4898e044257f59e22d5a1` but we want to rebase onto `042a274ee4317092fcaafcb6c9c057224bc617a7` which is the commit before `f5` was merged.  The syntax of `git rebase --onto` is:

```
git rebase --onto <new-parent> <old-parent>
```

So now we can do that:

```
git checkout feature/f6
git rebase --onto 042a274ee4317092fcaafcb6c9c057224bc617a7 a2318a96c43a5af2b6f4898e044257f59e22d5a1
```

Note that as we were working in the `feature/f6` branch we could see the file created by `feature/f5` but when we ran that last command, that file disappeared!

```
git push --force
```

Note the `--force` parameter on `git push` is needed here to force the upstream to accept our local rebasing.

We can now merge `f6` to `main` without bringing in the changes on `f5`. Note that this will also bring in the changes on `f3` which we avoided earlier as a side-effect, because we rebased `f6` to the root of `f5`. To avoid bringing in the `f3` changes, we should have rebased to the root of `f3`:

```
git checkout develop
git merge feature/f6
git push

git checkout main
git merge feature/f6
git push
```

We can now see that we have successfully merged `f6` into `main` without merging `f5`.

# Using a Release Branch

Using a release branch is another useful strategy. I'm not going to cover it in detail here but fundamentally the approach is as follows:

Create a `release` branch based on the `main` branch:

```
git checkout main
git checkout -b release
```

For each branch that needs to be merged to the `release` branch, use the `git merge --onto` commands as detailed above to ensure that later branches do not get merged. Then for each branch do this command once:

```
git merge feature/fx # replace "fx" with your actual feature name
```

You now have a `release` branch that can be deployed into your testing or staging or other pre-release environment.  Once you're happy with that, then you can merge the release branch into `main` using this command:

```
git checkout main
git merge release
```

At this point you can remove your `release` branch and recreate it later for your next release, or you can choose to leave it in place as a permanent record (give each release branch a different name in that case).
