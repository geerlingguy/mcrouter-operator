# Mcrouter Operator

[![Build Status](https://travis-ci.com/geerlingguy/mcrouter-operator.svg?branch=master)](https://travis-ci.com/geerlingguy/mcrouter-operator)

The Mcrouter Operator was built with the [Ansible Operator SDK](https://github.com/operator-framework/operator-sdk/blob/master/doc/ansible/user-guide.md). It is not yet intended for production use.

[Mcrouter](https://github.com/facebook/mcrouter) is a memcached protocol router for scaling memcached deployments, written by Facebook.

[Dylan Murray](https://github.com/dymurray)'s memcached operator was the original inspiration for this operator.

## Usage

This Kubernetes Operator is meant to be deployed in your Kubernetes cluster(s) and can manage one or more mcrouter instances in the same namespace as the operator.

First you need to deploy Mcrouter Operator into your cluster:

    kubectl apply -f https://raw.githubusercontent.com/geerlingguy/drupal-operator/master/deploy/mcrouter-operator.yaml

Then you can create instances of mcrouter, for example:

  1. Create a file named `my-mcrouter.yml` with the following contents:

     ```
     ---
     apiVersion: mcrouter.example.com/v1alpha2
     kind: Mcrouter
     metadata:
       name: my-mcrouter
     spec:
       size: 2
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

#### Testing if mcrouter and memcached are working as expected

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

### Release Process

There are a few moving parts to this project:

  1. The Docker image which powers Mcrouter Operator.
  2. The `mcrouter-operator.yaml` Kubernetes manifest file which initially deploys the Operator into a cluster.

Each of these must be appropriately built in preparation for a new tag:

#### Build a new release of the Operator for Docker Hub

Run the following command inside this directory:

    operator-sdk build geerlingguy/mcrouter-operator:0.0.1

Then push the generated image to Docker Hub:

    docker login -u geerlingguy
    docker push geerlingguy/mcrouter-operator:0.0.1

#### Build a new version of the `mcrouter-operator.yaml` file

Verify the `build/chain-operator-files.yml` playbook has the most recent version/tag of the Docker image, then run the playbook in the `build/` directory:

    ansible-playbook chain-operator-files.yml

After it is built, test it on a local cluster (e.g. `minikube start` then `kubectl apply -f deploy/mcrouter-operator.yaml`), then commit the updated version and push it up to GitHub, tagging a new repository release with the same tag as the Docker image.

## Helpful articles and referenced content below:

  - https://www.ansible.com/blog/ansible-operator
  - https://opensource.com/article/18/10/ansible-operators-kubernetes
  - https://blog.openshift.com/reaching-for-the-stars-with-ansible-operator/
  - https://github.com/Dev25/mcrouter-docker/
  - https://itnext.io/a-practical-kubernetes-operator-using-ansible-an-example-d3a9d3674d5b
  - https://github.com/helm/charts/tree/master/stable/mcrouter
