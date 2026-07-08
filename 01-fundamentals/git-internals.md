\# Git Internals



\## What it is

Git isn't really a version control system — it's a \*\*content-addressable

filesystem\*\* with a version control UI bolted on top. Once you see what's

actually in `.git/`, every git command starts making sense.



\## The mental model

```mermaid

flowchart LR

&#x20;   WD\[Working Directory] -->|git add| Index\[Staging Area / Index]

&#x20;   Index -->|git commit| Repo\[.git / Local Repo]

&#x20;   Repo -->|git push| Remote\[Remote Repo]

&#x20;   Remote -->|git fetch| Repo

&#x20;   Repo -->|git checkout| WD

```



Three areas locally: \*\*working directory\*\* (files you edit), \*\*index/staging

area\*\* (what will go in the next commit), \*\*repo\*\* (`.git/`, the object store).



\## The four object types

Everything in git is one of four objects, each identified by the SHA-1 hash of its contents:



\- \*\*blob\*\* — the content of a file. Just bytes. No filename.

\- \*\*tree\*\* — a directory listing. Maps filenames to blobs and other trees.

\- \*\*commit\*\* — a snapshot: points to one tree, plus author, message, and parent commit(s).

\- \*\*tag\*\* — a named pointer to a commit (annotated tags carry a message too).



```mermaid

flowchart TB

&#x20;   Commit\[commit abc123] --> Tree\[tree def456]

&#x20;   Tree --> Blob1\[blob: README.md content]

&#x20;   Tree --> Blob2\[blob: main.py content]

&#x20;   Tree --> SubTree\[tree: src/]

&#x20;   SubTree --> Blob3\[blob: utils.py content]

&#x20;   Commit --> Parent\[parent commit xyz789]

```



\## What a commit actually is

A commit is \*\*not a diff\*\*. It's a full snapshot pointer:

\- A pointer to a tree (the full state of the project at that moment)

\- Parent commit hash(es)

\- Author, committer, timestamp, message



Diffs are computed on the fly by comparing two trees.



\## What lives in .git/

\- `HEAD` — a pointer to the current branch (e.g. `ref: refs/heads/main`).

\- `refs/heads/` — branch pointers, each just a file containing a commit hash.

\- `refs/tags/` — tag pointers.

\- `objects/` — the object store (blobs, trees, commits, tags).

\- `index` — the staging area (binary file).

\- `config` — repo-local config.



Branches are literally just files with a commit hash in them. That's it.



\## The three-way merge

When you merge, git looks at three commits:

\- Your branch tip

\- Their branch tip

\- The common ancestor



It combines changes from both sides relative to the ancestor. Conflicts happen

when the same lines were changed differently on both sides.



\## Rebase vs merge

\- \*\*Merge\*\* — creates a new commit joining two histories. Preserves what actually happened.

\- \*\*Rebase\*\* — replays your commits on top of another branch. Rewrites history for a linear log.



Rule of thumb: \*\*rebase local, merge shared.\*\* Never rebase a branch someone else has pulled.



\## Commands worth understanding, not memorizing

\- `git log --oneline --graph --all` — see the DAG of history

\- `git reflog` — every place HEAD has been (your safety net; recovers "lost" commits)

\- `git reset --soft HEAD\~1` — uncommit but keep changes staged

\- `git reset --hard HEAD\~1` — nuke last commit and its changes (dangerous)

\- `git cherry-pick <sha>` — apply one commit onto current branch

\- `git stash` — shelve uncommitted changes

\- `git bisect` — binary-search history to find the commit that broke something

\- `git rev-parse HEAD` — resolve any ref to a full SHA



\## Common gotchas

\- \*\*`git pull` is `fetch + merge`\*\* — sometimes creates surprise merge commits. Prefer `git pull --rebase` or `fetch` + explicit merge.

\- \*\*Force-push (`git push -f`) rewrites remote history.\*\* Coworkers will hate you. Use `--force-with-lease` if you must.

\- \*\*Detached HEAD\*\* — you're on a commit, not a branch. Any commits you make are unreachable unless you branch first.

\- \*\*`.gitignore` doesn't untrack already-tracked files.\*\* Use `git rm --cached`.

\- \*\*Secrets committed once are committed forever\*\* — even if you delete them in a later commit. Use `git filter-repo` (or BFG) to purge from history.



\## What I want to try

\- Open `.git/objects/` in a real repo and use `git cat-file -p <sha>` to inspect a blob, tree, and commit by hand.

\- Recover a "lost" commit using `git reflog`.

\- Use `git bisect` to find a commit that breaks a test.

\- Rewrite a messy branch with `git rebase -i` (squash, reword, drop commits).



\## Sources

\- "Pro Git" — Chacon \& Straub (free at git-scm.com/book)

\- Julia Evans's git zines (wizardzines.com)

\- The Git Parable — Tom Preston-Werner



