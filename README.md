
## a bit of theoretical/empirical facts

* internally git only understands sha1, but offers two label helpers for people
  - static (tags) pointing to a specific sha1 commit
  - dynamic (branches) pointing to a commit but forwardable (a sliding tag)
* each commit has (at least) one parent, except for the initial(s) commit
* a commit is a complex object
  - has two associated people/timestamps en cada commit (author & committer) [avatar within avatar on github]
  - has no one to one mapping with filesystem entities [once, I believed three inodes for each one]
* it is possible to make commits **about** (not after) commits (annotated tags & git notes)


## what a commit looks like

## what we will see

http://visualize-your-git.herokuapp.com/

![Git commands](http://ks301030.kimsufi.com/gitcommands-markov.png)

## git 101

### creating a repository

Creating a new one [on existing directory]

    git init [--bare]

Create a bare repository is like a move into the .git subdirectory, with checkout and friends disabled. It is typical on the a pure git _server_

#### git files

Structure of bare repository [or .git subdir]
* config, self explanatory (`git config --local --list`)
* HEAD, which is the checked out branch (or the _default_ one for bare repositories)
* objects, where commits (blobs?) themselves live (although it will be useless for us)
* refs, [or why staging branch prevents staging/recommender] where the branch/tag labels are found, along many other things

### cloning a repository

Quite easy [requires non-existing directory]

    git clone repo_uri

or not ?
* optionally specify a directory name
* `--branch` to checkout a specific branch
* `--recursive` for submodules (if you really want to use them)
* `--depth` for shallow clones (downloading only part of the history)
* removed option to get only some of the remote branches
* and a few more

#### manual clone

A longer version, where command line arguments get into full commands [more or less]

    cd repoclone
    git init
    git remote add origin git@gitlab.fon.ofi:qa/asimov.git
    git fetch origin +refs/heads/*:refs/remotes/origin/*
    git checkout -b master origin/master

The `+refs/heads/*:refs/remotes/origin/*` (called refspec in documentation) is optional, but shows how
the remote branches are mapped to local-remote branches.
The first part (_:_ split) is how the branches are named on the remote, and the second ones how they are
named locally. And the leading _+_ mean that non fast-forwards are allowed [also named _forced_].

The conventios says that refs/heads are your local branches, and refs/remotes is used for the remote
ones, with one subdir for each remote (thats the _origin_ in the refspec), but it is not enforced. An
ugly trick to build tags on jenkins is to use `+refs/tags/*:refs/remotes/origin/tags/*` as refspec,
which makes that remote tags are seen as remote branches with a leading _tag/_
[http://erics-notes.blogspot.be/2013/05/jenkins-build-latest-git-tag.html]

## the sample repository

Lets start with an rare spice: a git repository within a git repository. But not in the submodule
flavour, but both in the same remote [the trick is that the inner repository is a bare one]

    git clone https://github.com/jenkinsci/gitlab-hook-plugin.git
    cd gitlab-hook-plugin
    git remote -v

usual up to here but

    cd spec/fixtures/testrepo.git
    git remote -v

and do you remember about the usefulness of knowing where the commits are?

    find objects -type f | wc -l
    git log --oneline --all | wc -l
    git gc
    find objects -type f | wc

## the sample repository

Create a new empty repo on github

    git remote add origin https://github.com/Feverup/git-workshop.git
    cd ..
    git diff --color testrepo.git/config
    cd -

    git push -u origin master
    cd ..
    git diff --color testrepo.git/config
    cd -

but only master is there, let push a few more branches

    git push origin refs/heads/feature/branch:refs/remotes/origin/feature/branch

where the refspec is a complex way to obscure a simple command which taks advantage
of short refspec format (`git push origin feature/branch`)

Only references can be pushed, which is probably the only point where sha1 are second class citizens

    git push origin 6957dc21ae95f0c70931517841a9eb461f94548c:master-ahead

so, cannot we create branches in our bare repository? Obviously. Yes, we can

    git branch -v
    echo 6957dc21ae95f0c70931517841a9eb461f94548c > refs/heads/master
    echo ba46b858929aec55a84a9cb044e988d5d347b8de > refs/heads/feature/branch
    git branch -v

and we can push them with different names (again using shortened refspecs)

    git push origin master:master-ahead feature/branch:branch-ahead

## confusing local branches

The refspecs are not enforced by any mean, nor the actuall branch names

    git checkout -b origin/master-ahead
    git branch -v
    git branch -va
    git push origin origin/master-ahead
    
    git checkout -b remotes/origin/master-ahead
    git branch -v
    git branch -va
    git push origin remotes/origin/master-ahead

although fortunatelly this last type cannot be pushed (now) to github

## get changes from remote

As everybody knows, the way to get changes from remote is `git pull` but, as often happens,
everybody is wrong. That is the quick & dirty way to do it. What's wrong there? Basically,
that history (network) gets very confusing (both for people and git itself). The visible sign
of this are messages like _"merge staging into staging"_, and nobody could prove that the
_spaguetti network_ is not the cause of commits dissapearance.

Surprisingly, the quick & clean way is only a bit longer: `git pull --rebase`. What is the
difference? Directly from fetch help

    More precisely, git pull runs git fetch with the given parameters and calls git merge to merge the retrieved
    branch heads into the current branch. With --rebase, it runs git rebase instead of git merge.

So, using _--rebase_ will just move your commits into the top of the remote branch, producing a
linear history, and removing all problems.

As happened wit clone, there is also a long and detailed path to do the pull. It starts by getting
remote changes into our local copy with `git fetch origin`, and the continuation differ on the
status of our local branch. If we have no changes and just want to sync our local branch with remote

    git merge --ff-only origin/branchname

which is the only case when a raw `git pull` produces exactly the same result. If we have local commits,
we perform a rebase with

    git rebase --onto origin/branchname HEAD~X

where X can be known usually from the fetch output, but also by running `git branch -v`, that will show
a message like '[ahead 1, behind 1]', where X is the _ahead_. But, anyway, all this guessing could be
avoided just runnning `git rebase branchname`, that should produce the same result.

## working with multiple remotes

Lets start with the first exercise, that will also simulate a forked repository.
We will end with two remotes, the one that we own, and upstream, and they behave
like a fork because the sha1 of both repositories are the same.

If this were a standard feature fork, working with the our and making a pull request when
finish will suffice. But if there are conflicts preventing the merge or we want to add
some features developed after our fork, we must use cherry-pick, rebase or merge depending
on our needs.

The simplest case is just a conflict in the merge, that is tipically solved by merging the
remote destination branch in our local branch, and then pushing again.

    git checkout my-feature-branch
    git fetch upstream
    git merge upstream/master
    # resolve your conflicts ...
    git commit -m 'Resolve conflicts'
    git push origin my-feature-branch

and the remaining work will be done by github, gitlab or whoever your remote is hosted on.
Although this is the standard flow when forking, when you have write access to the upstream
repository, it is much cleaner to do the merge locally, resolve the conflicts and push the
result

    gitsync upstream master
    git merge --no-commit my-feature-branch
    # resolve your conflicts ...
    git commit -m 'Resolve conflicts'
    git push upstream master

The use of cherry-pick and specially rebase are considered more advanced topics and will be described later.

## paradise in hell (workspace, stage, index, ...)

Although maybe you can guess what is workspace, the other terms are less clear. So, let's start with
a couple of surprising facts or misconceptions: 
First of all, `git add` does not add files to the repository, but moves content into stage. 
Then, using `git rm`, moves files from stage to workspace? Nope, it does actually remove files, but only from workspace. 
Yes, you're missing a few commands here. Let say that _checkout_ and _reset_ are basically what you miss there, although to see all this mess in action you need to find an enterprise grade merge conflict.

Basically, there are two areas in git where your files (or parts of them) can be classified. Or three
if we consider files out of source control. Let say that stage is the area of changes ready to be
committed, and workspace are those not yet ready. This means that if you run `git commit`, your stage
area will be transformed into a commit (and cleaned). If you are ussed to `git commit -a`, it is like
to run `git add` on the workspace and the commit. And running `git commit filename` will commit all
your changes (stage+workspace) from the specified file(s).

With this clear introduction in mind, lets review the commands mentioned before.
* `git add`, does move things into stage. The misconception comes from the fact that it is the only way to put an untracked file under source control. Once a thing get added, we can perform additional changes on it, and this new content will not get commited
* `git rm` moves files out of source control, with the side effect of actually removing them from the filesystem. If part of the file is in stage, you will need to force the removal
* `git reset HEAD` is the command that performs the opposite of _add_, moving content out of stage area, although with _--hard_ can also rewind the file into its last committed state
* `git checkout`, finally, is the command that you need to drop your changes in workspace, while keeping what you have in stage

