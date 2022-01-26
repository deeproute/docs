
## Delete all merged local branches

```
git branch | grep -v "master" | xargs git branch -d
```


## Search for commits where the file contents has the <string> keyword
Source: https://stackoverflow.com/questions/4404444/how-do-i-git-blame-a-deleted-line

```
git log -S <string> path/to/file
```

## Squash Commit
Source: https://stackoverflow.com/a/50880042/3253796

Checkout the branch for which you would like to squash all the commits into one commit. Let's say it's called feature_branch.

```
git checkout feature_branch
```

Step 1:
Do a soft reset of your origin/feature_branch with your local main branch (depending on your needs, you can reset with origin/main as well). This will reset all the extra commits in your feature_branch, but without changing any of your file changes locally.

```
git reset --soft main
```

Step 2:
Add all of the changes in your git repo directory, to the new commit that is going to be created. And commit the same with a message.

```
git add ...
git commit -m "commit message goes here"

```
