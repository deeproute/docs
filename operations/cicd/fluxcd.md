# FluxCD Bootstrap

## GitHub

### Login to Github using the CLI

```sh
gh auth login

#Retrieve the token:

gh auth status -t
```

### Run flux bootstrap

```sh
export GITHUB_USER=youruser
export GITHUB_TOKEN=yourtokenfrompreviousstep
export GITHUB_REPO=yourrepo
export GITHUB_PATH=yourrepoclusterpath

flux bootstrap github \
  --owner=$GITHUB_USER \
  --repository=$GITHUB_REPO \
  --branch=main \
  --path=$GITHUB_PATH \
  --personal
```
