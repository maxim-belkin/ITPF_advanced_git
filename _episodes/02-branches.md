---
title: "Git branches"
teaching: 20
exercises: 20
questions:
- "How do I create a new branch?"
- "How do I commit to a new branch?"
- "How do I merge two branches together?"
objectives:
- "Learn how to create, update, and merge branches."
keypoints:
- "`git branch` creates a new branch"
- "`git checkout <branchname>` switches to a branch called `<branchname>`"
- "`git merge <branchname>` merges branches called `<branchname>` into the currently checked out
    branch"
---

Let's go back to our main story line.
So, we're an employee in a Mega Store in one of its departments and our manager asks us to do two
things:

1. document how much stuff we have left (inventory control), and
2. develop a nice "About Us" page describing our department and why it is so great

We want to do both of these tasks immediately, but we don't want to screw anything up the main
`master` branch (which is continuously monitored by the store owner).
For that purpose we create a new branch in our repository:

~~~
git branch inventory_control
~~~
{: .language-bash}

When we execute this command, nothing changes in our working tree.
However, if we look at at the `.git/refs/heads` folder we'll see a new file there, called
`inventory_control`:

~~~
ls .git/refs/heads/
~~~
{: .language-bash}

~~~
inventory_control  master
~~~
{: .output}

Let's compare both files in that directory:

~~~
cat .git/refs/heads/master .git/refs/heads/inventory_control
~~~
{: .language-bash}

~~~
d1fbc13485d81ce536ce6ccd57d80ddd8e2a8a84
d1fbc13485d81ce536ce6ccd57d80ddd8e2a8a84
~~~
{: .output}

Yes, they are identical! And yes, this is all that Git has done when we executed that `git branch`
command -- it created a file in `.git/refs/heads` directory. Let's see how `git log` reports this to
us:

~~~
git log --oneline -1
~~~
{: .language-bash}

~~~
d1fbc13 (HEAD -> master, inventory_control) Adding 'Staff' section
~~~
{: .output}

Schematically, this is how it looks like:

![Creating a new branch with `git branch` command](../fig/svgs/git_branch_0.svg)

Note the new "inventory_control" label -- this is our new branch.
What we did here is quite simple: we created a new branch which, at the time of creation, is
identical to the active branch.

However, which branch are we one? In other words, if we create a new commit -- which label will
move?  `master` or `inventory_control`? Or, maybe, both?

In Git, currently checked out branch is pointed out by a special pointer called `HEAD`.
Just like with branches, it uses for this pointer a file called `HEAD` in `.git` directory.
Have a look at it!

~~~
cat .git/HEAD
~~~
{: .language-bash}

~~~
ref: refs/heads/master
~~~
{: .output}

Similarly to the output of `git log`, the content of this file tells us that our `HEAD`
still points to `master`.
It means that if we create a commit now, it will be

* created in the branch `master`, not `inventory_control`, and
* its parent will be the commit at which `HEAD` (and, therefore, `master`) points to.

To switch to the `inventory_control` branch we need to check it out:

~~~
git checkout inventory_control
~~~
{: .language-bash}

Let's see what changes in the output of `git log` after we do that:

~~~
git checkout inventory_control
git log --oneline -1
~~~
{: .language-bash}

~~~
d1fbc13 (HEAD -> inventory_control, master) Adding 'Staff' section
~~~
{: .output}

Excellent! We have now switched our `HEAD` to our newly created branch so new commits will be added
to it instead of `master`.


![Switching a branch with `git checkout` command](../fig/svgs/git_branch_1.svg)

> ## Git 201: Where am I?
>
> You don't have to use `git log` to figure out which branch you're on.
> Whenever you need to know your current branch, execute `git branch` without arguments and note
> the branch marked with an asterisk (`*`):
>
> ~~~
> git branch
> ~~~
> {: .language-bash}
>
> ~~~
> * inventory_control
>   master
> ~~~
> {: .output}
{: .callout}

