---
description: How to dig around our internal data to find (nearly) anything
---

# ⛏️ Recovering Stuff

GitButler saves data in a few different ways. As we're still in beta, sometimes things might break and it may look like you've lost work, but you almost certainly haven't. We're pretty good about saving stuff a lot. Here's how to recover almost anything you had in your working directory or virtual branches.

## GitButler References

If everything crashes or the UI isn't working at all, you may be surprised to know that even though your virtual branches don't show up in a normal `git branch` output, we _do_ actually constantly write them out as Git references (just not in `refs/heads`).

```
❯ git for-each-ref | grep gitbutler
e63b3bac82835dc17083a785d25db8b4b46744b9 commit	refs/gitbutler/add-can-create-method-to-notebook
98ef3cd6eea14ee4159a600e448271c0d777efe2 commit	refs/gitbutler/add-conditional-blocks-for-image-and-video
c7e27b9f99f25160a4d5f07d5972c217bdd44319 commit	refs/gitbutler/add-database-schema-conversion-script
4afdfed6c14b57491a9d295c31613fd79b92f63a commit	refs/gitbutler/add-gems-for-test-group
```

These references are just like git branches - they point to a commit that has the latest version of your branch. You can create other git branches off of them, you can push them to GitHub, etc.

You will have one for each virtual branch (applied or unapplied) that you've created (that you haven't deleted).

If you've committed everything on a virtual branch, the reference will just point to the latest commit. If you have work in progress on the branch, it will point to a WIP commit that includes those changes.

So for example, if I have the following two virtual branches, one fully committed and one with work pending:

<div align="center">

<figure><img src="../.gitbook/assets/CleanShot 2024-02-23 at 10.30.27@2x.png" alt=""><figcaption></figcaption></figure>

</div>

I can view the git branches like this:

```
❯ git show gitbutler/Convert-tables-to-utf8mb4
commit 841e4db701ca41206c03f1f4fe345f7e27d05eab
Author: Scott Chacon <schacon@gmail.com>
Date:   Fri Feb 23 10:30:17 2024 +0100

    my latest commit
    
❯ git show gitbutler/Add-database-schema-conversion-script
commit d95e7f4da1611ea6bb8a80da06e66ca923fbff55
Author: GitButler <gitbutler@gitbutler.com>
Date:   Fri Feb 23 10:30:18 2024 +0100

    GitButler WIP Commit
    
    This is a WIP commit for the virtual branch 'Add database schema conversion script'
    
    This commit is used to store the state of the virtual branch
    while you are working on it. It is not meant to be used for
    anything else.
```

See how the `Add-database-schema-conversion-script` reference points to a "WIP commit"? The tree of that commit has all those changed files in it as though we had committed them.

If you don't want to search through all your refs with `for-each-refs`, you can also just run a normal `git log` command and we'll show you what references we've written and which modified files are in each one:

<pre><code><strong>❯ git log
</strong><strong>commit 2d8afe0ea811b5f24b9a6f84f6d024bb323a2db5 (HEAD -> gitbutler/integration)
</strong>Author: GitButler &#x3C;gitbutler@gitbutler.com>
Date:   Fri Feb 23 10:30:18 2024 +0100

    GitButler Integration Commit
    
    This is an integration commit for the virtual branches that GitButler is tracking.
    
    Due to GitButler managing multiple virtual branches, you cannot switch back and
    forth between git branches and virtual branches easily.
    
    If you switch to another branch, GitButler will need to be reinitialized.
    If you commit on this branch, GitButler will throw it away.
    
    Here are the branches that are currently applied:
     - Add database schema conversion script (refs/gitbutler/Add-database-schema-conversion-script)
       - butler/Gemfile
       - butler/README.md
       - butler/db/schema.rb
       - butler/db/migrate/20240209144600_change_mysql_charset.rb
       - .pscale.yml
     - Convert tables to utf8mb4 (refs/gitbutler/Convert-tables-to-utf8mb4)
       branch head: 841e4db701ca41206c03f1f4fe345f7e27d05eab
       - butler/create_column_conversions.rb
    
    Your previous branch was: refs/heads/sc-branch-comments
    
    The sha for that commit was: 5e16e99667db9d26f78110df807853a896120ff3
    
    For more information about what we're doing here, check out our docs:
    https://docs.gitbutler.com/features/virtual-branches/integration-branch
