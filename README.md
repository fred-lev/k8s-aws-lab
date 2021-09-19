# Kubernetes AWS lab environment

## Install aws-cli

```console
brew info awscli
```

Set `~/.aws/config`

```console
[default]
region = ap-southeast-1
output = json
cli_pager = # without it, commands outputs are not sent to vscode terminal
```

## Installing Kubernetes with kops

Check the following official docs:

- [Installing Kubernetes with kops](https://kubernetes.io/docs/setup/production-environment/tools/kops/)

- [getting-started-with-kops-on-aws](https://github.com/kubernetes/kops/blob/master/docs/getting_started/aws.md#getting-started-with-kops-on-aws)


I am using my own domain here.

```console
ID=$(uuidgen) && aws route53 create-hosted-zone --name lab.flev.fr --caller-reference $ID | jq .DelegationSet.NameServers
```

On the domain hosting side I have added the NS records from the above cmd for the subdomain `lab.flev.fr`.

create an S3 bucket to save the state of the cluster and encrypt it.

```console
aws s3api create-bucket --bucket lab-flev-fr-state-store --create-bucket-configuration LocationConstraint=ap-southeast-1
aws s3api put-bucket-versioning --bucket lab-flev-fr-state-store  --versioning-configuration Status=Enabled
aws s3api put-bucket-encryption --bucket lab-flev-fr-state-store  --server-side-encryption-configuration '{"Rules":[{"ApplyServerSideEncryptionByDefault":{"SSEAlgorithm":"AES256"}}]}'
```

export cluster and state store names:

```console
export NAME=lab.flev.fr
export KOPS_STATE_STORE=s3://lab-flev-fr-state-store
```

List the az available in the region and pick one to generate the cluster config

```console
aws ec2 describe-availability-zones --region ap-southeast-1

kops create cluster \
    --zones=ap-southeast-1a,ap-southeast-1b,ap-southeast-1c \
    ${NAME}
```

Generate a new RSA key to be used by kops (elliptic curve isn't supported)

```console
ssh-keygen -t rsa -f  ~/.ssh/kops_rsa
ssh-add -K ~/.ssh/kops_rsa
```

Edit the cluster config and set the generated key for the admin user

```console
kops create secret --name  ${NAME} sshpublickey admin -i ~/.ssh/kops_rsa.pub
```

Get pub IP and restrict ssh and api access to that pub Ip by overwritting the 0.0.0.0/0 entry

```console
curl ifconfig.me
kops edit cluster ${NAME}
```

Create the cluster and export the admin config

```console
kops update cluster ${NAME} --yes
kops export kubecfg --admin
```

Validate the cluster creation.

```console
kops export kubecfg --admin
```

Note: it might take some time for the api.lab.flev.fr A record to resolve.

Test access from outside by deploying an nginx Pod and exposing it using a loadbalancer type service.

Add the following policy `ElasticLoadBalancingFullAccess` to the `masters.lab.flev.fr` role.

If not it won't be able to create the AWS loadbalancer.

```console
k run nginx --image=nginx
k expose pod nginx --type LoadBalancer --port 80
```

Try to reach the external DNS displayed under `EXTERNAL-IP` in the output of `k get svc nginx`.

Delete the cluster:

```console
kops delete cluster ${NAME} --yes
```

## managing a cluster using eksctl

Official doc for [eksctl](https://eksctl.io/)

- [minimum IAM permissions](https://eksctl.io/usage/minimum-iam-policies/) for a user to be able to fully handle the cluster creation via eksctl.

Best approach here is to create a group, give it the `minimum IAM permissions` and assign a new eksctl user to that group and run aws configure to login as that user.

### create the cluster

```console
eksctl create cluster \
--name lab1 \
--region ap-southeast-1 \
--with-oidc \
--ssh-access \
--ssh-public-key ~/.ssh/kops_rsa.pub \
--managed
```

### restrict kube api access to your public IP

```console
eksctl utils set-public-access-cidrs --cluster=lab1 $(curl -s ifconfig.me)/32 --approve
```

### list nodes

```console
kubectl get nodes -o wide
```

### delete the cluster

```console
eksctl delete cluster \
--name lab1 \
--region ap-southeast-1 \
```

## Minikube setup for local testing

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

## Github actions

This repo is using GH Actions to execute [super-linter](https://github.com/github/super-linter).

I am mostly interested in linting the ansible (YAML), terraform(HCL) and markdown files in the repo using:

- ansible-lint
- markdownlint
- terrascan
- tflint
- yamlint

super-linter is an easy, low maintenance way to do it.
