# Mcrouter Operator

[![Build Status](https://travis-ci.com/geerlingguy/mcrouter-operator.svg?branch=master)](https://travis-ci.com/geerlingguy/mcrouter-operator)

The Mcrouter Operator was built with the [Ansible Operator SDK](https://github.com/operator-framework/operator-sdk/blob/master/doc/ansible/user-guide.md). It is not yet intended for production use.

[Mcrouter](https://github.com/facebook/mcrouter) is a memcached protocol router for scaling memcached deployments, written by Facebook.

[Dylan Murray](https://github.com/dymurray)'s memcached operator was the original inspiration for this operator.

## Usage

This Kubernetes Operator is meant to be deployed in your Kubernetes cluster(s) and can manage one or more mcrouter instances in the same namespace as the operator.

First you need to deploy Mcrouter Operator into your cluster:

    kubectl apply -f https://raw.githubusercontent.com/geerlingguy/mcrouter-operator/master/deploy/mcrouter-operator.yaml

Then you can create instances of mcrouter, for example:

  1. Create a file named `my-mcrouter.yml` with the following contents:

     ```
     ---
     apiVersion: mcrouter.example.com/v1alpha3
     kind: Mcrouter
     metadata:
       name: my-mcrouter
     spec:
       mcrouter_image: devan2502/mcrouter:latest
       mcrouter_port: 5000
       memcached_image: memcached:1.5-alpine
       # The size of the memcached pool.
       memcached_pool_size: 3
       memcached_port: 11211
       # The memcached pool can be 'sharded' or 'replicated'.
       pool_setup: replicated
       # Set to '/var/mcrouter/fifos' to debug mcrouter with mcpiper.
       debug_fifo_root: /var/mcrouter/fifos
     ```

  2. Use `kubectl` to create the mcrouter instance in your cluster:

     ```
     kubectl apply -f my-mcrouter.yml
     ```

> **What's the difference between `sharded` and `replicated`**: `sharded` uses a key hashing algorithm to distribute Memcached `set`s and `get`s among Memcached Pods; this means a key `foo` may always go to pod A, while the key `bar` always goes to pod B. `replicated` sends all Memcached `set`s to all Memcached pods, and distributes `get`s randomly.

## Development

### Testing

#### Local tests with Molecule and KIND

Ensure you have the testing dependencies installed (in addition to Docker):

    pip install docker molecule openshift jmespath

Run the local molecule test scenario:

    molecule test -s test-local

#### Testing if mcrouter and memcached are working as expected

Get the Kubernetes network IP address for the mcrouter pod:

    kubectl describe pod -l app=mcrouter

Then run a `telnet` container to connect to mcrouter directly:

```sh
kubectl run -it --rm telnet --image=jess/telnet --restart=Never <mcrouter_pod_ip> 5000
```

After a few seconds you will see a message like `If you don't see a command prompt, try pressing enter.`. Don't press enter, because telnet doesn't display a prompt. Instead, enter the below commands:

In the telnet prompt send commands like the following:

```
    set mykey 0 0 5
    hello
    get mykey
    stats
    quit
```

You can also inspect Mcrouter fifos using `mcpiper`, by setting `spec.debug_fifo_root` to `/var/mcrouter/fifos`, then running `mcpiper` inside the mcrouter pod once it's reconfigured: `/usr/local/mcrouter/install/bin/mcpiper`. Note that you will not see any output (besides maybe an error message) until requests are sent to mcrouter.

### Release Process

There are a few moving parts to this project:

  1. The Docker image which powers Mcrouter Operator.
  2. The `mcrouter-operator.yaml` Kubernetes manifest file which initially deploys the Operator into a cluster.

Each of these must be appropriately built in preparation for a new tag:

#### Build a new release of the Operator for Docker Hub

Run the following command inside this directory:

    operator-sdk build geerlingguy/mcrouter-operator:0.2.0

Then push the generated image to Docker Hub:

    docker login -u geerlingguy
    docker push geerlingguy/mcrouter-operator:0.2.0

#### Build a new version of the `mcrouter-operator.yaml` file

Update the mcrouter-operator version in two places:

  1. `deploy/mcrouter-operator.yaml`: in the `ansible` and `operator` container definitions in the `mcrouter-operator` Deployment.
  2. `build/chain-operator-files.yml`: the `operator_image` variable.

Once the versions are updated, run the playbook in the `build/` directory:

    ansible-playbook chain-operator-files.yml

After it is built, test it on a local cluster:

    minikube start
    kubectl apply -f deploy/mcrouter-operator.yaml
    kubectl apply -f deploy/crds/mcrouter_v1alpha3_mcrouter_cr.yaml
    <test everything>
    minikube delete

If everything works, commit the updated version, then tag a new repository release with the same tag as the Docker image pushed earlier.

## More resources for Ansible Operator SDK and Mcrouter

  - [Ansible Operator: What is it? Why it matters? What can you do with it?](https://www.ansible.com/blog/ansible-operator)
  - [An introduction to Ansible Operators in Kubernetes](https://opensource.com/article/18/10/ansible-operators-kubernetes)
  - [Reaching for the Stars with Ansible Operator](https://blog.openshift.com/reaching-for-the-stars-with-ansible-operator/)
  - [Mcrouter Wiki](https://github.com/facebook/mcrouter/wiki)
  - [Mcrouter Docker image](https://github.com/Dev25/mcrouter-docker/)
  - [A Practical kubernetes Operator using Ansible — an example](https://itnext.io/a-practical-kubernetes-operator-using-ansible-an-example-d3a9d3674d5b)
  - [Mcrouter Helm Chart](https://github.com/helm/charts/tree/master/stable/mcrouter)
