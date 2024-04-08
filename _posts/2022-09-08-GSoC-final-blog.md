---
title: "GSoC @ Git Final Report"
categories:
  - blog
tags:
  - learn
---

My GSoC project: [More Sparse Index Integrations](https://summerofcode.withgoogle.com/programs/2022/projects/hz4rcOUB)

My proposal: [GSoC 2022 Git Contributor Proposal](https://docs.google.com/document/d/1VZq0XGl-wCxECxJamKN9PGVPXaBhszQWV1jaIIvlFjE/edit?usp=sharing)

Previous blogs:  
[GSoC Week 1](https://ffyuanda.github.io/blog/GSoC-week-1/)  
[GSoC Week 2](https://ffyuanda.github.io/blog/GSoC-week-2/)  
[GSoC Week 3 - 4](https://ffyuanda.github.io/blog/GSoC-week-3-4/)  
[GSoC Week 5 - 6](https://ffyuanda.github.io/blog/GSoC-week-5-6/)  
[GSoC Week 7 - 8](https://ffyuanda.github.io/blog/GSoC-week-7-8/)  
[GSoC Week 9](https://ffyuanda.github.io/blog/GSoC-week-9/)  

# Summary

During my GSoC project, I was trying to integrate a series of commands with
sparse index. There are three commands that I've been working on:
`git-mv`, `git-rm`, and `git-grep`. I'll explain what the situation was,
what I've done, and what is the result for each command.

`git-mv`

* Series "out-to-in" (in branch 'master')  
[GitHub branch](https://github.com/git/git/commit/0455aad1e39f21acbaa8a84129fec8eb77682e0d)  
[Mailing list](https://lore.kernel.org/git/20220331091755.385961-1-shaoxuan.yuan02@gmail.com/)  
mv: add check_dir_in_index() and solve general dir check issue  
mv: use flags mode for update_mode  
mv: check if `<destination>` exists in index to handle overwriting\
mv: check if out-of-cone file exists in index with SKIP_WORKTREE bit\
mv: decouple if/else-if checks using goto\
mv: update sparsity after moving from out-of-cone to in-cone\
t1092: mv directory from out-of-cone to in-cone\
t7002: add tests for moving out-of-cone file/directory\

* Series "in-to-out" (in branch 'next')  
[GitHub branch](https://github.com/git/git/commit/2316d9ce4d2cd85bdda3111358fc6b76af9f231a)  
[Mailing list](https://lore.kernel.org/git/20220719132809.409247-1-shaoxuan.yuan02@gmail.com/)  
mv: check overwrite for in-to-out move  
advice.h: add advise_on_moving_dirty_path()  
mv: cleanup empty WORKING_DIRECTORY  
mv: from in-cone to out-of-cone  
mv: remove BOTH from enum update_mode  
mv: check if `<destination>` is a SKIP_WORKTREE_DIR  
mv: free the with_slash in check_dir_in_index()  
mv: rename check_dir_in_index() to empty_dir_has_sparse_contents()  
t7002: add tests for moving from in-cone to out-of-cone  

`git-rm`

* Series "sy/sparse-rm" (in branch 'master')  
[GitHub branch](https://github.com/git/git/commit/9b9445cfdec254dfd5e78fb00ec4476cee3d578c)  
[Mailing list](https://lore.kernel.org/git/20220803045118.1243087-1-shaoxuan.yuan02@gmail.com/)  
rm: integrate with sparse-index  
rm: expand the index only when necessary  
pathspec.h: move pathspec_needs_expanded_index() from reset.c to here  
t1092: add tests for `git-rm  

`git-grep`

* Series "sy/sparse-grep" (in branch 'seen')  
[GitHub branch](https://github.com/git/git/commit/81ae80e29855f75043008b07071c8a604f2de8ab)  
[Mailing list](https://lore.kernel.org/git/20220817075633.217934-1-shaoxuan.yuan02@gmail.com/)  
builtin/grep.c: walking tree instead of expanding index with --sparse  
builtin/grep.c: integrate with sparse index  
builtin/grep.c: add --sparse option  

Notice that this report is trying to be brief, as all the information is
better preserved in the corresponding commit messages. View the commits and
their messages for better context and analysis.

# `git-mv`

## Situation

When I was trying to integrate `git-mv` with sparse index, I noticed some
interesting behaviors playing with the command. I realized that when moving
a file across the cone boundary, namely moving from inside of cone to outside
of cone, or vice versa, Git did not check whether the resulting file complies
with the sparse-checkout definition. For example, in command `git mv <f1> <f2>`,
if `<f1>` is out-of-cone and `<f2>` is in-cone, Git should check out the
resulting file. In conclusion, the compatibility between `git-mv` and
sparse-checkout was not ideal.

## What I've done

1. `git-mv` from out-of-cone to in-cone

Before this series, running `git mv <f1> <f2>`, with `<f1>` being out-of-cone and
`<f2>` being in-cone, Git errored because it cannot locate `<f1>` in the working
tree.

Changes are made so that when `<f1>` (either a regular file or a directory)
is out-of-cone, and when `--sparse` switch is used, Git checks the index to see
if `<f1>` exists and move it to the destination in the index, and check out the
file/directory to the worktree.

2. `git-mv` from in-cone to out-of-cone

Before this series, running `git mv <f1> <f2>`, with `<f1>` being in-cone and
`<f2>` being out-of-cone, was not possible, mainly because `<f2>` is a
destination that does not exist in the worktree.

Changes are made so we can make such move. When such move is a clean move, i.e.
`<f1>` has no unstaged changes, Git now moves `<f1>` to `<f2>` in the index,
then deletes `<f1>` from the worktree and turns on its CE_SKIP_WORKTREE bit (as it
is now out-of-cone). A dirty move should move `<f1>` to `<f2>`, both in the
worktree and the index, but should *not* remove the resulting path from the
working tree and should *not* turn on its CE_SKIP_WORKTREE bit.` 

## Results

`git-mv` can now handle out-to-in and in-to-out moves and is more compatible
with sparse-checkout.

There are still left-over-bits for `git-mv`:

1. out-to-out move.

What happens when both `<f1>` and `<f2>` are out-of-cone locations?
I honestly don't know, but this is the next question I'll dig.

2. sparse index integration.

After the out-to-out move, we can try to integrate `git-mv` with sparse index.
It'll be interesting to see how the new across cone logics react with sparse
index.

# `git-rm`

## Situation

`git-rm` was relatively compatible with sparse-checkout when I saw it. So the
integration with sparse index was started without much hindrance.

## What I've done

Before my patches, `git-rm` expands a sparse index unconditionally, which is
expensive. I reused an existing method `pathspec_needs_expanded_index()` to
determine if a pathspec needs a full index. Now, when the pathspec does not
match things outside of sparse-checkout definition, Git does not expand the
index unnecessarily.

Because `pathspec_needs_expanded_index()` seems to be a good candidate to be
reused elsewhere, I extracted it to be a public method for future usage.

After testing, I turned off the `command_requires_full_index` for `git-rm`,
marking this command as compatible with sparse index.

## Result

The `p2000` tests demonstrate a ~92% execution time reduction for
`git rm` using a sparse index.

# `git-grep`

## Situation

Unlike previous commands, `git-grep` does not have a `--sparse` option at
the time. Also, unlike `git-mv` or `git-rm`, `git-grep` does not write to the
index or worktree, it simply retrieves information from the repo. Like `git-rm`,
`git-grep` seems to be practically compatible with sparse-checkout, so we can
start the sparse index integration promptly.

## What I've done

The first thing I did is to add a `--sparse` option to the command, so we can
decide when to grep "sparse contents". 

After adding the new `--sparse` option, I added more tests to make sure the
sparse expands only when using `--sparse` and the behavior is as expected.

Using `--sparse` to switch on/off the sparse index is an easy optimization.
When using `--sparse`, the index expands as before this integration, which as
noted by my mentor, could be further optimized by implementing a custom
tree-walking logic.

I implemented a tree-walking logic to walk trees according to a given pathspec,
so Git is faster when the pathspec encloses an area smaller than the HEAD tree.

## Result

Without `--sparse`, the integration demonstrates a ~99.4% execution time
reduction for `git grep` using a sparse index.

Tree-walking logic also adds some speed (depending on the pathspec) to the
operation. For example, running `git grep --cached --sparse bogus --"f2/f1/f1/*"`
in the p2000 test setup repo gives a ~71% execution time reduction using this
tree-walking logic.

# Closing remarks

I was going to write about what I've learned along the way, but then I realized
that there are too many to be listed. I didn't write a single line of C or Bash
before GSoC, and now I can gradually catch up with some of the work in one of
the most influential communities.

If I have to put something down, I'll say, always try to learn actively and work
collaboratively. 

During these four months, every day of working added something
new to my knowledge base. Learning the codebase of Git and the tech stack used
is like drinking from a fire hose for me. Every change to be made requires researching,
referencing, and asking myriad questions. So I realized that active learning is
a key to understanding the work and keep making progress.

In a community, communication and collaboration can be more important than
grinding hard problems and being a hero. Discussing with fellow developers and
exchanging ideas are very crucial to deliver satisfactory work. More than often,
good or even great ideas come from bringing everybody's ideas together. When
I'm stuck on the work, I learned to ask for help from my mentors more actively,
instead of hitting against hard rock and getting nowhere.

More importantly, I'll maintain my passion for open source technologies and
be active as a young contributor. I'll also try to be helpful to the community
and people who are also interested in the open source world.

Thank you, my mentors, Victoria and Derrick.
Thank you, Git and the community.
Thank you, GSoC.

Thanks,
Shaoxuan Yuan
