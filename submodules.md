*   **What is a submodule?**

    A git repository embedded in another repository.

*   **Is all this as ad hoc as it looks, or is there a method to the madness?**

    Unknown.

*   **When I first clone a repository, how do I tell git to also populate all submodules to the recorded revisions?**

    I think `git submodule update` is all you need.

*   **When I pull from a repository, how do I tell git to also update all submodules to the recorded revisions, fetching if necessary?**

    Same deal, `git submodule update`.

*   **Does `git status` in an enclosing repository tell me what I've changed in submodules?**

*   **Does `git commit` in either a submodule or the enclosing repository do anything to the other?**

    No.

*   **OK then, how do I update the enclosing repository so that it understands I made changes in the submodule and I want to use them?**


*   **Where is the recorded revision stored in the enclosing repository?**

    `man gitsubmodules` says "the superproject tracks the submodule via
    a **gitlink** entry in the tree at `path/to/submodule`" and "The
    **gitlink** entry contains the object name of the commit that the
    superproject expects [...]."

*   **Is a "gitlink entry" an entry in a tree object?**

    Yes. It's an entry in the containing tree in the parent
    repository. That's how submodule entries get included in commits,
    how they're fetched and pushed across repositories, and so on.

*   **But doesn't that contradict the statement that there are only 4 object types in Git?**

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

