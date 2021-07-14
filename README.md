# kubebuilder-guestbook

- Inspired by [https://github.com/sethp-nr/guestbook-workshop](https://github.com/sethp-nr/guestbook-workshop)
- Using the [Kubebuilder](https://github.com/kubernetes-sigs/kubebuilder) Quick Start

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

> Kind and kubeconfig (note: tends to be buggy).
- To get config of default cluster `kind get kubeconfig`
- To get config for non default cluster, use cluster name: `kind get kubeconfig --name=guestbook`
- `export KUBECONFIG="$(kind get kubeconfig --name="guestbook")"`

## Runbook (imperative)
> This is an imperative runbook to create a cluster, create some resources using
`create/apply`, and then observe how editing and changing resources imperatively
will take effect.

> _Once we do this brief imperative "confirmation that things are working", we'll delete these resources and redo things declaratively.

- `kind create cluster guestbook`
- `k get nodes` --> control plane
- `k create -f https://raw.githubusercontent.com/kubernetes/examples/011284134a724c0ce30f9fa4ec966bdbdefb843e/guestbook/all-in-one/guestbook-all-in-one.yaml`
- `k get po` shows frontend and redis pods/replicas creating ....
- Port forward: `k port-forward service/frontend 8080:80`, and go to browser
http://localhost:8080 to see the Guestbook service running;
- `edit` the yaml by using `kubectl edit pod/{podname}` - like vim;
add an annotation under metadata for one of the front end pods, and then 
`kubectl get pods -l app=guestbook` and observe that you can see the annotations
in that pod.
- Cleanup: `k delete -f https://raw.githubusercontent.com/kubernetes/examples/011284134a724c0ce30f9fa4ec966bdbdefb843e/guestbook/all-in-one/guestbook-all-in-one.yaml`

## Kubebuilder
- `kubebuilder init --domain example.com`
- `kubebuilder create api`