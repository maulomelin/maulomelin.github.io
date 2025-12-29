
# How I Organize Source Code on My Computer

The way I organize source code on my computer has changed over time. It has been influenced by guidelines I have followed at large corporations, standards I've helped establish within growing teams, and simple pragmatic rules I follow on personal projects.

At a very high level, this setup is a part of how I organize my data globally. I designed it around three requirements:

1. **Independence.** Maintain a structure that is independent of folders managed by the OS (such as `Documents/` or `Library/`) to ensure it works on macOS, Windows, or Linux.

1. **Portability.** Create a walled garden workspace that is easy to set up or tear down, and work identically across machines.

1. **Efficiency.** Establish path predictability so that scripts and workflows remain consistent across environments.

## Directory Structure

This is my current setup on macOS, in path form:

```sh
# Workspace root folder.
~/_dev/

# Active repositories.
~/_dev/repos/{remote}/{account}/{repo}/

# Repository snapshots.
~/_dev/snapshots/{account}--{repo}--{description}.(zip,tar.gz)

# Experiments.
~/_dev/scratch/
```

And in tree form:

```sh
~/_dev/                         # Workspace root folder.
├── repos/                      # Active repositories.
│   └── {remote}/
│       └── {account}/
│           └── {repo}/
│               └── ...
├── snapshots/                  # Repository snapshots.
│   └── {account}--{repo}--{description}.(zip,tar.gz)
└── scratch/                    # Experiments.
    └── ...
```

## Folder Contents

### Everything is under `~/_dev/`

* All development-related work is consolidated under a `_dev/` directory. This keeps things scoped, centralized, and easy to remember.

* The `_dev/` directory is placed at the root of the home folder. This gives me a common location to set up and work from on any machine I use.

    ```sh
    # On macOS.
    ~/_dev/     # Shortcut for /Users/{username}/_dev/

    # On Windows.
    C:\Users\{username}\_dev\

    # On Linux.
    ~/_dev/     # Shortcut for /home/{username}/_dev/
    ```

* I like using the name "dev" for my workspace (instead of "src", "code", or other variants) because projects contain more than just code (e.g., spec documents, databases, or other files), and "dev" is sufficiently generic to indicate a project workspace.

* Adding the prefix `_` to any folder name (e.g., `_dev/`) does two important things for me:

    1. It tells me that this folder and its contents are managed by me (i.e., not the OS), and

    1. It sorts the folder at the top of the list in any file manager, so I can quickly tell which folders I've set up on any given machine.

* WARNING: Never configure the `_dev/` directory with any cloud-syncing service (including iCloud, OneDrive, Google Drive, Dropbox, etc.). This prevents potential repository corruption caused by clashes between cloud-sync services and Git.

### Active repos are in `~/_dev/repos/`

* Every folder in `~/_dev/repos/` is an active Git repository.

* Active repos are managed as usual: local updates are periodically pushed to its remote, and upstream updates are merged with local as needed. This convention allows me to easily replicate the setup on any machine and continue working regardless of what computer I'm on.

* Every repo is locally cloned to the directory `~/_dev/repos/{remote}/{account}/{repo}/`, which follows the pattern used by remotes.

    This works for several reasons:

    1. The use of the `{remote}` > `{account}` > `{repo}` taxonomy avoids name collisions.

    1. The directory path is consistent with a remote's URL and tells me where a source repo is at.

    1. I don't have to think about what to name or how to organize projects! The organization of my local projects is already consistent with those on a remote. The fact the URLs are unique online means they'll be unique locally.

    1. As a bonus, this simplifies some automation tasks. Since remote URLs can be generated from file paths, and vice-versa, I can write scripts that work on any machine I use.

* This is an example of what my `repos/` folder looks like, down to the `{repo}` level, in tree form:

    ```sh
    ~/_dev/
    └── repos/
        └── github.com/
            ├── maulomelin/
            │   ├── gists/
            │   ├── maulomelin.github.io/
            │   └── shell-scripts/
            ├── pages-themes/
            │   └── primer/
            ├── tailwindlabs/
            │   └── tailwindcss/
            └── ...
    ```

### Repo Snapshots are in `~/_dev/snapshots/`

