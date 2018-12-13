---
layout: layout.pug
navigationTitle: Connecting to Kubernetes
title: Connecting to Kubernetes clusters
menuWeight: 10
excerpt: Learn to connect to private Kubernetes clusters and the Kubernetes dashboard via web proxy
---

Now that we have set up and configured our cluster with a pair of Kubernetes clusters on our DC/OS Enterprise cluster, you will learn to configure your DC/OS Enterprise cluster's network to:

- communicate with our Kubernetes clusters using the Kubernetes CLI - `kubectl`
- view your Kubernetes dashboard via web proxy

all from outside the DC/OS cluster's perimeter using [DC/OS Edge-LB](/services/edgelb/latest/) for DC/OS Enterprise.

# Next: Configure Edge-LB to Connect to Kubernetes

As the next step of [the Getting Started Guide](../getting-started/), you will [configure Edge-LB to connect to our Kubernetes clusters](../getting-started/connecting-to-kubernetes/config-edgelb/). This will then allow you to quickly test your cluster connections and view the Kubernetes dashboard via web proxy.
