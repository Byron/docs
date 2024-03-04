---
description: Conflicts happen sometimes, what to do when we get there?
---

# 🍡 Merging

So, let's talk a little bit more about merging. Merging in GitButler is a little different than in Git because by default, all applied virtual branches are actually merged in your working directory, as though you had run a `git merge` or `git pull` command.&#x20;

However, it doesn't need to actually make the merge commit and tie those contexts together forever. It can simply do the merge, give you the result and keep track of what you do to it. This allows you to unmerge and remerge at will, as long as the branches merge cleanly.

The other interesting thing is that you cannot get two branches into a conflicting state while they are both applied, because all code in your working directory has to be owned by something. If you come across a part that overlaps, you have to choose which branch owns it or else unapply one of them.

## Merge Conflicts

You also cannot (currently) apply two virtual branches that conflict with each other. That means that the only way to merge two branches that have conflicts is to land one upstream in your base branch first, then integrate that so that it's your base and then pull those conflicting changes into your working directory and solve them.

In other words, branches can only conflict with upstream changes and you can only resolve one upstream conflicted branch at a time. This is something we will work on changing, but it is a limitation currently.

When you do have an upstream conflicting branch, you can choose to apply it, which will unapply all other branches and apply this one with conflict markers into your files. Your only branch lane will look something like this:

<div align="center">
  <figure>
    <img src="../../../.gitbook/assets/CleanShot 2023-07-24 at 15.38.24@2x.png" alt="" width="375">
    <figcaption>
      <p><i>A virtual branch in a conflicted state.</i></p>
    </figcaption>
  </figure>
</div>

While the branch is in this state, you cannot apply other branches or unapply this one. _(We are working on a way to abort this state, but right now you cannot)._

You need to resolve each issue, then hit the "Resolve" button next to each file once they are resolved. This is similar to running `git add` on resolved files in a conflicted merge state in stock Git.

Once all the files are marked as resolved, you need to commit the resolved merge. This will write a merge commit with both parents so that you have an updated merge base.

<div align="center">
  <figure>
    <img src="../../../.gitbook/assets/CleanShot 2023-07-24 at 16.56.25@2x.png" alt="" width="375">
    <figcaption>
      <p><i>Now you need to commit your resolution.</i></p>
    </figcaption
  ></figure>
</div>

Once the resolution is committed, you can unapply and apply branches again.
