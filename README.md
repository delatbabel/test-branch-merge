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

## What if you have already created f2 after f1 was merged, but don't want to merge f1?

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

# Reverting Merges

What if I have created branches f1 and f2, merged both, but I want to undo the merge of f1?

As a bit of ASCII art, your develop tree may now look like this:

```
 ---o1--o2--o3--M1--x1--x2--x3--x4--x5----M2
     \         /             \           /
      \---A---B   feature/f1  \         /
                               \---X---Y  feature/f2
```

The problem here is now how to revert feature/f1 but leave the changes for feature/f2 in place.

Firstly, you should start by reading this mail posting by Linus Torvalds: https://mirrors.edge.kernel.org/pub/software/scm/git/docs/howto/revert-a-faulty-merge.txt and also this stackoverflow post: https://stackoverflow.com/questions/12534312/revert-a-merge-after-being-pushed

There are two `git` commands that we will discuss here, which are `git reset` and `git revert`. It may help to read their instruction and tutorial pages here:

* https://git-scm.com/docs/git-reset and https://www.atlassian.com/git/tutorials/undoing-changes/git-reset
* https://git-scm.com/docs/git-revert and https://www.atlassian.com/git/tutorials/undoing-changes/git-revert

The difference between these two commands is:

* `git reset` resets the commits on a git branch by removing all of the commits after that point.
* `git revert` inserts a new commit into the commit tree that undoes all of the changes in one particular commit.

Looking again at the above commit tree, there are changes marked x1, x2, x3, etc on the tree which have nothing to do with either feature/f1 nor feature/f2. So we don't want to undo those changes.  We want to undo the changes in feature/f1 which have been merged with commit M1, but leave the changes in feature/f2 which have been merged with commit M2.  There are two simple ways that we can go about this:

## Method 1 -- Undo the latest merge, then rebase

* use `git reset --hard x5` which will undo the merge of the feature/f2 branch to the `develop` branch.  This just sets the HEAD of the `develop` branch to commit x5 (note that you have to use the SHA hash of commit x5 here, not just say "x5"), which has the effect of undoing your merge of feature/f2.  Note that we can only do this if there are no commits after M2, because those commits will all be lost.
* rebase the feature branch f2 to the BASE of feature/f1 by doing:

```
git checkout feature/f2
git rebase --onto x3 o1
```

This is the same process that I described in the section titled **What if you have already created f2 after f1 was merged, but don't want to merge f1?** above.

Although we have not reverted `feature/f1` from the `develop` branch, we are now free to merge `feature/f2` to the `release` branch as we were discussing above, and this will not merge any of the changes on branch `feature/f1`. This is ideal if you want to continue developing on your `develop` branch, e.g. to fix any problems introduced by feature f1 before merging that to the `release` and then `main` branches.

Finally we need to force push the changes using:

```
git push --force
```

## Method 2 -- using git revert

`git revert` is different to using `git reset` in that it doesn't undo any changes. Instead it inserts a new commit into the source tree that has the effect of undoing all of the changes in a previous commit. It's probably best I explain this by paraphrasing from Linus' post (adjusting branch and commit names). Let's start with our original position again:

```
 ---o1--o2--o3--M1--x1--x2--x3--x4--x5----M2
     \         /             \           /
      \---A---B   feature/f1  \         /
                               \---X---Y  feature/f2
```

We now do `git revert M1` which pushes a new change to the end of the `develop` branch which contains the opposite of the change in M1, which is the commit that we created when we merged `feature/f1`:

```
 ---o1--o2--o3--M1--x1--x2--x3--x4--x5----M2---W1
     \         /             \           /
      \---A---B   feature/f1  \         /
                               \---X---Y  feature/f2
```

I'll continue Linus' convention of using "W" to be the opposite of "M" because it's an upside-down "M".

Now continuing with Linus' post, looking at just feature/f1:

---

The history immediately after the "revert of the merge" would look like this:

```
 ---o---o---o---M---x---x---W
               /
       ---A---B
```

