## General questions

> What all is in the `.git` directory?

It contains:

*   "The index", a staging area (`.git/index`)
*   The object store (`.git/objects`).
    You can use `git cat-file` to examine objects.
    There are exactly 4 kinds of object in git:
    *   commits
    *   blobs, representing snapshots of files (?)
    *   trees, representing snapshots of directory trees
    *   tags, named pointers to commits.
        <http://www.gitguys.com/topics/git-object-tag/>
*   branch pointers (`.git/refs`)
*   logs for each ref and for HEAD (`.git/logs/refs` and `.git/logs/HEAD`)
*   `.git/HEAD` is whichever branch we've checked out,
    or just a revision if we're in detatched HEAD mode
*   You can also have `.git/{FETCH,ORIG,MERGE,CHERRY_PICK}_HEAD`.
*   You can also have notes (see `git notes`), stored in `.git/refs/notes/commits`.

Everything else is the working directory, and if *that* is a mess, that's on you.
git doesn't try to manage it.

> What intermediate states (like "merge in progress but user needs to
> resolve some conflicts") can a git repo be in?

???

> How do octopus merges work?

???

> What exactly is HEAD? Is it just a per-repository variable of type
> "branch name"?

Apart from detached HEAD state, yes, that's what it is.

In addition the whole history of HEAD is logged, by default
(this is called the reflog).

> Are there other things like HEAD?

In some states there are other "per-repository variables" like this.
They seem to be called "symbolic references", and they include
`FETCH_HEAD`, `ORIG_HEAD`, `MERGE_HEAD`, and `CHERRY_PICK_HEAD`.
I don't know if those have reflogs or not;
experiment suggests that `MERGE_HEAD` at least isn't logged.

There's a command, `git symbolic-ref`, which you can use to create a new
symbolic reference:

    $ git symbolic-ref -m "just for fun" BANANAS refs/heads/master
    $ cat .git/BANANAS
    ref: refs/heads/master
    $ git reflog BANANAS
    f4a0ce8 BANANAS@{0}: commit (initial): readme

This isn't a true reflog, though; see `man git-config` under `core.logAllRefUpdates`
which says only `HEAD` is logged by default.

`touch .git/logs/BANANAS` did not cause git to start logging changes to
`BANANAS`, for me. Anyway, I'm way off in the weeds.

> Why would anyone want to use `git symbolic-ref` to create a symbolic
> reference, anyway?

??? HMMM

> How is HEAD stored?

A file, `.git/HEAD`, contains something like `ref: refs/heads/master`.

`git show-ref` shows that we have a ref called `refs/heads/master`.
So HEAD points to that, and that points to a revision.

If we're in detached HEAD state, then `.git/HEAD` is just a revision number,
and each time you commit, git changes the file to the new commit's hash.

> If you `git add` a file to the index, a "blob" is created in the
> repository's store, right?

Yes. The tree is only created if you `git commit`.

> What would you use hooks for?

??? HMMM

> Is there a way to set up a .git/hooks hook other than hand-munging
> that directory?

???

> What does `git gc` do? Why doesn't it obliterate commits that are
> not reachable via commit back-pointers from any ref?

Everything that *has been* reachable from a reference during the past 2
weeks (by default) is retained. This is configurable (see `man
git-config` under `gc.expire`).

`git gc` apparently does other random and unspecified "housekeeping
tasks" besides collecting objects that are no longer reachable due to
rebases, branch deletion, and so on.

(I can't help imagining an actual, fairly naive mark and sweep
implemented somewhere under `git prune` or `git fsck`. But who knows,
maybe it is not-so-naive.)

> Is there a command to just list all objects?

??? HMMM

> What does `git remote -v` query to get the list of remotes?

According to strace, it queries config files, `.git/HEAD`, and files
under `.git/refs/heads`. This is weird because I would have expected it
to query `.git/refs/remotes/`. It doesn't.

I'm guessing `.git/config` is the usual source of truth...

???

> What does it mean to say that a file is "tracked"? (`git add FILE`
> causes git to remember that FILE is now "tracked".)

According to gitguys.com's glossary,
it literally just means that the file is in the index.

> What does `git tag -l` query to find all tags?

(presumably `.git/refs/tags`)

???

> How can I edit a *github* repository's upstream - or more generally,
> is it possible to edit a remote repository's remote URLs the way you
> can edit the local repository's remote URLs with `git remote set-url`?
> (Scott Burns asked about this on twitter.)

???

This stuff only seems to be stored in `.git/config` under
`[remote "origin"]`, so I don't immediately see a way to do it via
`git`. It may be accessible via some github API.

> What is `git pull --ff`?

???

> What is `.gitreview`?

???

> What happens if you `git add` a symlink?

???

> In a bare repository, how does `git` know that the directory is a git
> repository?  I thought that was determined by looking for `.git`. But
> a bare repository doesn't have one.

???

> What are configuration variables?

Knobs you can twiddle to tweak git's behavior in various ways.

git loads configuration variables from four files (see `man git-config`);
`.git/config` is the last and overrides the other three.

The variables in `.git/config` are used to configure remotes and branches
as well as git itself.

A plain `git init` on my machine produces this `.git/config` file:

    [core]
    	repositoryformatversion = 0
    	filemode = true
    	bare = false
    	logallrefupdates = true

Of course there are like a billion of these things, all optional. See `man git-config`.

> OK, so what are "git logical variables" (`git var`)?

???

> What happens if I type `git add .git`? Is that bad?

Nothing happens. git silently ignores this, I imagine because
this happens every time you do `git add .*`.

If you try something more specifically wrong,
like `git add .git/` or `git add .git/HEAD`,
you get an error message.

> How do credentials work? What are the guarantees? (For example, will a
> server detect/prevent author spoofing by someone who is authorized to
> push to that server?)

???


# Updating (push and fetch)

> How does `git pull` differ from `git pull origin master`?

???

> How does `git push` differ from `git push origin master`?

???

> What does `git pull` do, exactly?

??? HMMM


## References and reflogs

> What exactly is a ref? How are they stored? What are the rules
> regarding how they are updated on push/pull/clone?

???

> What is a "remote-tracking branch"?

I think this refers to exactly references of the form
`.git/refs/remotes/<remotename>/<branchname>`.

I suspect they figure in the algorithms for `git fetch` and `git push`.

> Can you `git checkout` a remote-tracking branch?

???

> What is this `origin/master` syntax, what is up with that? When would
> I want to use it?

OMG. The slash is literally a file path separator. This works because
the relevant reference is stored in `refs/remotes/origin/master`.

*(facepalm forever)*

> What exactly is the reflog?

It is a log that tells what HEAD (and each reference in `.git/refs`) pointed to in the past.

Each reference in `.git/refs/**` also has its own reflog (stored in `.git/logs/refs/**`).

It's also possible to configure a repository or maybe an individual
reference not to have a reflog.

> How is the reflog stored?

Reflogs are stored under `.git/logs`, and each one is just a text file, one line per entry.

`git checkout` adds an entry to `.git/logs/HEAD`;
`git commit` when you're on a branch adds an entry to `.git/logs/refs/heads/<branch>`;
and so on.

> Is the reflog shared when you push/pull/clone?

No. This is very much per-repo.


## The object store

> What is the format of the object store?

`.git/objects` seems to contain three interesting things:

*   Directories with two-hex-digit names, like `45` and `c8`.
    In these directories are files with 38-hex-digit names.
    Note 2 + 38 = 40 which is the number of hex digits in a full SHA-1 hash.

*   `.git/objects/pack` can contain `.idx` and `.pack` files and when I clone
    a largeish repository from github this is what I get.

*   `.git/objects/info` is an empty directory in every git repo I've got.

> `.git/objects/info` always seems to be empty. What's it for?

`git update-server-info` generates something in this directory.
That command claims to be for the benefit of clients of dumb servers; I
imagine this means servers that just serve the repository as a tree of
raw binary files.

???

> Is the data `.git/objects/pack` always just a compressed form of files
> that could be equivalently spelled out in an unpacked form, as
> `.git/objects/xx/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx` files?

Apart from commands specifically designed to expose the difference
(like `git count-objects`)
it looks like this is true.

See `man git-repack`, `man git-pack-objects`.

> Does github automatically pack repositories?

???

> How can I query the object store?

For a given SHA, you can `git cat-file -t OBJECT` to get an object's type;
or `git cat-file -p OBJECT` to pretty-print its contents;
or `git cat-file blob OBJECT` to dump raw bytes of a blob to stdout;
etc.

`git show OBJECT` is another nice way to show things.
That way if OBJECT is a commit, you get a patch (with colors, even).

If no such object exists, or if you ask for a blob but the object isn't a blob,
you get an ugly error message.


## The index

> What exactly is the index? How is it stored? Not as an object, right?

"The 'index' holds a snapshot of the content of the working tree, and it
is this snapshot that is taken as the contents of the next commit."

It's not an object. It's stored in `.git/index` and it's really just a
mapping `{FILENAME: SHA}` with the SHA pointing to a blob object already
stored in the object store.

> Is there a command to just dump the contents of the index?

`git ls-files --stage` seems to do this. But this raises another question:

> What is a "stage number"? (`git ls-files --stage` shows this for each
> file in the index.)

It's used during merges (see the section below on "conflicts").

> What's the format of the index?

???

> How does `git commit` know the parents of the new commit? Are they
> stored in the index? (This is really about how intermediate
> merge-states are represented on disk.)

???

> Can the index store an empty directory?

???


## Branches and merging

(Note there's a separate section about conflicts below.)

> `git init` initializes `.git/HEAD` to `ref: refs/heads/master`
> but does not create a `master` branch!
> What commands fail on a brand new repository because of this silliness?
> (I'm sure I've seen this at least two or three times.)

One place where this is special-cased, at least, is `git diff --staged`.
The manual page has this bizarre sentence: "If HEAD does not exist
(e.g. unborn branches) and <commit> is not given, it shows all staged
changes."

???

> What does `git merge BRANCH` do to the branch?
> Is it removed? Changed to point to the current branch?

Neither. The branch being merged is left completely unchanged.

Think about it - suppose you have a branch called "topic" which is the
current branch, and then you do `git merge master`. Obviously you don't
want "master" to go away.

> Is there a parallel between command line syntax for `git rebase` and
> `git merge`?

???

> What's the difference between refs and branches? Some refs are to do
> with remotes, but consider `refs/heads/master`. Is the branch `master`
> anything else, besides that ref?

Branches are additionally configurable in `.git/config`, so each branch
can (for example) be associated with a default remote
`branch.<branchname>.remote`, a setting other refs can't have.

I don't know of anything else.

???

> Can I `git log` branches I'm not on?

Yes, just `git log branch`. Also `man gitrevisions` has a whole section
on how to specify a set of commits for `git log` and other similar
commands ("SPECIFYING RANGES").

(It's like `hg help revsets` except not as capable and fantastically cryptic.)

> Can I `git log --graph` all branches?

Yes, `git log --graph --all`.
(Note: this shows all commits reachable from any reference found under `.git/refs`,
not all commits in the object store.)

You can also do `git log --graph master branch1 branch2` to see multiple
specific branches at once.


## Conflicts

> When git detects conflicts, is there a way to ask it to open a merge
> tool automatically?

???

> When git detects conflicts, are the conflict-markers it inserts into
> your files useful? Common ancestor?

Yes.

> A dag is not necessarily a lattice. If two revisions being merged have
> more than one latest common ancestor, what do the conflict-markers
> look like? For that matter, does git have to choose an ancestor arbitrarily
> in order to even *attempt* the merge? (How else would it work?)

??? HMMM

> After git detects conflicts, how is the resulting state represented in
> `.git`? What commits are the parent of the index in this state?

In this state there is a `.git/HEAD` and a `.git/MERGE_HEAD`.
These will become the parents of the new commit.

Conflicts are indicated using the stage number.
I read about this in `man gitrevisions`:
"During a merge, stage 1 is the common ancestor, stage 2 is the target
branch's version [...], and stage 3 is the version from the branch which
is being merged."

So you'll have multiple entries in the index for the same file.

Files that were successfully merged only have one entry, and it's stage 0.

`git add FILE` replaces all entries for FILE with a single stage 0 entry:

    $ git ls-files --stage
    100644 b023018cabc396e7692c70bbf5784a93d3f738ab 0	goodbye.txt
    100644 ce013625030ba8dba906f756967f9e9ca394464a 1	hello.txt
    100644 a7691826998f893ba7d6ffeff52c6c820a70cfbd 2	hello.txt
    100644 d0e08a8d7b83fce6094afb1e4eb78ef49e3ed41d 3	hello.txt
    $ vi hello.txt  # fix conflicts
    $ git add hello.txt
    $ git ls-files --stage
    100644 b023018cabc396e7692c70bbf5784a93d3f738ab 0	goodbye.txt
    100644 40491b3ad4f6a554851bdbbe008160ef839a9c3a 0	hello.txt

> After an operation like a rebase stops with conflicts,
> how does `git rebase --continue` (or whatever) know what to do?

???

> After something like a rebase stops with conflicts, if you use `git
> add FILE` to resolve each conflict, is there anything stopping you
> from then doing `git commit`? Wouldn't that be bad?

???

> Can you git-checkout or git-pull with outstanding changes? What
> happens to those changes?  If they are just re-applied on top of the
> new checkout, how are conflicts handled?

You can `git checkout` with outstanding changes, and they are re-applied.
Don't know about conflicts yet.

???

> Suppose I do `git merge` and there are edit-edit conflicts.
> Can I commit without addresing the conflicts?

Nope:

    $ git commit -m 'merge, ignoring conflicts'
    U	hi.txt
    error: commit is not possible because you have unmerged files.
    hint: Fix them up in the work tree, and then use 'git add/rm <file>'
    hint: as appropriate to mark resolution and make a commit.
    fatal: Exiting because of an unresolved conflict.

> What commands detect this "uncommitted merge" state?

`git status` does it.
If there are unresolved conflicts in the index, you get this:

    $ git status
    On branch master
    You have unmerged paths.
      (fix conflicts and run "git commit")

    Unmerged paths:
      (use "git add <file>..." to mark resolution)

        both modified:   hi.txt

    no changes added to commit (use "git add" and/or "git commit -a")

After you fix the conflicts and do `git add hi.txt`, you'll see:

    $ git status
    On branch master
    All conflicts fixed but you are still merging.
      (use "git commit" to conclude merge)

    Changes to be committed:

        modified:   hi.txt

> When merging, the output of `git status` says what branch I'm "On".
> I've read that this is usually, but not always, the first parent of
> the merge. Is that true? Where did I read that? If the two *are*
> somehow different, what does `git status` say, and if it doesn't
> specify both, how can I recover the other information?

???

> When merging, `.git/HEAD` points to a branch, and `.git/MERGE_HEAD`
> points to a specific revision. In this state, is it possible to use
> `git reset` to change the branch pointer? Does that affect the
> parents of the merge commit I'm working on?

???

> When merging, can you `git pull`? What happens?

???

> When merging, the output of `git status` says what branch I'm on, but
> not what branch I'm merging. How can I find out?

I don't know an obvious way yet.

You can see what commits you're merging by doing `git log MERGE_HEAD..HEAD`.

`git rev-parse MERGE_HEAD` gives the hash of the commit you're merging,
but I don't know how to resolve that to a branch name.

???

> What is up with `git diff` in this state?

(ha ha it's so awesome)

???

> Is the effect of git-mv stored specially in the repo? Or does it just
> change entries in the index?

It just changes entries in the index.
Copying and removing the file manually has the same effect:

    $ cp hi.txt greeting.txt
    $ git add greeting.txt
    $ git rm hi.txt
    rm 'hi.txt'
    $ git status
    On branch master
    Changes to be committed:
      (use "git reset HEAD <file>..." to unstage)

        renamed:    hi.txt -> greeting.txt

`git diff --cached` in this situation shows one file being removed and another being added,
unless you've done `git config diff.renames true`.

> How is an edit-move conflict handled?

(prediction) git doesn't notice the conflict. The edit is clobbered.

???

> Is there a `git cp`? If so, how is an edit-copy merge conflict handled?

There isn't.

(You *can* do `git config diff.renames copy` to enable
`git diff -C` to detect and show that a file was created by copying
another file. Not really worth it.)

> How is an edit-delete conflit handled?

???

> Move-delete?

???

> Move-move?

???

> Delete-delete?

(prediction) this is not considered a conflict.

???


## Undoing

> How can I undo `git commit`?

Solution: `git reset --soft HEAD^`

> How do I undo `git rm FILE/DIR`?

Solution: `git checkout FILE/DIR`

> How do I undo `git mv FILE1 FILE2`?

Solution: `git mv FILE2 FILE1`

You could also do it with `git reset` or `git checkout`.
Those are complicated commands, but maybe it's useful to show:

1.   Recover the old file: `git checkout HEAD FILE1`

2.   Get rid of the new one:  `git rm -f FILE2`
