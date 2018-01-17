
# making a fork

    # at spec/fixtures/testrepo.git
    git remote add origin https://github.com/my-user/git-workshop.git
    cd ..
    git diff --color testrepo.git/config
    cd -
    git push -u origin master
    cd ..
    git diff --color testrepo.git/config
    cd -

    git push origin refs/heads/feature/branch:refs/remotes/origin/feature/branch # == git push origin feature/branch
    git push origin 6957dc21ae95f0c70931517841a9eb461f94548c:master-ahead

    git branch -v
    echo 6957dc21ae95f0c70931517841a9eb461f94548c > refs/heads/master
    echo ba46b858929aec55a84a9cb044e988d5d347b8de > refs/heads/feature/branch
    git branch -v

    git push origin master:master-ahead feature/branch:branch-ahead