where A and B are on the side development that was not so good, M is the
merge that brings these premature changes into the mainline, x are changes
unrelated to what the side branch did and already made on the mainline,
and W is the "revert of the merge M" (doesn't W look M upside down?).
IOW, `"diff W^..W"` is similar to `"diff -R M^..M"`.

Such a "revert" of a merge can be made with:

    $ git revert -m 1 M

After the developers of the side branch fix their mistakes, the history
may look like this:

```
 ---o---o---o---M---x---x---W---x
               /
       ---A---B-------------------C---D
```

where C and D are to fix what was broken in A and B, and you may already
have some other changes on the mainline after W.

If you merge the updated side branch (with D at its tip), none of the
changes made in A or B will be in the result, because they were reverted
by W.  That is what Alan saw.

Linus explains the situation:

    Reverting a regular commit just effectively undoes what that commit
    did, and is fairly straightforward. But reverting a merge commit also
    undoes the _data_ that the commit changed, but it does absolutely
    nothing to the effects on _history_ that the merge had.

    So the merge will still exist, and it will still be seen as joining
    the two branches together, and future merges will see that merge as
    the last shared state - and the revert that reverted the merge brought
    in will not affect that at all.

    So a "revert" undoes the data changes, but it's very much _not_ an
    "undo" in the sense that it doesn't undo the effects of a commit on
    the repository history.

    So if you think of "revert" as "undo", then you're going to always
    miss this part of reverts. Yes, it undoes the data, but no, it doesn't
    undo history.

In such a situation, you would want to first revert the previous revert,
which would make the history look like this:

```
 ---o---o---o---M---x---x---W---x---Y
               /
       ---A---B-------------------C---D
```

where Y is the revert of W.  Such a "revert of the revert" can be done
with:

    $ git revert W

This history would (ignoring possible conflicts between what W and W..Y
changed) be equivalent to not having W or Y at all in the history:

```
 ---o---o---o---M---x---x-------x----
               /
       ---A---B-------------------C---D
```

and merging the side branch again will not have conflict arising from an
earlier revert and revert of the revert.

```
 ---o---o---o---M---x---x-------x-------*
               /                       /
       ---A---B-------------------C---D
```

Of course the changes made in C and D still can conflict with what was
done by any of the x, but that is just a normal merge conflict.

On the other hand, if the developers of the side branch discarded their
faulty A and B, and redone the changes on top of the updated mainline
after the revert, the history would have looked like this:

```
 ---o---o---o---M---x---x---W---x---x
               /                 \
       ---A---B                   A'--B'--C'
```

If you reverted the revert in such a case as in the previous example:

```
 ---o---o---o---M---x---x---W---x---x---Y---*
               /                 \         /
       ---A---B                   A'--B'--C'
```

where Y is the revert of W, A' and B' are rerolled A and B, and there may
also be a further fix-up C' on the side branch.  `"diff Y^..Y"` is similar
to `"diff -R W^..W"` (which in turn means it is similar to `"diff M^..M"`),
and `"diff A'^..C'"` by definition would be similar but different from that,
because it is a rerolled series of the earlier change.  There will be a
lot of overlapping changes that result in conflicts.  So do not do "revert
of revert" blindly without thinking..

```
 ---o---o---o---M---x---x---W---x---x
               /                 \
       ---A---B                   A'--B'--C'
```

In the history with rebased side branch, W (and M) are behind the merge
base of the updated branch and the tip of the mainline, and they should
merge without the past faulty merge and its revert getting in the way.

To recap, these are two very different scenarios, and they want two very
different resolution strategies:

* If the faulty side branch was fixed by adding corrections on top, then doing a revert of the previous revert would be the right thing to do.

* If the faulty side branch whose effects were discarded by an earlier revert of a merge was rebuilt from scratch (i.e. rebasing and fixing, as you seem to have interpreted), then re-merging the result without doing anything else fancy would be the right thing to do.

However, there are things to keep in mind when reverting a merge (and
reverting such a revert).

For example, think about what reverting a merge (and then reverting the
revert) does to bisectability. Ignore the fact that the revert of a revert
is undoing it - just think of it as a "single commit that does a lot".
Because that is what it does.

When you have a problem you are chasing down, and you hit a "revert this
merge", what you're hitting is essentially a single commit that contains
all the changes (but obviously in reverse) of all the commits that got
merged. So it's debugging hell, because now you don't have lots of small
changes that you can try to pinpoint which _part_ of it changes.

But does it all work? Sure it does. You can revert a merge, and from a
purely technical angle, Git did it very naturally and had no real
troubles. It just considered it a change from "state before merge" to
"state after merge", and that was it. Nothing complicated, nothing odd,
nothing really dangerous. Git will do it without even thinking about it.

So from a technical angle, there's nothing wrong with reverting a merge,
but from a workflow angle it's something that you generally should try to
avoid.

If at all possible, for example, if you find a problem that got merged
into the main tree, rather than revert the merge, try _really_ hard to
bisect the problem down into the branch you merged, and just fix it, or
try to revert the individual commit that caused it.

Yes, it's more complex, and no, it's not always going to work (sometimes
the answer is: "oops, I really shouldn't have merged it, because it wasn't
ready yet, and I really need to undo _all_ of the merge"). So then you
really should revert the merge, but when you want to re-do the merge, you
now need to do it by reverting the revert.
