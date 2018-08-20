---
title: 创建自定义集群
weight: 1
---

## 创建集群

1. From the **Clusters** page, click **Add Cluster**.

2. Choose **Custom**.

3. Enter a **Cluster Name**.

4. {{< step_create-cluster_member-roles >}}

5. {{< step_create-cluster_cluster-options >}}

6.	Click **Next**.

7.	From **Node Role**, choose the roles that you want filled by a cluster node.

	>**Bare-Metal Server Reminder:**
	>
	If you plan on dedicating bare-metal servers to each role, you must provision a bare-metal server for each role (i.e. provision multiple bare-metal servers).

8.	**Optional**: Add **Labels** to your cluster nodes to help schedule workloads later.

	[Kubernetes Documentation: Labels](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/)

9. Copy the command displayed on screen to your clipboard.

10. Log in to your Linux host using your preferred shell, such as PuTTy or a remote Terminal connection. Run the command copied to your clipboard.

	>**Note:** Repeat steps 7-10 if you want to dedicate specific hosts to specific node roles. Repeat the steps as many times as needed.

11. When you finish running the command(s) on your Linux host(s), click **Done**.

{{< result_create-cluster >}}

## Amazon Only:<br/>Tag Resources

If you have configured your cluster to use Amazon as **Cloud Provider**, tag your AWS resources with a cluster ID.

[Amazon Documentation: Tagging Your Amazon EC2 Resources](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html)

>**Note:** You can use Amazon EC2 instances without configuring a cloud provider in Kubernetes. You only have to configure the cloud provider if you want to use specific Kubernetes cloud provider functionality. For more information, see [Kubernetes Cloud Providers](https://kubernetes.io/docs/concepts/cluster-administration/cloud-providers/)


The following resources need to tagged with a `ClusterID`:

- **Nodes**: All hosts added in Rancher.
- **Subnet**: The subnet used for your cluster
- **Security Group**: The security group used for your cluster.

	>**Note:** Do not tag multiple security groups. Tagging multiple groups generates an error when creating Elastic Load Balancer.

The tag that should be used is:

```
Key=kubernetes.io/cluster/<CLUSTERID>, Value=owned
```

`<CLUSTERID>` can be any string you choose. However, the same string must be used on every resource you tag. Setting the tag value to `owned` informs the cluster that all resources tagged with the `<CLUSTERID>` are owned and managed by this cluster.

If you share resources between clusters, you can change the tag to:

```
Key=kubernetes.io/cluster/CLUSTERID, Value=shared
```
