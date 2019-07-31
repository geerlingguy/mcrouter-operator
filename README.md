# Mcrouter Operator

[![Build Status](https://travis-ci.com/geerlingguy/AnsOpDemo.svg?branch=master)](https://travis-ci.com/geerlingguy/AnsOpDemo)

The Mcrouter Ansible Operator was built with the [Operator SDK](https://github.com/operator-framework/operator-sdk) and [Ansible Operator](https://www.ansible.com/blog/ansible-operator). It is not yet intended for production use.

[Mcrouter](https://github.com/facebook/mcrouter) is a memcached protocol router for scaling memcached deployments, written by Facebook.

[Dylan Murray](https://github.com/dymurray)'s memcached operator was the original inspiration for this operator.

## Instructions to Run

### Requirements

  - minikube
  - kubectl
  - operator-sdk

### Connect to minikube docker environment

```sh
eval $(minikube docker-env)
```

### Create the custom resource definition for mcrouter

```sh
kubectl create -f deploy/crds/mcrouter_v1alpha2_mcrouter_crd.yaml
```

### Build the mcrouter-operator docker image

```sh
operator-sdk build mcrouter-operator:v0.0.1
```

### Create service account, role and role_bindings

```sh
kubectl create -f deploy/service_account.yaml
kubectl create -f deploy/role.yaml
kubectl create -f deploy/role_binding.yaml
```

### Deploy the Operator

```sh
kubectl create -f deploy/operator.yaml
```

### Create the custom resources for mcrouter

You can change the pods to be deployed inside the files

```sh
kubectl create -f deploy/crds/mcrouter_v1alpha2_mcrouter_cr.yaml
```

> Once everything is deployed you can use the `kubectl get all` command to check of mcrouter-operator and memcache and deployed mcrouter are working.

### Use the below testing scenario to check if mcrouter and memcached are working as expected

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
