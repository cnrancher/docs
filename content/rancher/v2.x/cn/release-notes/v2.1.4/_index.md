---
title: Release v2.1.4
weight: 15
---

# Release v2.1.4

With this release, the following [versions](https://rancher.com/docs/rancher/v2.x/en/installation/server-tags/) are now latest and stable:

|Type | Rancher Version | Docker Tag |Helm Repo| Helm Chart Version |
|---|---|---|---|---|
| Latest | v2.1.4 | `rancher/rancher:latest` | server-charts/latest | 2018.12.4 |
| Stable | v2.1.4 | `rancher/rancher:stable` | server-charts/stable | 2018.12.4 | 



## Known Major Issues

* Clusters created through Rancher can sometimes get stuck in provisioning [[#15970](https://github.com/rancher/rancher/issues/15970)] [[#15969](https://github.com/rancher/rancher/issues/15969)] [[#15695](https://github.com/rancher/rancher/issues/15695)]
* The upgrade for Rancher node-agent daemonset can sometimes get stuck due to pod removal failure on a Kubernetes side [[#16722](https://github.com/rancher/rancher/issues/16722)]


## Major Bug Fixes since v2.1.3

* Fixed an issue when upgrade for RKE clusters failed sometimes with "Failed to generate kube-etcd certificate: x509: cannot parse IP address of length 0" error [[16975](https://github.com/rancher/rancher/issues/16975)]
* Fixed an issue when in RKE provisioned clusters public access to API server had to be enabled in order to add a new worker[[16976](https://github.com/rancher/rancher/issues/16976)]
* Fixed a UI issue when Kubernetes version v1.12.3 wasn't labeled as experimental  [[17050](https://github.com/rancher/rancher/issues/17050)]


## Versions

> **NOTE** - Image Name Changes: Please note that as of v2.0.0, our images will be rancher/rancher and rancher/rancher-agent. If you are using v1.6, please continue to use rancher/server and rancher/agent.

### Images
- rancher/rancher:v2.1.4
- rancher/rancher-agent:v2.1.4

### Tools
- cli - [v2.0.6](https://github.com/rancher/cli/releases/tag/v2.0.6)
- rke - [v0.1.14](https://github.com/rancher/rke/releases/tag/v0.1.14)

### Kubernetes

-  [1.10.11](https://github.com/rancher/hyperkube/releases/tag/v1.10.11-rancher1)
-  [1.11.5](https://github.com/rancher/hyperkube/releases/tag/v1.11.5-rancher1) (default)
-  [1.12.3](https://github.com/rancher/hyperkube/releases/tag/v1.12.3-rancher1) (experimental)


## Upgrades and Rollbacks

Rancher supports both upgrade and rollback starting with v2.0.2.  Please note the version you would like to [upgrade](https://rancher.com/docs/rancher/v2.x/en/upgrades/) or [rollback](https://rancher.com/docs/rancher/v2.x/en/backups/rollbacks/) to change the Rancher version.

Due to the HA improvements introduced in the v2.1.0 release, the Rancher helm chart is the only supported method for installing or upgrading Rancher. Please use the Rancher helm chart to install HA Rancher. For details, see the [HA Install - Installation Outline](https://rancher.com/docs/rancher/v2.x/en/installation/ha/#installation-outline).

If you are currently using the RKE add-on install method, see [Migrating from a RKE add-on install](https://rancher.com/docs/rancher/v2.x/en/upgrades/upgrades/migrating-from-rke-add-on/) for details on how to move to using a helm chart.

**Any upgrade from a version prior to v2.0.3, when scaling up workloads, new pods will be created [[#14136](https://github.com/rancher/rancher/issues/14136)]** - In order to update scheduling rules for workloads [[#13527](https://github.com/rancher/rancher/issues/13527)], a new field was added to all workloads on `update`, which will cause any pods in workloads from previous versions to re-create. 

> **Note:** When rolling back, we are expecting you to rollback to the state at the time of your upgrade. Any changes post upgrade would not be reflected. In the case of rolling back using a [Rancher single-node install](https://rancher.com/docs/rancher/v2.x/en/installation/single-node-install/), you must specify the exact version you want to change the Rancher version to, rather than using the default `:latest` tag.

> **Note:** If you had the helm stable catalog enabled in v2.0.0, we've updated the catalog to start pointing directly to the Kubernetes helm repo instead of an internal repo. Please delete the custom catalog that is now showing up and re-enable the helm stable. [[#13582](https://github.com/rancher/rancher/issues/13582)]
