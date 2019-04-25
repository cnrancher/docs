---
title: Release v2.1.6
weight: 107
---

# Release v2.1.6

## Important
This release addresses two security vulnerabilities found in Rancher. The first vulnerability allows users in the Default project of a cluster to escalate privileges to that of a cluster admin through a service account. The second vulnerability allows members to have continued access to create, update, read, and delete namespaces in a project after they have been removed from it. You can view the official CVEs here [CVE-2018-20321](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20321) and here [CVE-2019-6287](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-6287).

For a more detailed explanation of the CVEs and how we've addressed them, you can read our [blog article](https://rancher.com/blog/2019/2019-01-29-explaining-security-vulnerabilities-addressed-in-rancher-v2-1-6-and-v2-0-11/).

With this release, the following [versions](https://rancher.com/docs/rancher/v2.x/en/installation/server-tags/) are now latest and stable:

|Type | Rancher Version | Docker Tag |Helm Repo| Helm Chart Version |
|---|---|---|---|---|
| Latest | v2.1.6 | `rancher/rancher:stable (或者rancher/rancher:latest)` | server-charts/latest | 2019.1.2  |
| Stable | v2.1.6 | `rancher/rancher:stable` | server-charts/stable | 2019.1.2  | 

## Known Major Issues

The known issues for this release remain unchanged from v2.1.5:
* Clusters created through Rancher can sometimes get stuck in provisioning [[#15970](https://github.com/rancher/rancher/issues/15970)] [[#15969](https://github.com/rancher/rancher/issues/15969)] [[#15695](https://github.com/rancher/rancher/issues/15695)]
* The upgrade for Rancher node-agent daemonset can sometimes get stuck due to pod removal failure on a Kubernetes side [[#16722](https://github.com/rancher/rancher/issues/16722)]

## Major Bug Fixes since v2.1.5
*  Addressed CVE-2018-20321 that allowed users in the Default project of a cluster to escalate privileges to that of a cluster admin through a service account. [[#17725](https://github.com/rancher/rancher/issues/17725)]
*  Addressed CVE-2019-6287 that allowed members to have continued access to create, update, read, and delete namespaces in a project after they had been removed from it. [[#17724](https://github.com/rancher/rancher/issues/17724), [#17244](https://github.com/rancher/rancher/issues/17244)]

## Versions

> **NOTE** - Image Name Changes: Please note that as of v2.0.0, our images will be rancher/rancher and rancher/rancher-agent. If you are using v1.6, please continue to use rancher/server and rancher/agent.

### Images
- rancher/rancher:v2.1.6
- rancher/rancher-agent:v2.1.6

### Tools
- cli - [v2.0.6](https://github.com/rancher/cli/releases/tag/v2.0.6)
- rke - [v0.1.15](https://github.com/rancher/rke/releases/tag/v0.1.15) 

### Kubernetes

-  [1.10.12](https://github.com/rancher/hyperkube/releases/tag/v1.10.12-rancher1)
-  [1.11.6](https://github.com/rancher/hyperkube/releases/tag/v1.11.6-rancher1) (default)
-  [1.12.4](https://github.com/rancher/hyperkube/releases/tag/v1.12.4-rancher1) (experimental)


## Upgrades and Rollbacks

### IMPORTANT: v2.1.6 specific rollback caveats and instructions
Because the fix for CVE-2018-20321 involves a data migration (deleting a service account and creating it elsewhere), rolling Rancher back from v2.1.6 to a version prior to the patch is more complicated than usual. We have documented the extra steps [here](http://rancher.com/docs/rancher/v2.x/en/upgrades/rollbacks/). Review these steps prior to upgrading so that you understand their implications.

### Standard upgrade and rollback notes:
The following information regarding upgrades and rollbacks remains unchanged from v2.1.5:

Rancher supports both upgrade and rollback starting with v2.0.2.  Please note the version you would like to [upgrade](https://rancher.com/docs/rancher/v2.x/en/upgrades/) or [rollback](https://rancher.com/docs/rancher/v2.x/en/backups/rollbacks/) to change the Rancher version.

Due to the HA improvements introduced in the v2.1.0 release, the Rancher helm chart is the only supported method for installing or upgrading Rancher. Please use the Rancher helm chart to install HA Rancher. For details, see the [HA Install - Installation Outline](https://rancher.com/docs/rancher/v2.x/en/installation/ha/#installation-outline).

If you are currently using the RKE add-on install method, see [Migrating from a RKE add-on install](https://rancher.com/docs/rancher/v2.x/en/upgrades/upgrades/migrating-from-rke-add-on/) for details on how to move to using a helm chart.

**Any upgrade from a version prior to v2.0.3, when scaling up workloads, new pods will be created [[#14136](https://github.com/rancher/rancher/issues/14136)]** - In order to update scheduling rules for workloads [[#13527](https://github.com/rancher/rancher/issues/13527)], a new field was added to all workloads on `update`, which will cause any pods in workloads from previous versions to re-create. 

> **Note:** When rolling back, we are expecting you to rollback to the state at the time of your upgrade. Any changes post upgrade would not be reflected. In the case of rolling back using a [Rancher single-node install](https://rancher.com/docs/rancher/v2.x/en/installation/single-node-install/), you must specify the exact version you want to change the Rancher version to, rather than using the default `:latest` tag.

> **Note:** If you had the helm stable catalog enabled in v2.0.0, we've updated the catalog to start pointing directly to the Kubernetes helm repo instead of an internal repo. Please delete the custom catalog that is now showing up and re-enable the helm stable. [[#13582](https://github.com/rancher/rancher/issues/13582)]