Let's make couple of commits in this branch:
1. Let's create a file called `departments/<department>/inventory.yml` (replace `<department>` with
   the name of the department you've been assigned to) with a few items in it and commit it to the
   repository. This file will list items in our inventory  in the following format:
   ~~~
   item:
     count: <number>
     price: <number>
   ~~~
   {: .code}
2. We'll update this file to include one (or more) items and commit the changes again.
   Please use the department's name assigned to you.

~~~
mkdir -p departments/kids

cat > departments/kids/inventory.yml <<EOS
barbie:
  count: 35
  price: 12.5
pikachu:
  count: 10
  print: 19.99
EOS

git add departments/kids/inventory.yml
git commit -m "Kids department: inventory"

cat >> departments/kids/inventory.yml <<EOS
robot:
  count: 100
  price: 8.99
EOS

git add -u .
git commit -m "Kids department: added  robot"
~~~
{: .language-bash}

Now let's check that the right commits have been created in the right branch:

~~~
git log --oneline -3
~~~
{: .language-bash}

~~~
eb9ef42 (HEAD -> inventory_control) Kids department: added  robot
7d6e34d Kids department: inventory
d1fbc13 (master) Adding 'Staff' section
~~~
{: .output}

This output shows that `inventory_control` branch is two commits ahead of `master`.

![Branch `inventory_control` is 2 commits ahead of `master`](../fig/svgs/git_branch_2.svg)

Congratulations! We've created a branch and created a few commits in it. Now, let's switch back to
the `master` and see what happened there.

> ## Switching branches
>
> Recall the syntax that you used to switch to a branch. Use it to switch to the `master` branch.
>
> > ## Solution
> > ~~~
> > git checkout master
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}

~~~
ls
~~~
{: .language-bash}

~~~
README.md
~~~
{: .output}

Wow! There is no `departments` folder altogether! But don't worry - we haven't lost any of our work.
When we switched branches with `git checkout`, Git updated our working directory to match the state
at which it was when we started working on `inventory_control` branch.

## More work -- more fun!

Our supervisor approached us and asked to urgently add department's
working hours to a `README.md` file in `departments/kids` directory.

Well, we've done this many times already so please do it on your own.

> ## Solution. (Don't peek!) 🤨
> ~~~
> mkdir -p departments/kids
> cat > departments/kids/README.md <<EOS
> # Kids department
>
> Hours of operation: 8 AM - 8 PM
> EOS
> git add departments/
> git commit -m "Kids department: adding hours of operation"
> ~~~
> {: .language-bash}
{: .solution}

**Great job!**


## Merging branches

Now, we're ready to combine our changes we've added in the `inventory_control` branch to the main
`master` branch. In Git, it is as easy as:

~~~
# on branch master
git merge inventory_control
~~~
{: .language-bash}

with the only condition that we have to be on the branch with which we are merging the specified
branch. The above command will open your text editor to specify a commit message. By default, the
message will read:

~~~
Merge branch 'inventory_control'
~~~
{: .code}

Hit <kbd>Return</kbd> and wait a split second for Git to merge the branches.
Now, our Git repository history should look like this:

![Commit layout after `git merge`](../fig/svgs/git_branch_3.svg)

Let's have a look how Git shows such commits in the log.
We will use `git log` command again. However, this time we will specify a new option `--graph`

~~~
git log --oneline --graph -5
~~~
{: .language-bash}

~~~
*   9af2304 (HEAD -> master) Merge branch 'inventory_control'
|\
| * eb9ef42 (inventory_control) Kids department: added  robot
| * 7d6e34d Kids department: inventory
* | bac2779 Kids department: adding hours of operation
|/
* d1fbc13 Adding 'Staff' section
~~~
{: .language-bash}

Git shows that the last (top-most) commit in the history (`9af2304`) is connected to two commits:
`eb9ef42` and `bac2779`.

Now, let's look at this commit with `git cat-file`:

~~~
git cat-file -p 9af2304
~~~
{: .language-bash}

