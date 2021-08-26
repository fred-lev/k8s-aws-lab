# Kubernetes AWS lab environment

## Minikube commands

Delete any exiting/outdated minikube configuration

```console
minikube delete --all --purge
```

Start minikube using virtualbox driver

```console
minikube start --driver=virtualbox
```

Minikube autocompletion with zsh seems to be a bit broken:

Works by doing:

add  `minikube` in  plugin in `~/.zsh.rc`

```console
plugins=(
  minikube
)
```

remove double quotes for `aliashash` commands in completion cached file:

```console
sed -i "" 's/aliashash\["\([a-z]*\)"\]/aliashash[\1]/g' ~/.oh-my-zsh/cache/minikube_completion
```

## Installing Kubernetes with kops

[Installing Kubernetes with kops](https://kubernetes.io/docs/setup/production-environment/tools/kops/)

macOS and Linux From Homebrew

```console
brew update && brew install kops
```


## Github actions

This repo is using GH Actions to execute [super-linter](https://github.com/github/super-linter).

I am mostly interested in linting the ansible (YAML), terraform(HCL) and markdown files in the repo using:

- ansible-lint
- markdownlint
- terrascan
- tflint
- yamlint

super-linter is an easy, low maintenance way to do it.
