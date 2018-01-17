
# multiple remotes

    # at spec/fixtures/testrepo.git
    git clone $PWD ../myrepo
    cd ../myrepo

or

    git clone https://github.com/my-user/git-workshop.git myrepo
    cd myrepo

    cat .git/config
    git remote add upstream git@github.com:Feverup/git-workshop.git
    cat .git/config

    git branch -va
    git fetch upstream
    git branch -va
    
    git checkout workshop
    cat .git/config

