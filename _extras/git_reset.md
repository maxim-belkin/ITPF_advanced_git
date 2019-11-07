---
title: "Git reset"
permalink: /extras/reset.html
---

## Git reset

{% comment %}
teaching: 20
exercises: 20
questions:
- "What does `git reset` do?"
- "What do `--soft`, `--mixed`, `--hard` options do?"
- "When should I use it?"
{% endcomment %}

`git reset` is a useful tool when you need to fix / update / simplify or, vice a versa,
complicate your local history.

Let's have a look at the effects of `git reset` with different flags.
Let's assume we've got a repository that has the following history:

![Initial state of the repository before applying any `git reset`](../fig/svgs/git_reset_0.svg)

With `--soft` flag, all Git
1. moves the `HEAD` and branch pointers to the specified location,

![Effect of `git reset --soft HEAD~2`](../fig/svgs/git_reset_1.svg)

With `--mixed` flag (which is the default flag if you don't specify any flag), Git:

1. moves the `HEAD` and branch pointers to the specified location,
2. "clears" out the Index.

![Effect of `git reset --mixed HEAD~2`](../fig/svgs/git_reset_2.svg)

With `--hard` flag, Git:

1. moves the `HEAD` and branch pointers to the specified location,
2. "clears" out the Index.
3. resets working directory to the state that corresponds to the specified location.

![Effect of `git reset --hard HEAD~2`](../fig/svgs/git_reset_3.svg)

