Mcrouter role for Mcrouter Operator
=========

An Ansible role which deploys Mcrouter and Memcached into a Kubernetes cluster. Meant to be used as part of the Mcrouter Operator.

Requirements
------------

A Kubernetes cluster.

Role Variables
--------------

    mcrouter_image: devan2502/mcrouter:latest

The container image to be used to run Mcrouter

    mcrouter_port: 5000

The port on which Mcrouter should be running. Used also to set the `containerPort` in the Mcrouter Deployment.

    memcached_image: memcached:1.5-alpine

The container image to be used to run Memcached.

    memcached_pool_size: 2

How many instances should be in the Memcached pool.

    memcached_port: 11211

The port on which Memcached should be running. Used also to set the `containerPort` in the Memcached StatefulSet, and for the connection between Mcrouter and Memcached.

    pool_setup: replicated

Can be one of `replicated` or `sharded`.

  - `replicated`: sends all Memcached `set`s to all Memcached pods, and distributes `get`s randomly.
  - `sharded`: uses a key hashing algorithm to distribute Memcached `set`s and `get`s among Memcached Pods; this means a key `foo` may always go to pod A, while the key `bar` always goes to pod B.

    debug_fifo_root: ''

You can debug and inspect Mcrouter fifos using `mcpiper`, by setting `debug_fifo_root` to `/var/mcrouter/fifos`. Use `mcpiper` inside the mcrouter pod once it's reconfigured: `/usr/local/mcrouter/install/bin/mcpiper`. Note that you will not see any output (besides maybe an error message) until requests are sent to mcrouter.

Dependencies
------------

None.

Example Playbook
----------------

This role is meant to be used in an Ansible Operator inside a Kubernetes cluster, so it may not work as expected in typical Ansible playbooks.

License
-------

BSD

Author Information
------------------

Anshul Behl (anshulbehl.19@gmail.com)
John Lieske (@johnlieske)
Jeff Geerling (@geerlingguy)
