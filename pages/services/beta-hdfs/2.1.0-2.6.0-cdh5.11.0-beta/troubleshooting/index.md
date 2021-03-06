---
layout: layout.pug
navigationTitle: 
title: Troubleshooting
menuWeight: 90
excerpt:
featureMaturity:
enterprise: false
---

<!-- This source repo for this topic is https://github.com/mesosphere/dcos-commons -->


# Replacing a Permanently Failed Node
The DC/OS HDFS Service is resilient to temporary node failures. However, if a DC/OS agent hosting an HDFS node is permanently lost, manual intervention is required to replace the failed node. The following command should be used to replace the node residing on the failed server.

```bash
$ dcos beta-hdfs --name=<service-name> pod replace <node_id>
```

# Restarting a Node
If you must forcibly restart a node, use the following command to restart the node on the same agent node where it currently resides. This will not result in an outage or loss of data.

```bash
$ dcos beta-hdfs --name=<service-name> pod restart <node_id>
```
