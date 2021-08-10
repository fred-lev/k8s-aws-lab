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

```console
minikube completion zsh > ~/.minikube-completion
sed -i "" 's/aliashash\["\([a-z]*\)"\]/aliashash[\1]/g' ~/.minikube-completion
source <(~/.minikube-completion)
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