~~~
tree 5683ebbc08890f88f7d61a4e3dd76adc4db8253f
parent bac277955646cdf9c5d29320d01c9718a3e76423
parent eb9ef4233977497f092cfd6d84f474067031b7cc
author Maxim Belkin <maxim.belkin@gmail.com> 1573145267 -0600
committer Maxim Belkin <maxim.belkin@gmail.com> 1573145267 -0600
gpgsig -----BEGIN PGP SIGNATURE-----

 iQJLBAABCgA1FiEEoEV7mTIybOfwM4TyrHFWDUxfIzgFAl3ES4EXHG1heGltLmJl
 bGtpbkBnbWFpbC5jb20ACgkQrHFWDUxfIzjGZw/9EV3hn+vaMcDlHDTwCmzKA8MT
 uywUtqtPMQu7k/HqEKC6I3ZMbWMfYmbfKBvytd7XlZjydopQOxrGCcTwODs++x0c
 i2KPVoikrUGId+ww36W/4XlI4PD5D71CIZRfxpGnxvHhD1oI6DBvd4A3ttA4pL6D
 2N1OAx7iJcH2pxxikDQp06LVW9K3CQ8pT6AYj0eZ1JJR4OrT+OQ9LGlDuR5vZH4R
 Wzcvuhus8H946VWJLCgBgHY4YYyK77F6U150arTqLxyKR7s5hgJYX+Ok47qY/8Ca
 RxnmHGJ6tGbUxGypPcVr4Bo/6M5kmXI/XbmHaY6KDyO+pI6XRsZWNMGlHwxNxnNy
 Bq0yTQ8KA8WWDwg+JewcFS2Mrhc+umt6ujKYZw9oNQA394xBezs+mS3uAuwpwlFm
 owOD9WXGwMAU1fpSUkg+hMrDuLrclndgUkL0A31YU7d4m7/AArYl4O4R9QZ9vKgn
 4DJBWtZ9BsfjA3mvpbYJw/nKwr09y6s0hlwq9u+qbR9B+7oZdVAgBcouGHIHSkAu
 qeT5ffJhdMJi+yVfgSszH0KiOj58l3yqb0paw7AMW6RVG8422/14n9QUdt/ys5SF
 RRVNnh0psjbnyzWJMLp8Sc4NhScbP3RotIuSdD4x7PFwmfCjhu0ruHTbIjVplhdU
 kTb32TcSCD8GXkpuS3w=
 =Z/Ig
 -----END PGP SIGNATURE-----

Merge branch 'inventory_control'
~~~
{: .output}

Note the two `parent` lines near the top of the provided output.
When a commit has more than one parent, it is called a "merge" commit.
This brings us to the following classification:

In Git, commits may have:

- 0 "parents". These commits are called "orphans"
- 1 "parent". These are normal commits.
- 2+ "parents". These commits are called "merges".

We can list all these types of commits using `--max-parents=` and `--min-parents=` options for
`git log`:

~~~
git log --max-parents=0 --oneline # list 'orphans' (e.g., initial commits)
git log --min-parents=2 --oneline # list 'merges'
~~~
{: .language-bash}

~~~
4685727 Initial commit.
9af2304 (HEAD -> master) Merge branch 'inventory_control'
~~~
{: .output}


> ## List regular commits
>
> How would you list regular commits only?
>
> > ## Solution
> >
> > Unfortunately, there is a `--parents` option that does something else: it  adds hashes of
> > parent commits to the output of `log` (try it when you get a chance). The proper solution is,
> > therefore, less elegant than it could've been:
> >
> > ~~~
> > git log --max-parents=1 --min-parents=1 --oneline
> > ~~~
> > {: .language-bash}
> >
> > Also, note, that you can quickly list all merges in the repository using `--merges` flag:
> > ~~~
> > git log --merges --oneline
> > ~~~
> > {: .language-bash}
> {: .solution}
{: .challenge}

[git-branching]: https://git-scm.com/book/en/v1/Git-Branching-What-a-Branch-Is

{% include links.md %}