* Every file in `~/_dev/snapshots/` is a clean snapshot of a repository.

    A clean snapshot occurs when a package is created using the `git archive` command. This creates a version of a repo at a specific moment without any Git metadata (e.g., no `.git` folder or commit history). These snapshots are generated for various reasons, such as preparing a package for a tagged release, creating an artifact for deployment, or preserving a golden master before a repository is decommissioned.

* I like using the name "snapshots" because the [Git docs refers to these as "snapshots"](https://git-scm.com/book/en/v2/Getting-Started-What-is-Git%3F#_snapshots_not_differences), and it feels more descriptive than "archives" or "bundles" or other variants I explored.

* Anything inside the `~/_dev/snapshots/` folder is disposable.

    Snapshots are simply archive files. They can't be cloned because they are not proper repositories. For repository snapshots I keep, my long-term storage vault is a cloud-synced directory `{cloud_home}/_home/_dev/snapshots/`.

    When setting up a new machine, I simply copy any necessary snapshots from my cloud-based vault into my local `~/_dev/snapshots/` folder for reference or experimentation. For this reason, the `~/_dev/snapshots/` folder is disposable. All my snapshot masters are backed up in the cloud. If my system crashes, I can easily replicate them on any new computer from my vault.

* The schema I follow for naming snapshots is based on the file path for active repos:

    ```sh
    [{remote}--]{account}--{repo}--{description}.(zip,tar.gz)
    ```

    where:

    * The `{account}` and `{repo}` fields are the same ones used at the remote.

    * The `{description}` field can be a tag name, the date when the `git archive` command was executed, or a short label describing the snapshot.

    * The `{remote}--` prefix is optional. In practice, I do not store enough snapshots to need it to avoid filename collisions, and they would all would start with a redundant `github.com--`. However, it's in the schema if I ever need to use it.

* This is an example of what my `snapshots/` folder looks like, in tree form:

    ```sh
    ~/_dev/
    └── snapshots/
        ├── maulomelin--searchphp--2006-10-25.zip
        ├── maulomelin--sudoku-solver--v1.0.1.tar.gz
        ├── pages-themes--primer--v0.6.0.tar.gz
        ├── pages-themes--primer--v0.6.0.zip
        └── ...
    ```

* Notes:

    * Repo snapshots in the `snapshots/` folder are distinct from archived repos on a remote. When a project is archived on GitHub, for example, the repo is marked as read-only but remains available for reference: you can still `git clone` that repo to your local machine. Repos I can clone from a remote, whether they're archived in the cloud or not, are kept under `~/_dev/repos/`.

    * A package generated by the `git archive` command is a single file that excludes all Git metadata. Not to be confused with the package generated by the `git bundle` command which is a single file that DOES include all Git metadata. The latter can be used to create a new local active repository using the `git clone` command.

    * How do I handle a local Git repo with multiple remotes? I could clone multiple remote repos and ensure I push/sync periodically across them, but I prefer to simply clone under the primary `{remote}/{account}/{repo}/` folder and set up multiple remotes on it.

    * I generate both `.zip` files and `.tar.gz` tarballs for my snapshots to mirror what GitHub does. GitHub releases are based on Git tags, which mark a specific point in a repository's history. When GitHub creates a release, it automatically generates these two formats for compatibility across OSs. I simply decided to replicate what GitHub does for any snapshots I generate manually:

        ```sh
        # Create a zip file off the latest commit on the current branch.
        $ git archive --format=zip --output={filename}.zip HEAD

        # Create a compressed tarball archive.
        $ git archive --format=tar.gz --output={filename}.tar.gz HEAD
        ```

    * How do I snapshot gists/snippets? To snapshot a GitHub gist, I simply save it with its `{gist_id}` instead of `{repo}`, include the `{remote}` name (e.g., `gist.github.com--`) and an appropriate `{description}`. GitLab snippets use a different URL structure, but can be mapped to this schema using `{remote}=gitlab.com`, `{repo}={snippet_id}`, and an appropriate `{description}`. Other service providers likely follow similar structures; if I ever start using them heavily, this may be revised.

    * Why not store my snapshots in a "snapshot" repo, instead of a directory in my cloud storage? Honestly, I don't have a reason not to. Repo size limits are ridiculously large.
    
        The only guideline that gives me pause comes from [GitHub's "Repository limits" documentation](https://docs.github.com/en/repositories/creating-and-managing-repositories/repository-limits#activity) where they recommend keeping files small and enforce a 100MB limit per file. My largest snapshot is about 10MB; even if it was larger, GitHub Large File Storage (LFS) is an option.
        
        So why not move snapshots to a repo? Cloud storage works. Moving them into a repo wouldn't change my workflow - it would only change the location I pull from when setting up a new machine and where I store new snapshots. Since both scenarios are infrequent: if it ain't broke, why fix it?

### Experiments are in `~/_dev/scratch/`

* Anything inside `~/_dev/scratch/` is disposable.

    None of the content in this folder is in a repo, and it is not cloud-synced. The code here is primarily experiments that I am willing to lose if my system crashes. I can live with that.

* If a particular experiment becomes interesting to keep, I simply move it to the `~/_dev/repos/` folder, put it under version control, and push it to a remote for safekeeping. It is then treated as an active repo.

## Failed Iterations and Dead Ends

I thought it worthwhile to share some dead ends I encountered on the road to my current setup. I hope documenting my mistakes helps you avoid them.

### Do Not Cloud-Sync Your Workspace

I once set up my workspace root folder in an iCloud drive on one of my laptops.

Everything worked fine. I had a cloud-synced backup as well as my GitHub remote, and that felt like a good safety net.

Then I went ahead and set up my workspace on a second laptop. I missed the obvious: iCloud synced updates from the second laptop with an active repo on the first. I did not notice until I went back to work on my primary laptop. I followed my usual push/pull/sync workflow. And then Warnings. Errors. Sync Conflicts. I was mystified until it dawned on me... Ugh... I ended up rebuilding the repo and lost all commit history in the process.

Lesson learned: Never configure a directory with two or more competing sync engines.

### GitHub Gists Are Not Regular Git Repos

I once tried setting up a top-level folder for gists, `~/_dev/gists/`.

I was motivated by [GitHub's "Creating gists" documentation](https://docs.github.com/en/get-started/writing-on-github/editing-and-sharing-content-with-gists/creating-gists), which states that gists are Git repositories.

I set out to create a library of gist repos similar to regular repos. Gists have no repo name, but they can be uniquely identified by their `{gist_id}` (e.g., `dc591f733ac66699ff81f14acc6b290d`). I'd add the `{gist_name}` for readability, and because the naming convention differed from regular repositories, a separate `~/_dev/gists/` folder felt like a natural choice:

```sh
~/_dev/gists/{remote}/{account}/{gist_id}--{gist_name}/
```

Gists, however, turn out to be sufficiently different from regular repositories that the same workflows do not apply. Getting them to behave as a proper repo required too many workarounds:

* To edit/update required the specific use of the GitHub CLI (e.g., `$ gh edit`) instead of Git commands. Many experiments and errors faced to arrive at this fact omitted to spare the reader.

* VSCode (my current IDE) did not have native integration with gists. VSCode extensions exist, but they are not attractive and none fit within my repo workflow.

* Gists do not support folders. Thus, in order to render a README.md file inside a gist, we must rename it to "_README.md" to render at the top of the gist - and so on for other files, to have them displayed in a particular order.

* Others were also having trouble:

    * [https://gist.github.com/jakelevi1996/9901b95b45aa9d0f4e888fa7ae111120](https://gist.github.com/jakelevi1996/9901b95b45aa9d0f4e888fa7ae111120)
    * [https://github.com/orgs/community/discussions/62203](https://github.com/orgs/community/discussions/62203)
    * [https://feeding.cloud.geek.nz/posts/using-github-gist-like-git-repo/](https://feeding.cloud.geek.nz/posts/using-github-gist-like-git-repo/)

Ultimately, this approach was not worthwhile to pursue.

I ended up saving all gists in a "gists" repo under `~/_dev/repos/github.com/maulomelin/gists/` with each gist under its own folder. I simply copy/paste any updates I make to a gist to its corresponding gist under [https://gist.github.com/maulomelin](https://gist.github.com/maulomelin) - a minor tweak to my workflow I can easily live with because these updates are straight-forward and relatively infrequent.

Lesson learned: GitHub gists are not normal repos; treat GitHub gists as files in a "gists" repo.