</code></pre>

You can see the two `gitbutler` refs under the "Here are the branches that are currently applied" section.

Again, these are real git refs, just not under `refs/heads` so that we don't pollute your `git branch` output. But if GitButler crashes at some point, you can still push them to GitHub or whatever you want. Here is an example pushing my virtual branch to a GitHub branch called `convert-tables`:

```
❯ git push origin refs/gitbutler/Convert-tables-to-utf8mb4:refs/heads/convert-tables
Enumerating objects: 6, done.
Counting objects: 100% (6/6), done.
Delta compression using up to 10 threads
Compressing objects: 100% (4/4), done.
Writing objects: 100% (4/4), 474 bytes | 474.00 KiB/s, done.
Total 4 (delta 2), reused 1 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (2/2), completed with 2 local objects.
remote: 
remote: Create a pull request for 'convert-tables' on GitHub by visiting:
remote:      https://github.com/gitbutlerapp/web/pull/new/convert-tables
remote: 
To github.com:gitbutlerapp/web.git
 * [new branch]        refs/gitbutler/Convert-tables-to-utf8mb4 -> convert-tables

```

## GitButler Operations Log

Ok, let's say that your work was not in one of those refs for some reason. Maybe you hit some weird bug and it completely changed everything in a way where now you're sitting on the couch in the dark with a glass of whisky, slowly mumbling the word "GitButler..." and plotting your revenge.

Most of the time, we'll have whatever you're looking for in our operations log.

The easiest way to access this is to use the built in Project History UI: [timeline.md](../features/timeline.md "mention")

However, let's dig into how this works, just in case you want to check it out yourself.

Every time that GitButler does some possibly data-changing operation, we store a snapshot of your project state in our operations log before the operation happens so you can undo it if you want to. This is stored as a Git commit history that is parallel to your projects (ie, no common parents).

You can inspect this by looking at the `.git/gitbutler/operations-log.toml`file.

```
❯ tree .git/gitbutler
.git/gitbutler
├── operations-log.toml
└── virtual_branches.toml

1 directory, 2 files

❯ cat .git/gitbutler/operations-log.toml
head_sha = "16e47cb1d091ca9dd44327fef2f5305b09403a95"

[modified_at]
secs_since_epoch = 1717663406
nanos_since_epoch = 560458000
```

If we look at this commit, we can see the history of all of the project history snapshots that GitButler is keeping:

```
❯ git log 16e47cb1d091ca9dd44327fef2f5305b09403a9 -2
commit 16e47cb1d091ca9dd44327fef2f5305b09403a95
Author: GitButler <gitbutler@gitbutler.com>
Date:   Thu Jun 6 10:43:26 2024 +0200

    CreateBranch

    Version: 1
    Operation: CreateBranch
    name: Virtual branch

commit 2c95aa06d76b3230f1a51d9f89a211770d93ae51
Author: GitButler <gitbutler@gitbutler.com>
Date:   Thu Jun 6 10:28:30 2024 +0200

    UpdateWorkspaceBase

    Version: 1
    Operation: UpdateWorkspaceBase
    
```

You can see that before creating a branch or updating our workspace with upstream work, we're recording the state of our project so we have an undo point. So what data are we keeping here in addition to this trailer information?

Let's look at the tree of one of these commits:

```
❯ git cat-file -p 16e47cb1d091ca9dd44327fef2f5305b09403a95^{tree}
040000 tree 4b825dc642cb6eb9a060e54bf8d69288fbee4904    conflicts
040000 tree b283f22f5abf4ed9c612c1b5b3b9a98ec25474b0    index
040000 tree b283f22f5abf4ed9c612c1b5b3b9a98ec25474b0    target_tree
100644 blob d839dca7e14f5833ad737b4adbf337bd20489927    virtual_branches.toml
040000 tree a0821552c0e7d5defe369d577af5e3a87b442469    virtual_branches
040000 tree b283f22f5abf4ed9c612c1b5b3b9a98ec25474b0    workdir
```

So, it's not a snapshot of your working directory alone, but it includes a bunch of other meta information.

The state of your working directory is indeed stored in the `workdir` tree:

