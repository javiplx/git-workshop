
# sample repository

    git clone https://github.com/jenkinsci/gitlab-hook-plugin.git
    cd gitlab-hook-plugin
    git remote -v
    cd spec/fixtures/testrepo.git
    git remote -v
    git status

    git log --oneline --all # just all reachable commits
    git log --oneline ba46b858929aec55a84a9cb044e988d5d347b8de
    git log --oneline 6957dc21ae95f0c70931517841a9eb461f94548c

    find objects -type f | wc -l
    ls
    git gc
    find objects -type f | wc -l
    ls

    cd ..
    git status
    git checkout -- testrepo.git
    git status
    ls testrepo.git

