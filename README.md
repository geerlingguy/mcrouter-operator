# Mcrouter Operator

[![Build Status](https://travis-ci.com/geerlingguy/mcrouter-operator.svg?branch=master)](https://travis-ci.com/geerlingguy/mcrouter-operator)

The Mcrouter Operator was built with the [Ansible Operator SDK](https://github.com/operator-framework/operator-sdk/blob/master/doc/ansible/user-guide.md). It is not yet intended for production use.

[Mcrouter](https://github.com/facebook/mcrouter) is a memcached protocol router for scaling memcached deployments, written by Facebook.

[Dylan Murray](https://github.com/dymurray)'s memcached operator was the original inspiration for this operator.

## Usage

This Kubernetes Operator is meant to be deployed in your Kubernetes cluster(s) and can manage one or more mcrouter instances in the same namespace as the operator.

First you need to deploy Mcrouter Operator into your cluster:

    kubectl apply -f TODO

Then you can create instances of mcrouter, for example:

  1. Create a file named `my-mcrouter.yml` with the following contents:

         ```
         ---
         TODO
         ```

  2. Use `kubectl` to create the mcrouter instance in your cluster:

         ```
         kubectl apply -f my-mcrouter.yml
         ```

## Development

### Testing

#### Local tests with Molecule and KIND

Ensure you have the testing dependencies installed (in addition to Docker):

    pip install docker molecule openshift jmespath

Run the local molecule test scenario:

    molecule test -s test-local

#### Local development with minikube

##### Requirements

  - minikube
  - kubectl
  - operator-sdk

##### Connect to minikube docker environment

```sh
eval $(minikube docker-env)
```

##### Create the custom resource definition for mcrouter

```sh
kubectl create -f deploy/crds/mcrouter_v1alpha2_mcrouter_crd.yaml
```

##### Build the mcrouter-operator docker image

```sh
operator-sdk build mcrouter-operator:v0.0.1
```

##### Create service account, role and role_bindings

```sh
kubectl create -f deploy/service_account.yaml
kubectl create -f deploy/role.yaml
kubectl create -f deploy/role_binding.yaml
```

##### Deploy the Operator

```sh
kubectl create -f deploy/operator.yaml
```

##### Create the custom resources for mcrouter

You can change the pods to be deployed inside the files

```sh
kubectl create -f deploy/crds/mcrouter_v1alpha2_mcrouter_cr.yaml
```

> Once everything is deployed you can use the `kubectl get all` command to check of mcrouter-operator and memcache and deployed mcrouter are working.

##### Use the below testing scenario to check if mcrouter and memcached are working as expected

Connect to mcrouter using the telnet container and send the following commands to see if you get expected output

```sh
kubectl run -it --rm telnet --image=jess/telnet --restart=Never <mcrouter_pod_ip> 5000
```

In the telnet prompt send below commands

```
    set mykey 0 0 5
    hello
    get mykey
    quit
```

Connect to memcached service using the telnet container and send the `stats` command to see if it gives you output.

## Helpful articles and referenced content below:

  - https://www.ansible.com/blog/ansible-operator
  - https://opensource.com/article/18/10/ansible-operators-kubernetes
  - https://blog.openshift.com/reaching-for-the-stars-with-ansible-operator/
  - https://github.com/Dev25/mcrouter-docker/
  - https://itnext.io/a-practical-kubernetes-operator-using-ansible-an-example-d3a9d3674d5b
  - https://github.com/helm/charts/tree/master/stable/mcrouter