```
❯ git cat-file -p 16e47cb1d091ca9dd44327fef2f5305b09403a95^{tree}:workdir
040000 tree 6676978c86f95a96a59d712c2a4432a7df39f4c9    .cargo
100644 blob 2cb803c6460cd42bce8611d101e1e546b3f57ada    .editorconfig
100644 blob 50ca329f24bb341a39f5c9cbc3fd4a083c162f9e    .gitattributes
040000 tree 717cae25b7e27bc7ddbb3e5e99b0c2889ffb6f9b    .github
100644 blob 3863377de3e79a055dbe5778cfa3c64f34db1501    .gitignore
100644 blob 0f391aedf3b8621560f1f4c950b68710e0cff825    CODE_OF_CONDUCT.md
100644 blob be2128195a883e39bb59931d30a8ce63c4fd9b0f    CONTRIBUTING.md
100644 blob 73cea1430a8f83a1580684fc97975dabc7b4293e    Cargo.lock
100644 blob eccda7ef5fc62b6e862eee89c3dfb22216735620    Cargo.toml
100644 blob 5fc97481cac5ccbf9f6b6f7b646820553f6d865c    DEVELOPMENT.md
100644 blob 11d87d8a56a7482062c1fa377a1f7f006c3e721b    LICENSE.md
100644 blob 66db4fe325336f213ac34008de7cff129a654f53    README.md
100644 blob 994f624c90283bb1df2b9b8a3924e9b8aa8a13e1    SECURITY.md
040000 tree f88da8e9695e023c330a3fd2def9da0d4561fe22    app
040000 tree 8238a9217ea4884c20c599eb5007d1928ce9b13e    crates
100644 blob 147e034ca02262f01bc07a1bf8cc8c70d64edd10    flake.lock
100644 blob d3add173b05930b82363b043eac2370b661d1c7f    flake.nix
100644 blob 672fcfcf3486ae15e5ee3470f4981130d6c700d8    package.json
100644 blob c3cf5173d6baa349a67032ef0083b71d0d95606e    pnpm-lock.yaml
100644 blob 5dddf2919f467ba1bdf22376235f1c8a55173c06    pnpm-workspace.yaml
100644 blob 0de364ea8e94ce1fdf711be2993e9cff47e0e746    rust-toolchain.toml
100644 blob 830b80e861ab3b8d8948268a1a58a5209fda1cfa    rustfmt-nightly.toml
040000 tree 61bceaa29b69e258fb61b723d8b32cbf9f011201    scripts
```

So you can easily recover file contents from any of these snapshots if needed:

```
❯ git cat-file -p 16e47cb1^{tree}:workdir/.github/dependabot.yml
version: 2

updates:
  - package-ecosystem: "github-actions"
    directory: "/"
    schedule:
      interval: "weekly"
```

Beyond that, we also store the state of your index in `index` as a tree, the state of what your target (trunk) tree looked like in `target_tree`, conflicts if you were in a conflicted state in `conflicts` and the state of your virtual branches.

The virtual branches toml file has the metadata:

```
❯ git cat-file -p 16e47cb1d091ca9dd44^{tree}:virtual_branches.toml
[default_target]
branchName = "master"
remoteName = "origin"
remoteUrl = "git@github.com:gitbutlerapp/gitbutler.git"
sha = "e00e54af7f903ef84600079a45490a7f07e4702e"
pushRemoteName = "origin"

[branch_targets]

[branches.09ef54c4-1081-4a52-8182-a5ec725016b6]
id = "09ef54c4-1081-4a52-8182-a5ec725016b6"
name = "commit signing settings"
notes = ""
applied = false
upstream = "refs/remotes/origin/commit-signing-settings"
upstream_head = "b60a66452dfecef74103346af6a3291ad677d246"
created_timestamp_ms = "1717489406268"
updated_timestamp_ms = "1717489406268"
tree = "b28e7eefdd7b6f36456516b696146a2ea7638ca4"
head = "b60a66452dfecef74103346af6a3291ad677d246"
ownership = ""
order = 4
selected_for_changes = 1717489406268
```

The `virtual_branches` tree has the actual contents of those computed branches in case we need to recreate them:

```
── virtual_branches
   └── [branch-id]
       ├── commit-message.txt
       └── tree (subtree)
   └── [branch-id]
       ├── commit-message.txt
       └── tree (subtree)
```

This allows you to get contents of any file in any of your virtual branch states as well.
