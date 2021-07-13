# kubebuilder-guestbook

## Dependencies
This installation assumes Docker for Mac (which has [known issues](https://kind.sigs.k8s.io/docs/user/known-issues/));
we'll be working around those issues with our deps and runbook.

- kind
- Docker for Mac 
- Kubectl wwith brew `brew install kubectl-cli`
- Overwrite symlinks created by Docker for Mac `brew link --overwrite kubernetes-cli`


> Go
- `GO111MODULE` on
- `go env`

> Kind and kubeconfig
- To get config of default cluster `kind get kubeconfig`
- To get config for non default cluster, use cluster name: `kind get kubeconfig --name=guestbook`
- `export KUBECONFIG="$(kind get kubeconfig --name="guestbook")"`

## Runbook

- `kind create cluster guestbook`
- `k get nodes` --> control plane

- `k create -f ./create/create.yaml`
- `k get po` shows frontend and redis pods/replicas creating ....
- Port forward: `k port-forward service/frontend 8080:80`
- 
