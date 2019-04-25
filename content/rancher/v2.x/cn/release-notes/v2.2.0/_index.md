---
title: Release v2.2.0
weight: 200
---


Rancher v2.2.0 focuses on improving supportability and operability of your Kubernetes clusters . New features like automated etcd backup and restore, a multi-tenant catalog, ability to create and manage global DNS, advanced monitoring with Promethues and multi-cluster apps make it easy to support enterprise grade, production ready Kubernetes clusters. 

With this release, the following versions are now latest and stable:

| Type   | Rancher Version | Docker Tag               | Helm Repo            | Helm Chart Version |
| ------ | --------------- | ------------------------ | -------------------- | ------------------ |
| Latest | v2.2.0          | `rancher/rancher:stable (或者rancher/rancher:latest)` | server-charts/latest | 2.2.0              |
| Stable | v2.1.7          | `rancher/rancher:stable` | server-charts/stable | 2.1.7              |

**`v2.2.0`** or a patched version of it will be promoted to stable after a period of hardening and bake-in in our open source community.

Please review our [version documentation](https://rancher.com/docs/rancher/v2.x/en/installation/server-tags/) for more details on versioning and tagging conventions.

Below is a list of the major items addressed in this release. For a comprehensive list of all items, see the [v2.2 GitHub Milestone](https://github.com/rancher/rancher/milestone/140?closed=1). 


## Features and Enhancements

* **[Rancher Advanced Monitoring](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/tools/monitoring/) [[#14230](https://github.com/rancher/rancher/issues/14230)]** - Monitoring of clusters, projects and k8s components is now supported through integration with Prometheus. By leveraging Prometheus, Rancher is able to provide a time series view into your data, and more sophisticated collection of stats and metric types beyond the basic Rancher monitoring. Multi-tenancy support in terms of cluster and project-only Prometheus instances is supported. Integrated Grafana is included for better visualization.
* **[Multi-Cluster Apps](https://rancher.com/docs/rancher/v2.x/en/catalog/multi-cluster-apps/) [[#16016](https://github.com/rancher/rancher/issues/16016)]** - Rancher now supports deploying, managing, and upgrading applications across multiple clusters and projects. By augmenting the functionality of Helm with Rancher's multi-cluster management capabilities, users are now able to seamlessly manage their applications across clusters. Use this feature for high availability, disaster recovery, or ensuring your clusters are all running a common set of applications.
* **[Global DNS](https://rancher.com/docs/rancher/v2.x/en/catalog/globaldns/) [[#16014](https://github.com/rancher/rancher/issues/16014)]** - Rancher now supports configuring and managing DNS endpoints across clusters and projects. By combining Rancher's multi-cluster management capabilities with Kubernetes' external-dns controller, users now have unprecedented control over managing the DNS entries of their applications across availability zones, data centers, and even cloud providers. We support Route53 and AliDNS, and have alpha support for CloudFlare. Global DNS is only available in HA setups with the local cluster enabled.
* **[Multi-Tenant Catalogs](https://rancher.com/docs/rancher/v2.x/en/catalog/#catalog-scopes) [[#14990](https://github.com/rancher/rancher/issues/14990)]** - Rancher now allows custom catalogs to be configured and managed at both a [cluster](https://rancher.com/docs/rancher/v2.x/en/catalog/custom/adding/#adding-cluster-catalogs) and [project](https://rancher.com/docs/rancher/v2.x/en/catalog/custom/adding/#adding-project-level-catalogs) level. This allows admins and users more flexibility to specify catalogs that users have access to deploy from. It also makes catalogs more private and hence secure for teams working on sensitive projects.
* **Cluster [Backup](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/backing-up-etcd/) and [Restoration](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/restoring-etcd/)  [[#13688](https://github.com/rancher/rancher/issues/13688)]** - For Rancher launched Kubernetes clusters, perform scheduled and ad hoc backups of etcd on local storage, or any S3-compatible object store. Users can view, manage and restore these snapshots through the Rancher UI.
* **[Support for Certificate Rotation for Kubernetes Components](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/certificate-rotation/) [[#16795](https://github.com/rancher/rancher/issues/16795)]** - For Rancher launched Kubernetes clusters, you can now rotate the certificates for the Kubernetes system components, which allows you to easily update the certificates if they are set to expire or have been compromised. Note for clusters created previously in Rancher, the certificates had a 1 year expiration, but new clusters will be created with a 10 year expiration. 
* **[Support for managing node template credentials](https://rancher.com/docs/rancher/v2.x/en/user-settings/cloud-credentials/) [[#15696](https://github.com/rancher/rancher/issues/15696)]** - Users can now decouple node templates from the credentials used to provision nodes with them, making their infrastructure more secure and flexible.
* **[Expanded support for hosted Kubernetes offerings](https://rancher.com/docs/rancher/v2.x/en/admin-settings/drivers/cluster-drivers/) [[#12698](https://github.com/rancher/rancher/issues/12698)] [[#16359](https://github.com/rancher/rancher/issues/16359)] [[#16357](https://github.com/rancher/rancher/issues/16357)] [[#16358](https://github.com/rancher/rancher/issues/16358)]** - Hosted Kubernetes drivers are now pluggable so that third-party providers can integrate with Rancher. We've bundled in drivers for the Kubernetes offerings from [Huawei](https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/hosted-kubernetes-clusters/cce/), [Tencent](https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/hosted-kubernetes-clusters/tke/), and [Aliyun](https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/hosted-kubernetes-clusters/ack/) as three new options.
* **Enhanced support for [GKE](https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/hosted-kubernetes-clusters/gke/), [EKS](https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/hosted-kubernetes-clusters/eks/), and [AKS](https://rancher.com/docs/rancher/v2.x/en/cluster-provisioning/hosted-kubernetes-clusters/aks/) cluster provisioning [[#14634](https://github.com/rancher/rancher/issues/14634)] [[#13910](https://github.com/rancher/rancher/issues/13910)] [[#15697](https://github.com/rancher/rancher/issues/15697)]** - Cluster owners now have greater control and flexibility when provisioning clusters with these providers.
* **[Support for Okta authentication](https://rancher.com/docs/rancher/v2.x/en/admin-settings/authentication/okta/) [[#15574](https://github.com/rancher/rancher/issues/15574)]** - Administrators can now configure Okta as a SAML authentication provider.
* **[Support for accessing clusters without proxying through Rancher](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/kubeconfig/#accessing-rancher-launched-kubernetes-clusters-without-rancher-server-running) [[#13698](https://github.com/rancher/rancher/issues/13698)]** - Cluster owners can now configure Rancher launched Kubernetes clusters to be accessed directly and still use Rancher authentication. This reduces the Rancher server's role as a critical point of failure and improves access to clusters that are geographically far from the Rancher server. 
* **[Support for selecting Weave as a networking provider](https://rancher.com/docs/rancher/v2.x/en/faq/networking/cni-providers/#weave) [[#12909](https://github.com/rancher/rancher/issues/12909)]** - Cluster owners can select Weave as a networking provider when provisioning Rancher launched Kubernetes clusters through Rancher. 
* **[Support for Bitbucket in pipelines](https://rancher.com/docs/rancher/v2.x/en/project-admin/tools/pipelines/#version-control-providers) [[#12625](https://github.com/rancher/rancher/issues/12625)]** - Users can now select Bitbucket as their version control provider when configuring pipelines.
* **Support for Linode and Cloud.ca as optional [node providers](https://rancher.com/docs/rancher/v2.x/en/admin-settings/drivers/node-drivers/) [[#17968](https://github.com/rancher/rancher/issues/17968)] [[#15444](https://github.com/rancher/rancher/issues/15444)]** - Administrators can now enable Linode and Cloud.ca as node drivers for use when provisioning Rancher launched Kubernetes clusters.

## Experimental Feature
* **[Experimental support for ARM64](https://rancher.com/docs/rancher/v2.x/en/installation/arm64-platform/) [[#16461](https://github.com/rancher/rancher/issues/16461)]** - Users can now manage ARM64 clusters with Rancher. Not all Rancher features currently work with ARM64 clusters. Monitoring, alerts, logging, and many items in the bundled Helm catalogs are known to not work.

## Major Bug Fixed Since v2.1.x
We've addressed over [180 bugs in this release](https://github.com/rancher/rancher/issues?utf8=%E2%9C%93&q=is%3Aissue+milestone%3Av2.2+label%3Akind%2Fbug). The following is a list of the major bugs fixed:
* Fixed several issues where Helm catalog answers were not being interpreted correctly, leading to failed application deployments [[#17122](https://github.com/rancher/rancher/issues/17122)] [[#13158](https://github.com/rancher/rancher/issues/13158)] [[#16680](https://github.com/rancher/rancher/issues/16680)] [[#14608](https://github.com/rancher/rancher/issues/14608)]
* Fixed several issues where poor catalog management was causing Rancher to consume too many resources and too much space in etcd. [[#14322](https://github.com/rancher/rancher/issues/14322)] [[#18217](https://github.com/rancher/rancher/issues/18217)] [[#16076](https://github.com/rancher/rancher/issues/16076)] [[#15600](https://github.com/rancher/rancher/issues/15600)] [[#13380](https://github.com/rancher/rancher/issues/13380)] [[#17330](https://github.com/rancher/rancher/issues/17330)]
* Fixed an issue where Rancher server would break if it was ran behind Amazon's Application Load Balancher (ALB). [[#14931](https://github.com/rancher/rancher/issues/14931)]
* Fixed an issue where networking between hosts in a cluster could be lost when using Flannel as the network provider. [[#13644](https://github.com/rancher/rancher/issues/13644)]
* Fixed an issue where using the Fluentd logging driver could lead to Rancher leaking file descriptors and causing clusters to become unavailable. [[#14987](https://github.com/rancher/rancher/issues/14987)]
* Fixed an issue where deleting a namespace with audit logging enabled would cause kubectl to hang indefinitely. [[#17027](https://github.com/rancher/rancher/issues/17027)]
* Fixed an issues where specifying a custom cloud provider configuration while provisiong an RKE cluster would lead to kubelets constantly restarting. [[#17502](https://github.com/rancher/rancher/issues/17502)]
* Fixed an issue where tolerations could not be added to a workload in the Rancher UI. [[#15585](https://github.com/rancher/rancher/issues/15585)]
* Fixed an issue where using the AWS cloud provider could lead to hitting a security group rule limit and failure to program additional ingress rules. Users can now specify a custom Kubernetes cloud provider configuration to prevent this. [[#17345](https://github.com/rancher/rancher/issues/17345)]
* Improved stability of the connection to AKS clusters provisioned through Rancher. [[#14354](https://github.com/rancher/rancher/issues/14354)]

## Important Notes

### Server Chart Versioning Change
We've changed the Rancher Server chart versioning scheme from the "datestamp" style (2019.1.x) used in v2.1 to versions that match the Rancher Server release (2.1.x).  With the release of 2.2.0, the previous "datestamp" style releases have been removed from the repo and replaced by charts versioned in step with the Rancher Server version. 

In the process of creating security updates for the 2.0.x series, we discovered that the "datestamp" style of versioning we had adopted with the 2.1.x release would not allow us to easily create security updates to the 2.1.x series once we released 2.2.0.

For more details and history of our Rancher Server chart versioning, please see this issue [#17663](https://github.com/rancher/rancher/issues/17663).

### Additional Steps Required for Air Gap Installations and Upgrades
In this release, we've introduce a "system catalog" for managing microservices that Rancher deploys for certrain features such as Global DNS, Alerts, and Monitoring. These additional steps are documented as part of [air gap installation instructions](https://rancher.com/docs/rancher/v2.x/en/installation/air-gap-high-availability/).

### Docs Changes
With this release, the docs navigation has been updated for better usability. Please refer to the [docs release notes](https://github.com/rancher/docs/releases/tag/v2.2.0) for more details. 

### RKE Add-on Install is only Supported up to Rancher v2.0.8
Due to the HA improvements introduced in the v2.1.0 release, the Rancher helm chart is the only supported method for installing or upgrading Rancher. Please use the Rancher helm chart to install HA Rancher. For details, see the [HA Install - Installation Outline](https://rancher.com/docs/rancher/v2.x/en/installation/ha/#installation-outline).

If you are currently using the RKE add-on install method, see [Migrating from a RKE add-on install](https://rancher.com/docs/rancher/v2.x/en/upgrades/upgrades/migrating-from-rke-add-on/) for details on how to move to using a helm chart.

### Known Major Issues
- Users wishing to utilize the new backup and restore functionality must edit their clusters, explicitly enable the feature, and save the changes. The backup functionality exposed in v2.1 is now considered legacy, but it will continue to function until you explicitly edit and save your cluster. [[#18754](https://github.com/rancher/rancher/issues/18754)]
- Manual backups are deleted based on retention count configuration settings and recurring snapshot creation time is impacted by taking a manual snapshot. [[#18807](https://github.com/rancher/rancher/issues/18807)]
- If snapshotting is disabled, users cannot restore from existing backups or take manual backups. [#18793](https://github.com/rancher/rancher/issues/18793)
- Cannot rotate CA certificate for RKE clusters. The CA certificate has a 10 year expiration, so it is not in danger of expiring in the near future. [[19050](https://github.com/rancher/rancher/issues/19050)] 
- Global DNS entries are not properly updated when a node that was hosting an associated ingress becomes unavailable. A records to the unavailable hosts will remain on the ingress and in the DNS entry. [[#18932](https://github.com/rancher/rancher/issues/18932)]
- When viewing a Global DNS entry that targets a multi-cluster application, if the user doesn't have access to all of the projects that are a part of the mult-cluster application, they will see broken links to those projects. Clicking on the broken links will log the user out of Rancher. [[#19174](https://github.com/rancher/rancher/issues/19174)]
- Deactivating or removing the creator of a cluster will prevent the monitoring feature from deploying successfully. In the case of the "local" cluster, this is the default admin account. [[#18787](https://github.com/rancher/rancher/issues/18787)]
- The monitoring feature reserves resources such as CPU and memory. It will fail to deploy if the cluster does not have sufficient resources available for reservation. See [our documentation](https://rancher.com/docs/rancher/v2.x/en/cluster-admin/tools/monitoring/#resource-consumption) for the recommended resource reservations you should make when enabling monitoring. [[#19078](https://github.com/rancher/rancher/issues/19078)]

## Versions

### Images
- rancher/rancher:v2.2.0
- rancher/rancher-agent:v2.2.0

### Tools
- cli - [v2.2.0](https://github.com/rancher/cli/releases/tag/v2.2.0)
- rke - [v0.2.0](https://github.com/rancher/rke/releases/tag/v0.2.0)

### Kubernetes

-  [1.11.8](https://github.com/rancher/hyperkube/releases/tag/v1.11.8-rancher1)
-  [1.12.6](https://github.com/rancher/hyperkube/releases/tag/v1.12.6-rancher1) 
-  [1.13.4](https://github.com/rancher/hyperkube/releases/tag/v1.13.4-rancher1) (default)


## Upgrades and Rollbacks

Rancher supports both upgrade and rollback starting with v2.0.2.  Please note the version you would like to [upgrade](https://rancher.com/docs/rancher/v2.x/en/upgrades/) or [rollback](https://rancher.com/docs/rancher/v2.x/en/backups/rollbacks/) to change the Rancher version.

Due to the HA improvements introduced in the v2.1.0 release, the Rancher helm chart is the only supported method for installing or upgrading Rancher. Please use the Rancher helm chart to install HA Rancher. For details, see the [HA Install - Installation Outline](https://rancher.com/docs/rancher/v2.x/en/installation/ha/#installation-outline).

If you are currently using the RKE add-on install method, see [Migrating from a RKE add-on install](https://rancher.com/docs/rancher/v2.x/en/upgrades/upgrades/migrating-from-rke-add-on/) for details on how to move to using a helm chart.

**Any upgrade from a version prior to v2.0.3, when scaling up workloads, new pods will be created [[#14136](https://github.com/rancher/rancher/issues/14136)]** - In order to update scheduling rules for workloads [[#13527](https://github.com/rancher/rancher/issues/13527)], a new field was added to all workloads on `update`, which will cause any pods in workloads from previous versions to re-create. 

**Note:** When rolling back, we are expecting you to rollback to the state at the time of your upgrade. Any changes post upgrade would not be reflected. In the case of rolling back using a [Rancher single-node install](https://rancher.com/docs/rancher/v2.x/en/installation/single-node-install/), you must specify the exact version you want to change the Rancher version to, rather than using the default `:latest` tag.

**Note:** If you had the helm stable catalog enabled in v2.0.0, we've updated the catalog to start pointing directly to the Kubernetes helm repo instead of an internal repo. Please delete the custom catalog that is now showing up and re-enable the helm stable. [[#13582](https://github.com/rancher/rancher/issues/13582)]