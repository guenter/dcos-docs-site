---
layout: layout.pug
navigationTitle: Quick Start for GCP
title: Quick Start for GCP
menuWeight: 3
excerpt: A quick start guide for Kubernetes using the Mesosphere Universal Installer.
---

<!-- This source repo for this topic is https://github.com/mesosphere/dcos-kubernetes-cluster -->

This guide will take you through a (mostly) automated process of setting up and installing DC/OS 1.12, then installing the Mesosphere Kubernetes Engine and Kubernetes clusters onto the DC/OS cluster. All of the heavy lifting has been done for you through the default configuration in order to provide a basic DC/OS installation capable of hosting Kubernetes clusters. This is an open source installation meant to provide a quick way to evaluate Kubernetes on DC/OS.

![k8s UI](docs/assets/ui-install.gif)

**NOTE:** The latest `dcos-kubernetes-quickstart` doesn't support any Kubernetes framework version before `2.0.0-1.12.1`. The reason is that creating Kubernetes clusters now requires the installation of the [Mesosphere Kubernetes Engine](/services/kubernetes/latest/overview/#cluster-manager).

## Known limitations

Before proceeding, please check the [current package limitations](/services/kubernetes/latest/limitations/).

## Prerequisites

Check the requirements for running this quickstart:

* Linux or MacOS
* [Terraform 0.11.x](https://www.terraform.io/downloads.html). On MacOS, you can install with [brew](https://brew.sh/):
  ```bash
  $ brew install terraform
  ```
* [Google Cloud Platform Account](https://cloud.google.com/)

**WARNING**: When running this quickstart, you might experience some issues
with cloud resource limits. Please, verify your [quotas](https://cloud.google.com/compute/quotas)
before proceeding.

## Install the Google Cloud SDK and add your credentials

Make sure to have previously installed [Google Cloud SDK](https://cloud.google.com/sdk/downloads).

Then, retrieve the credentials needed for Terraform to manage your
Google Cloud resources:

```bash
$ gcloud auth login
$ gcloud auth application-default login
```

**Note:** If you want to use a GCP Service Account key instead of GCP SDK, follow the additional instructions 

## Setup SSH access

Next, you need to setup SSH as per [official GCP documentation](https://cloud.google.com/compute/docs/instances/adding-removing-ssh-keys) if you setup Google Cloud SDK or Google Cloud Service Account.

Also add your new SSH private key to your session:

```bash
$ ssh-add ~/.ssh/google_compute_engine
```

Later, you will be asked to add the SSH public key to the Terraform cluster profile.

## Prepare infrastructure configuration

Now, to start setting up your installation, start by generating the default infrastructure configuration:

```bash
$ make gcp
```

This will output sane defaults to `.deploy/desired_cluster_profile`.
Now, edit said file and set your `project-id` and the `gce_ssh_pub_key_file`
(the SSH public key you will use to log-in into your new VMs later).

**WARNING:** Please, do not set a smaller instance (VM) type on the risk of failing to install Kubernetes.

```shell
custom_dcos_download_path = "https://downloads.dcos.io/dcos/stable/1.12.0/dcos_generate_config.sh"
num_of_masters = "1"
num_of_private_agents = "4"
num_of_public_agents = "1"
#
gcp_project = "YOUR_GCP_PROJECT"
gcp_region = "us-central1"
gce_ssh_pub_key_file = "/PATH/YOUR_GCP_SSH_PUBLIC_KEY.pub"
#
gcp_bootstrap_instance_type = "n1-standard-1"
gcp_master_instance_type = "n1-standard-8"
gcp_agent_instance_type = "n1-standard-8"
gcp_public_agent_instance_type = "n1-standard-8"
# Inbound Master Access
admin_cidr = "0.0.0.0/0"
```

**Note:** Google Cloud Service Account

If you want to use GCP Service Account key instead of GCP SDK, uncomment the line to look as shown below in `desired_cluster_profile` and update it with the path to the ssh key file:

```shell
...
gcp_credentials_key_file = "/PATH/YOUR_GCP_SERVICE_ACCOUNT_KEY.json"
...
```

For more advanced scenarios, please check the [terraform-dcos documentation for Google Cloud](https://github.com/dcos/terraform-dcos/tree/master/gcp).

## Download the quickstart repository

Now that you are all set up with your cloud, clone this repo:

```bash
git clone git@github.com:mesosphere/dcos-kubernetes-quickstart.git && cd dcos-kubernetes-quickstart
```

### Configuring Kubernetes

#### Option for Role Based Access Control

**NOTE:** This `quickstart` will provision a Kubernetes cluster without `RBAC` support.

To deploy a cluster with enabled [RBAC](https://docs.mesosphere.com/services/kubernetes/2.0.1-1.12.2/operations/authn-and-authz/#rbac) update `.deploy/options.json` to add the authorization mode:

```json
{
  "service": {
    "name": "dev/kubernetes01"
  },
  "kubernetes": {
    "authorization_mode": "RBAC"
  }
}
```

**NOTE:** The authorization mode for a cluster must be chosen when installing the package. Changing the authorization mode after installing the package is not supported.

If you want to give users access to the Kubernetes API check [documentation](https://docs.mesosphere.com/services/kubernetes/2.0.1-1.12.2/operations/authn-and-authz/#giving-users-access-to-the-kubernetes-api).

#### Option for High Availability Cluster

**NOTE:** By default, it will provision a Kubernetes cluster with one (1) worker node, and a single instance of every control plane component.

To deploy a **highly-available** cluster with three (3) private Kubernetes nodes update `.deploy/options.json`:

```json
{
  "service": {
    "name": "dev/kubernetes01"
  },
  "kubernetes": {
    "high_availability": true,
    "private_node_count": 3
  }
}
```

### Download command-line tools

If you don't have them as binaries already, please download the DC/OS CLI client, `dcos` and the DC/OS Kubernetes client, `kubectl` by running:

```bash
make get-cli
```

The `dcos` and `kubectl` binaries will be downloaded to the current workdir.
It's up to you to decided whether or not to copy or move them to another path,
e.g. a path included in `PATH`.

### Install

You are now ready to provision the DC/OS cluster and install the Kubernetes package:

```bash
make deploy
```

Terraform will provision the infrastructure on your chosen cloud provider, and then proceed to install DC/OS. This should take somewhere around 15 minutes.

When DC/OS is up and running, the Kubernetes package installation will take place.

Wait until all tasks are running before trying to access the Kubernetes API.

You can watch the progress what was deployed so far with:

```bash
make watch-kubernetes-cluster
```

Below is an example of how it looks like when the install ran successfully:

```shell
Using Kubernetes cluster: dev/kubernetes01
deploy (serial strategy) (COMPLETE)
   etcd (serial strategy) (COMPLETE)
      etcd-0:[peer] (COMPLETE)
   control-plane (dependency strategy) (COMPLETE)
      kube-control-plane-0:[instance] (COMPLETE)
   mandatory-addons (serial strategy) (COMPLETE)
      mandatory-addons-0:[instance] (COMPLETE)
   node (dependency strategy) (COMPLETE)
      kube-node-0:[kubelet] (COMPLETE)
   public-node (dependency strategy) (COMPLETE)
```

You can access DC/OS Dashboard and check Kubernetes package tasks under Services:

```bash
make ui
```

### Exposing the Kubernetes API

Check the [exposing Kubernetes API doc](/services/kubernetes/__VERSION__/exposing_kubernetes_api.md) to understand how
the Kubernetes API gets exposed. To actually expose the Kubernetes API for the new Kubernetes cluster using Marathon-LB, run:

```bash
make marathon-lb
```

**NOTE:** If you have changed the `num_of_public_agents` to more than `1` in your `.deploy/desired_cluster_profile`, please scale `marathon-lb` service to the same number so that you can access Kubernetes API from any DC/OS public agent.

### Accessing the Kubernetes API

In order to access the Kubernetes API from outside the DC/OS cluster, one needs to configure `kubectl`, the Kubernetes CLI tool:

```bash
make kubeconfig
```

Let's test accessing the Kubernetes API and list the Kubernetes cluster nodes:

```bash
./kubectl --context devkubernetes01 get nodes
NAME                                                  STATUS   ROLES    AGE     VERSION
kube-control-plane-0-instance.devkubernetes01.mesos   Ready    master   5m18s   v1.12.2
kube-node-0-kubelet.devkubernetes01.mesos             Ready    <none>   2m58s   v1.12.2
```

And now, let's check how the system Kubernetes pods are doing:

```bash
./kubectl --context devkubernetes01 -n kube-system get pods
NAME                                                                          READY   STATUS    RESTARTS   AGE
calico-node-s9828                                                             2/2     Running   0          3m21s
calico-node-zc8qw                                                             2/2     Running   0          3m38s
coredns-6c7669957f-rvz85                                                      1/1     Running   0          3m38s
kube-apiserver-kube-control-plane-0-instance.devkubernetes01.mesos            1/1     Running   0          4m43s
kube-controller-manager-kube-control-plane-0-instance.devkubernetes01.mesos   1/1     Running   0          4m42s
kube-proxy-kube-control-plane-0-instance.devkubernetes01.mesos                1/1     Running   0          4m48s
kube-proxy-kube-node-0-kubelet.devkubernetes01.mesos                          1/1     Running   0          3m21s
kube-scheduler-kube-control-plane-0-instance.devkubernetes01.mesos            1/1     Running   0          4m26s
kubernetes-dashboard-5cbf45898-nkjsm                                          1/1     Running   0          3m37s
local-dns-dispatcher-kube-node-0-kubelet.devkubernetes01.mesos                1/1     Running   0          3m21s
metrics-server-594576c7d8-cb4pj                                               1/1     Running   0          3m35s
```

### Accessing the Kubernetes Dashboard

You will be able to access the Kubernetes Dashboard by running:

```bash
kubectl --context devkubernetes01 proxy
```

Then pointing your browser at: [http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/).

The Kubernetes Dashboard will show you the [login view](https://github.com/kubernetes/dashboard/wiki/Access-control#login-view).
In this screen you should choose the **Kubeconfig** option, click the **Choose kubeconfig file** text box and pick the location of your kubeconfig file (typically, `$HOME/.kube/config`).

The set of actions you can perform within the Kubernetes Dashboard using this login method is tied to the role or cluster role bound to the service account that is being used in the kubeconfig file you specify.
In particular, this means that if you have been granted the `cluster-admin` cluster role, you will have full administrative privileges on the target Kubernetes cluster.

## Uninstall Kubernetes

To uninstall the DC/OS Kubernetes package while leaving your DC/OS cluster up,
run:

```bash
make uninstall
```

**NOTE:** This will only uninstall Kubernetes. Make sure you destroy your DC/OS cluster using the instructions below when you finish testing or otherwise you will need to delete all cloud resources manually!

## Destroy cluster

To destroy the whole deployment:

```bash
make destroy
```

Last, clean generated resources:

```bash
make clean
```

That's it! For more information, have a look at different topics in the [Operations Sections](/services/kubernetes/__VERSION__/operations/).