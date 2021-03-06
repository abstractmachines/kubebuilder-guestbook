# kubebuilder-guestbook

- Inspired by [https://github.com/sethp-nr/guestbook-workshop](https://github.com/sethp-nr/guestbook-workshop)
- Using the [Kubebuilder](https://github.com/kubernetes-sigs/kubebuilder) Quick Start

## Table of contents

> Theory:
- [Kubernetes API Design](#Kubernetes-API-Design)

> Project related:
- [Dependencies for installation](#Dependencies)
- [Imperative quick runbook](#Runbook-(imperative))
- [Kubebuilder Init](#Kubebuilder-installation)
- [API : CRDs](#Kubebuilder-API)
- [Controller: Logic for the API](#Controller)


## Kubernetes API Design

- Kubernetes CRDs are the canonical API for custom resources.

- Controllers are the business logic behind that API. Controllers _reconcile_ current state (`state`) with desired state (`spec`).

- Think of a `state machine`.

- Book: Programming Kubernetes

  To grok Kubernetes API Design and the "why", I highly recommend reading [Programming Kubernetes, an O'Reilly book](https://www.oreilly.com/library/view/programming-kubernetes/9781492047094/).

  There are a lot of topics in the first several chapters such as `kind`, `GVK` `GVR`,
  versions, groups, and the history and theory of these concepts. It's a fantastic start to this material.

  This tutorial assumes knowledge of that material, particularly chapters 1-3 "Kubernetes API Basics" and "client-go."

- Kubebuilder Kubernetes API Design docs [here](https://book.kubebuilder.io/cronjob-tutorial/api-design.html)

> Some basic API design rules for Kubernetes:

- Serialized fields must be camelCase; use JSON structs to specify this.
- A lot of these "types" are structs for documentation, metadata and validation (struct tags).
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

> *Once we do this brief imperative "confirmation that things are working", we'll delete these resources and redo things declaratively.*

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

## Kubebuilder installation

> Kubebuilder init to create project structure and "boilerplate", and do a smoke test running main go module

  ```
  go mod init
  kubebuilder init --domain example.com
  go run main.go
  ```

You'll see something like:
  ```
  controller-runtime.metrics	metrics server is starting to listen	{"addr": ":8080"}
  setup	starting manager
  controller-runtime.manager	starting metrics server	{"path": "/metrics"}
  ```
- curl metrics endpoint `localhost:8080/metrics` and you'll see a lot of REST calls, that confirms it's working.

## Kubebuilder API

> **Overview:** Custom Resources are considered an API for Kubernetes operators. When we give our resource a `kind`, we are creating a new `type` of sorts.

- Once we create an API, we write a controller to implement that API.

- Kubebuilder Kubernetes API Quickstart [here](https://book.kubebuilder.io/quick-start.html#create-an-api)

> Let's create our API:

- Again note the credit for the phrasing and teaching of these ideas should go to Seth;
please check out his repo on this material [here](https://github.com/sethp-nr/guestbook-workshop).

- Create an API: `kubebuilder create api --group webapp --version v1 --kind Guestbook`
- Select `Yes` for both Create Resource and Create Controller opts
- Once it's installed, `tree api controllers` to tree both of those directories;
the `/api/v1` dir has api types and some generated function defs, and `controllers`
is where our "operational expertise" will be encoded (controller logic).

> Now run tests:
  ```
  make test
  ```
  ... expected result is "no test files.... ok" result.

## Controller

> Reconciliation: Spec and State

- Let's fill out the controller logic with "reconciling" the current state with 
our desired state (e.g. spec), in `./controllers/guestbook_controller.go`.

