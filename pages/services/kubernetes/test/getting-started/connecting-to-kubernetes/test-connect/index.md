---
layout: layout.pug
navigationTitle: Testing Connections
title: Testing Connections and Viewing the Kubernetes Dashboard
menuWeight: 10
excerpt: Learn to test your connection to private Kubernetes clusters and view the Kubernetes dashboard via public web proxy.
---

In this section, we will test our connections to our Kubernetes clusters one at a time and then lastly connect to the Kubernetes dashboard and view our Kubernetes clusters' information.

## Connect to each Kubernetes cluster via NGINX web proxy

To test our connections we must ensure that the right ports are open, set our cluster's context with the DC/OS Kubernetes service, get Kubernetes nodes' information, and test with a simple NGINX deployment.

### Check that Port `:6443` and `:6444` are open.

Before attempting to connect `kubectl` to the MKE clusters, you will want to check to ensure that ports `:6443` and `:6444` are accessible by your local machine to the DC/OS Cluster. Closed ports `:6443` and `:6444` will cause `kubectl` commands to just hang. Moreover, if using a cloud provider such as AWS, these would typically be rules configured in your **EC2-->Security Groups** tab.

### Test the connection to `kubernetes-cluster1` at port `:6443`

1. <strong>First, configure your</strong> `context`<strong> to</strong> `kubernetes-cluster1` <strong>at port </strong>`:6443`<strong>.</strong>

    In your CLI, enter the following snippet:

    ```bash
    dcos kubernetes cluster kubeconfig \
        --insecure-skip-tls-verify \
        --context-name=kubernetes-cluster1 \
        --cluster-name=kubernetes-cluster1 \
        --apiserver-url=https://${EDGELB_PUBLIC_AGENT_IP}:6443
    ```

    which will configure DC/OS Kubernetes to connect to `kubernetes-cluster1` from the Kubernetes CLI, `kubectl`.

1. <strong>Next, get your Kubernetes nodes' info for </strong>`kubernetes-cluster1`<strong>.</strong>

    In your CLI, enter:

    ```bash
    kubectl get nodes
    ```
    to receive details regarding your the nodes of `kubernetes-cluster1`.

    <!-- better validation here of this step:OUTPUT ^^ -->


1. <strong>Then, create a NGINX deployment on </strong>`kubernetes-cluster1`<strong>.</strong>

    Enter the following in your CLI:

    ```bash
    kubectl apply -f https://k8s.io/examples/application/deployment.yaml
    ```

    to launch the deployment.

1. <strong>View the NGINX deployment just launched.</strong>

    In your CLI, run:

    ```bash
    kubectl get deployments
    ```

    The resulting output should look similar to the following:

    ```bash
    $ kubectl get deployments
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   2         2         2            2           23s
    ```

1. <strong>Lastly, delete the NGINX deployment to finish up this connection test for</strong> `kubernetes-cluster1`<strong>.</strong>

    Run the following command in your CLI:

    ```bash
    kubectl delete deployment nginx-deployment
    ```

    and the deployment will be deleted.

### Test the connection to `kubernetes-cluster2` at port `:6444`

Now you will do the same to test `kubernetes`

1. <strong>Switch contexts for inbound</strong> `kubectl` <strong>commands to your second Kubernetes cluster </strong>`kubernetes-cluster2`<strong>.</strong>

    In your CLI, paste in the following and press run the command:

    ```bash
    dcos kubernetes cluster kubeconfig \
        --insecure-skip-tls-verify \
        --context-name=kubernetes-cluster2 \
        --cluster-name=kubernetes-cluster2 \
        --apiserver-url=https://${EDGELB_PUBLIC_AGENT_IP}:6444
    ```

    Your `kubectl` commands should now connect to `kubernetes-cluster2`.

    Any time you would like to switch to communicating with a different cluster using the DC/OS CLI, you will want to use a command with flags configured like this one, replacing `kubernetes-cluster2` with the name of the cluster service.

1. <strong>Then, like before for </strong>`kubernetes-cluster1`<strong>, get the nodes' information for </strong>`kubernetes-cluster2`<strong>.</strong>

    In your CLI, enter:

    ```bash
    kubectl get nodes
    ```

    and note that the output should show that you are now using `kubernetes-cluster2`, similar to the following:

    ```bash
    $ kubectl get nodes
    NAME                                                      STATUS   ROLES    AGE    VERSION
    kube-control-plane-0-instance.kubernetes-cluster2.mesos   Ready    master   145m   v1.12.1
    kube-node-0-kubelet.kubernetes-cluster2.mesos             Ready    <none>   142m   v1.12.1
    ```

1. <strong>Next, create a NGINX deployment on</strong> `kubernetes-cluster1`<strong>:</strong>

    Enter the following in your CLI:

    ```bash
    kubectl apply -f https://k8s.io/examples/application/deployment.yaml
    ```

    and the deployment will begin.

1. <strong>View that NGINX deployment in action:</strong>

    ```bash
    kubectl get deployments
    ```

    The resulting output should look similar to below:

    ```bash
    $ kubectl get deployments
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   2         2         2            2           23s
    Delete NGINX deployment:
    ```

1. <strong>Finally, delete your test deployment.</strong>

    Simply enter:

    ```bash
    kubectl delete deployment nginx-deployment
    ```

    and the deployment will be shut down.

## Access the Kubernetes Dashboard from your browser

In one simple but satisfying validation of our connection into our Kubernetes cluster manager from outside the cluster, we will access the Kubernetes Dashboard and view our clusters.

1. <strong> Access the Kubernetes dashboard.</strong>

    In your CLI, enter:

    ```bash
    kubectl proxy
    ```

    which should start the proxy server.

    <!-- better validation here of this step:OUTPUT ^^ -->

1. <strong>Then, navigate to:</strong>  [http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/) and you should see your Kubernetes dashboard and both clusters' information.

    <!-- better validation here of this step:Screenshot ^^ -->

# Mission Complete!

Well done! You have successfully completed the [Getting Started Guide for Kubernetes](/services/kubernetes/test/getting-started/). You have set up your DC/OS Enterprise cluster to be able to run Kubernetes as a service on DC/OS.

For more information regarding operating Kubernetes as a service on DC/OS, check out the [Operating](/services/kubernetes/test/operating/) section.
