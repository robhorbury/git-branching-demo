# Demo on Git, Branching, Rebasing and Merging

This repo is designed to give you a series of demos on git branching.
Specifically, we will look at merging, rebasing and cherry-picking commits.

## Step 1: Setup

We need to create a new repo for this demo. We will create it in the parent
folder to this repo. To do this we need to change our directory to the parent
file, then make a new folder, go into it, and initialise a repo.
``` sh
cd ..
mkdir git_demo_repo
cd git_demo_repo
git init
```
We now have a new repo - Lets add something to it. To do this, we will create a
new file, and then commit it.

```sh
touch A
git add A
git commit A -m "Added A"
```
We now have a file called `A` in our repo.


## Step 2: Branching

Problem: We want to add something to the repo, but we can't add it on the main
branch?

Solution: Create a branch!

We are going to create a new branch, and add our changes to it. We can then
merge our changes back into the main branch at a later point.

To create a new branch we use
```sh
git branch <branch name>
```
In this case, we will create a branch called `step2`,
so we run `git branch step2`.

We have now created a new branch - but we are still looking at the main branch:
```
➜ git-branching-basics (main)
```
To switch to the new branch, we need to use `git checkout` or `git switch`. I
am in the habit of using _checkout_, so I will run:
```sh
git checkout step2
```

Now we are on a new branch, we want to make some changes.

We will make two changes, Create a file called `B` and commit it:
```sh
touch B
git add B
git commit B -m "added B"
```
Do the same again, but for a file called `C`

```sh
touch C
git add C
git commit C -m "added C"
```
Our repo will now look like this:
```
    B - C *step2
   /
  A       main
```
So far, there has been no divergence from main - we can simply add these commits
on top of main. This is what is called a _fast forward_ - We don't have to
resolve any issues, we can simply just add the commits.

### Fast Forward Merge

Lets do this as a demo. First, lets create a copy of main. We are going to do
this so that we can demonstrate the effect of a merge, within changing the state
on main:

```sh
git checkout main
git checkout -b main_fast_forward_merge
```

Now this is our current state:
```
    B - C step2
   /
  A       *main_fast_forward_merge
```

If we want to merge `step2` into `main_fast_forward_merge`, we simply need to
run:
```sh
git merge step2
```
Notice in the terminal, we will see the phrase `fast-forward`. This means git
has merged without creating a merge commit, since there was none to resolve.

Our branch now looks like this:

```
  A - B - C     *main_fast_forward_merge
```
This is our first example of a `linear history`. This means that after a merge,
our branch history is _linear_ - There are no branches or merge commits present.

### Merge

So what happens if we make a change to main, before we merge the branch?

Lets try that - first we will add new files and commit on main:
```sh
git checkout main
touch D
git add D
git commit D -m "added D"
touch E
git add E
git commit E -m "added E"
```
Now we have this:
```
    B - C     step2
   /
  A - D - E   main
```
This is where it gets interesting - We can no longer _fast forward_ our step2
branch onto main. How can we combine these two branches? Practically, we have 2
options, _merge_ and _rebase_.

>[!IMPORTANT]
>Merging attempts to combine branches that have diverged from a common past.
>When the branches are combined, a new _merge commit_ is created.

This new commit resolves the
two branches, and indicates that two branches have been merged together.

Lets try it - but first, lets create a copy of our main branch. That we we can
come back to this point later, and compare the effect of rebasing.

Create a copy:
```sh
git checkout main
git branch main_to_merge
git checkout main_to_merge
```
This is now our setup:
```
    B - C    step2
   /
  A - D - E  main_to_merge
```
To merge these branches, we need to be on the branch that we want to merge
_into_. So in this case, we want to be on `main_to_merge`
```sh
git checkout main_to_merge
git merge step2
```
Immediately, you will notice that we need to enter a commit message - we did not
need to do this for the _fast forward_ merge. This is telling us that git is
resolving the differences in the branch, and this new commit signifies that.

Once we commit the merge, we can take a look at the state of the repo:

```
     B --- C
   /         \
  A - D - E - M  main_to_merge
```

Our branch now contains a new commit - where the merge occurred. In the repo
history, we can see _exactly_ what has happened. We can see the old branch,
and the commit that occurred at the same time as the merge.

In this case, the history of our branch is **not** linear.

### Rebasing

The other option we have to resolve this situation is to _rebase_ our feature
branch - but _what is rebasing?_

>[!IMPORTANT]
>As the name suggests, rebasing is the proccess of chaging the commit that a
>branch points to - _changing the base of the branch_.

Lets give rebase a go, recall we have the following situation:

```
     B --- C   step2
   /
  A - D - E    main
```

Once more, lets create a backup for the `main` branch:
```sh
git checkout main
git checkout -b main_to_use_in_rebase
```

Rebase works slightly differently to merge - we need to rebase our
_feature branch_ on _main_. This means we need to checkout the feature branch:

```sh
git checkout step2
git rebase main_to_use_in_rebase
```

Notice the state of the repo:
```
            B --- C   *step2
           /
  A - D - E           main_to_use_in_rebase
```
By rebasing, we have not changed the main branch, we have only changed the
base of our feature branch. This was done by adding the commits on the main
branch onto `step2` until we reached the end of the main branch.

Notice that `step2` no longer diverges from `main_to_use_in_rebase` - i.e we
can _fast forward_ merge it:
```sh
git checkout main_to_use_in_rebase
git merge step2
```
and now:
```
  A - D - E - B - C     *main_to_use_in_rebase
```
We have a linear history - the branches are combined without a merge commit.


#### Pros

As we have seen, rebasing can give us a linaer history. This can make it really
clear exactly which commit introduced which change - and if we want to undo
a commit in the future, this will be _much_ easier with a linear history.

It isn't just undoing commits which is made easier with a linear history - It
makes it easier to selectively cherry-pick commits, as well as well as work out
the commit that introduced certain features or bugs.

#### Cons

When we rebased `step2`, what we effectively did was add the commits from `main`
at the start of `step2` - this _changes the history_ of the `step2` branch.

If this branch is just local, then this is not a problem. If this branch has
been pushed to GitHub, then this will cause a conflict between our branch and
the remote branch.

Equally, if someone else is working on the same branch, then this will cause a
conflict with their branch.

>[!IMPORTANT]
> For this reason, it is best practice to **only** rebase local branches, or
> feature branches that only you are working on. You should not rebase the main
> branch, or shared branches

### Dealing with conflicts

Both rebasing and merging have the potential to generate conflicts. If the
commits on either of the branches modify the same piece of code, then a conflict
will occur.

When we merge the branches, we deal with the conflicts in the merge commit.

When we rebase, we will deal with the conflicts as the branch is rebased.
When we resolve the issues, there will be no record on the conflict occuring,
since the history will be updated in the rebase.

