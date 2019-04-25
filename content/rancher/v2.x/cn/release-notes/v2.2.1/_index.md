---
title: Release v2.2.1
weight: 201
---


This release comes with the latest Kubernetes versions  - i.e. v1.11.9, v1.12.7, v1.13.5 - for clusters launched by Rancher to address Kubernetes [CVE-2019-9946](https://github.com/kubernetes/kubernetes/pull/75455) and [CVE-2019-1002101](https://github.com/kubernetes/kubernetes/pull/75037). We recommend upgrading your Kubernetes clusters to these versions. 

With this release, the following versions are now latest and stable:

|Type | Rancher Version | Docker Tag |Helm Repo| Helm Chart Version |
|---|---|---|---|---|
| Latest | v2.2.1 | `rancher/rancher:stable (或者rancher/rancher:latest)` | server-charts/latest | 2.2.1 |
| Stable | v2.1.8 | `rancher/rancher:stable` | server-charts/stable | 2.1.8 | 

**`v2.2.1`** or a patched version of it will be promoted to stable after a period of hardening and bake-in in our open source community.

Please review our [version documentation](https://rancher.com/docs/rancher/v2.x/en/installation/server-tags/) for more details on versioning and tagging conventions.

## Major Bug Fixed Since v2.2.0

* Added Kubernetes versions (i.e. v1.11.9, v1.12.7, v1.13.5) - for clusters launched by Rancher to address Kubernetes [CVE-2019-9946](https://github.com/kubernetes/kubernetes/pull/75455) and [CVE-2019-1002101](https://github.com/kubernetes/kubernetes/pull/75037). We recommend upgrading your Kubernetes clusters to these versions. 

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
- rancher/rancher:v2.2.1
- rancher/rancher-agent:v2.2.1

### Tools
- cli - [v2.2.0](https://github.com/rancher/cli/releases/tag/v2.2.0)
- rke - [v0.2.0](https://github.com/rancher/rke/releases/tag/v0.2.0)

### Kubernetes

-  [1.11.9](https://github.com/rancher/hyperkube/releases/tag/v1.11.9-rancher1)
-  [1.12.7](https://github.com/rancher/hyperkube/releases/tag/v1.12.7-rancher1) 
-  [1.13.5](https://github.com/rancher/hyperkube/releases/tag/v1.13.5-rancher1) (default)


## Upgrades and Rollbacks

Rancher supports both upgrade and rollback starting with v2.0.2.  Please note the version you would like to [upgrade](https://rancher.com/docs/rancher/v2.x/en/upgrades/) or [rollback](https://rancher.com/docs/rancher/v2.x/en/backups/rollbacks/) to change the Rancher version.

Due to the HA improvements introduced in the v2.1.0 release, the Rancher helm chart is the only supported method for installing or upgrading Rancher. Please use the Rancher helm chart to install HA Rancher. For details, see the [HA Install - Installation Outline](https://rancher.com/docs/rancher/v2.x/en/installation/ha/#installation-outline).

If you are currently using the RKE add-on install method, see [Migrating from a RKE add-on install](https://rancher.com/docs/rancher/v2.x/en/upgrades/upgrades/migrating-from-rke-add-on/) for details on how to move to using a helm chart.

**Any upgrade from a version prior to v2.0.3, when scaling up workloads, new pods will be created [[#14136](https://github.com/rancher/rancher/issues/14136)]** - In order to update scheduling rules for workloads [[#13527](https://github.com/rancher/rancher/issues/13527)], a new field was added to all workloads on `update`, which will cause any pods in workloads from previous versions to re-create. 

**Note:** When rolling back, we are expecting you to rollback to the state at the time of your upgrade. Any changes post upgrade would not be reflected. In the case of rolling back using a [Rancher single-node install](https://rancher.com/docs/rancher/v2.x/en/installation/single-node-install/), you must specify the exact version you want to change the Rancher version to, rather than using the default `:latest` tag.

**Note:** If you had the helm stable catalog enabled in v2.0.0, we've updated the catalog to start pointing directly to the Kubernetes helm repo instead of an internal repo. Please delete the custom catalog that is now showing up and re-enable the helm stable. [[#13582](https://github.com/rancher/rancher/issues/13582)]
