---
title:  "Storage and Workload Separation in Kubernetes with Longhorn"
date:   2023-02-26 18:20:00 +0100
categories: 
  - k8s, longhorn, k3s

share: true
comments: true
classes: wide
author_profile: true
output: 
    html_document:
        fig_caption: yes
---
# Context
Let's say you have a k8s or a k3s cluster composed of 6 worker nodes, and for some reasons, you want to separate compute from storage in a way that some worker nodes run only `Longhorn` pods and take care of storage (volumes, replicas), while the rest of the worker nodes run any other pods and use the volumes that are on the other worker nodes.

For example, you want worker nodes 1 and 2 to run only `Longhorn` pods and create volumes that are ready to be consumed. worker nodes 3, 4, and 5 have no volumes on their disks, only pods, and consume the volumes from worker nodes 1 and 2. How do you do that?


## How
You can make this on an existing Longhorn or on a newly installed one.

Let's start: We call worker nodes 1 and 2, the storage nodes and the worker nodes 3, 4, and 5, the compute nodes.

### Configure the storage nodes

1. Label the storage nodes with the label `node.longhorn.io/create-default-disk=true`.

  ```
  kubectl label nodes node-1 node-2 node.longhorn.io/create-default-disk=true
  ```


2. Install `Longhorn` with the setting "Create Default Disk on Labeled Nodes" set to true. 

    if you're using Helm to install `longhron` you can set that value using :

    ```
    defaultSettings:
      # -- Setting that allows Longhorn to automatically create a default disk only on nodes with the label "node.longhorn.io/create-default-disk=true" (if no other disks exist). When this setting is disabled, Longhorn creates a default disk on each node that is added to the cluster.
      createDefaultDiskLabeledNodes: true
    ```

    If you're using another method, have a look at how to configure this setting, see https://longhorn.io/docs/archives/1.2.2/references/settings/#create-default-disk-on-labeled-nodes


Now, if you go to the `Longhorn` dashboard, you'll see that the compute nodes are disabled and have no disk.

📝📌 PS: If you've already installed `Longhorn` and want to configure this setting on a working `Longhorn`, you need to remove the compute nodes from the cluster and bring them back again to have this take effect.

### Rejecting no Longhron pods from being scheduled on the storage nodes

The current changes we made will only tell `Longhorn` to stop using the compute nodes for storage but will not restrict all the pods except `Longhorn`'s pods from being scheduled on the storage nodes. Taint and toleration to the rescue!

`Longhorn` components can be configured to tolerate taints. So, the idea is to taint the storage nodes and make only the `Longhorn` pods tolerate these taints. This will result in allowing only the `Longhorn` pods to be scheduled on the storage nodes and rejecting any other pod.

1. Apply the taint on the storage nodes

    ```
    kubectl taint nodes node-1 node-2 node=storage:NoSchedule
    ```
2. Make the Longhorn pods tolerate the taint
Just to not make this article long, depending on whether you installed `Longhorn` or are planning to install it and with which method, steps differ. So, follow up on the blog depending on your case. Link https://longhorn.io/docs/archives/1.2.2/advanced-resources/deploy/taint-toleration/#setting-up-taints-and-tolerations-after-longhorn-has-been-installed.


**Tips**: If you're trying this on an installed `Longhorn`, one of the steps in the link I provided is asking to make all volumes detached. An easy way is to run this command for every namespace you have:

- This will scale down all deployments to 0 in the namespace my-namespace.

  ```
  kubectl scale deployment -n my-namespace --replicas=0 --all
  ```

- This will scale down all stateful sets to 0 in the namespace my-namespace.
  ```
  kubectl scale statefulset -n my-namespace --replicas=0 --all
  ```
- When the setting is configured, you can bring them back by (if the replica count was 1 ofc):

  ```
  kubectl scale deployment -n my-namespace --replicas=1 --all
  ```
  ```
  kubectl scale statefulset -n my-namespace --replicas=1 --all
  ```

Now, you have node 1 and node 2 responsible only for storing volumes and replicas and nothing else, and nodes 3, 4, and 5 responsible only for your workload pods.