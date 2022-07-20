# Git submodules

*   **What is a submodule?**

    A git repository embedded in another repository.

*   **What is a superproject?**

    The enclosing git repository that contains a submodule.

*   **Is all this as ad hoc as it looks, or is there a method to the madness?**

    Read on.


## Basic tasks

*   **When I first clone a repository that contains submodules (the
    superproject), how do I tell git to also populate all submodules to the
    recorded revisions?**

    `git submodule update`

*   **When I pull from the superproject, how do I tell git to also update all
    submodules to the recorded revisions, fetching if necessary?**

    `git submodule update`

*   **Is it dangerous to do that if I have uncommitted changes in the submodule
    repo / if I have done `git checkout $other_branch` in the submodule repo?**

    (Unknown.)

*   **Does `git status` in an enclosing repository tell me what I've changed in
    submodules?**

    Not in detail, but it'll say:

        Changes not staged for commit:
        ...
                modified:   path/to/my/submodule (modified content)

*   **Does `git commit` in either a submodule or the enclosing
    repository do anything to the other?**

    No. Committing in a submodule (or pulling new changes into a
    submodule) will result in the submodule being "ahead" of the
    enclosing repository's expectations; `git status` in the enclosing
    repo will then say:

        Changes not staged for commit:
        ...
                modified:   path/to/my/submodule (new commits)

*   **OK then, how do I update the enclosing repository so that it
    understands I made changes in the submodule and I want to use
    them?**

    ```
    git add path/to/my/submodule
    ```

    This stages the change (to the superproject's index).
    You still have to commit that change, like any other.

*   **What if the change happened in a separate remote (e.g. a separate GitHub
    repo)? How do I get that change into my superproject? That is, I want to
    change the superproject to point to a newer revision of one of its
    submodules.**

    ```
    cd path/to/my/submodule
    git pull                   (or `git checkout foo` or whatever)
    cd -

    git add path/to/my/submodule
    git commit
    ```

    It's a lot like any other change you want to make in a git repo. You change
    the thing in your working directory to have the desired state (in this
    case, updating the submodule's `HEAD` to point to the desired commit) and
    then you stage and commit it.



## How it works

*   **Where is the recorded revision stored in the enclosing repository?**

    `man gitsubmodules` says "the superproject tracks the submodule via
    a **gitlink** entry in the tree at `path/to/submodule`" and "The
    **gitlink** entry contains the object name of the commit that the
    superproject expects [...]."

*   **Is a "gitlink entry" an entry in a tree object?**

    Yes. It's an entry in the containing tree in the parent
    repository. That's how submodule entries get included in commits,
    how they're fetched and pushed across repositories, and so on.

*   **But doesn't that contradict the statement that there are only 4
    object types in Git?**

    No, there's no such thing as a "gitlink object". A "gitlink entry"
    is just a line item in a tree object.

    The entry contains a SHA, but it's the SHA of a commit in the
    submodule repository; there won't be anything in the parent repo
    with that SHA.

*   **How can I read a "gitlink entry"?**

    `git ls-tree HEAD <dir>`, where `<dir>` is the submodule
    directory, shows it.

    (Or you can just `git submodule status` to see all of them.)

*   **Is there any other documentation on "gitlink entries"?**

    Not really. The man pages mention that they exist, and so does
    Documentation/technical/index-format.txt.
    That's about it.

*   **How did you figure that out?**

    Cloned the Git source repository and did some grepping in the
    Documentation directory.

