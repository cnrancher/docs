---
title: Release v2.0.11
weight: 12
---

## Important
This release addresses two security vulnerabilities found in Rancher. The first vulnerability allows users in the Default project of a cluster to escalate privileges to that of a cluster admin through a service account. The second vulnerability allows members to have continued access to create, update, read, and delete namespaces in a project after they have been removed from it. You can view the official CVEs here [CVE-2018-20321](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2018-20321) and here [CVE-2019-6287](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-6287).

For a more detailed explanation of the CVEs and how we've addressed them, you can read our [blog article](https://rancher.com/blog/2019/2019-01-29-explaining-security-vulnerabilities-addressed-in-rancher-v2-1-6-and-v2-0-11/).

## Versions

> **NOTE** - Image Name Changes: Please note that as of v2.0.0, our images will be `rancher/rancher` and `rancher/rancher-agent`. If you are using v1.6, please continue to use `rancher/server` and `rancher/agent`. 

- rancher/rancher:v2.0.11
- rancher/rancher-agent:v2.0.11

## Rancher Server Tags

Rancher server has 2 different tags. For each major release tag, we will provide documentation for the specific version.
- `rancher/rancher:stable (或者rancher/rancher:latest)` tag will be our latest development builds. These builds will have been validated through our CI automation framework. These releases are not meant for deployment in production.
- `rancher/rancher:stable` tag will be our latest stable release builds. This tag is the version that we recommend for production.  

Please do not use releases with a `rc{n}` suffix. These `rc` builds are meant for the Rancher team to test builds.

#### Latest - v2.1.6 - `rancher/rancher:stable (或者rancher/rancher:latest)`
#### Stable - v2.1.6 - `rancher/rancher:stable`

## Upgrades and Rollbacks

### IMPORTANT: v2.0.11 specific rollback instructions
Because the fix for CVE-2018-20321 involves a data migration (deleting a service account and creating it elsewhere), rolling Rancher back from v2.0.11 to a version prior to the patch is more complicated than usual. We have documented these steps [here](http://rancher.com/docs/rancher/v2.x/en/upgrades/rollbacks/). Review these steps prior to upgrading so that you understand their implications.

### Standard upgrade and rollback notes:
Rancher supports both upgrade and rollback starting with v2.0.2.  Please note the version you would like to [upgrade](https://rancher.com/docs/rancher/v2.x/en/upgrades/) or [rollback](https://rancher.com/docs/rancher/v2.x/en/backups/rollbacks/) to change the Rancher version.

**Any upgrade after v2.0.3, when scaling up workloads, new pods will be created [[#14136](https://github.com/rancher/rancher/issues/14136)]** - In order to update scheduling rules for workloads [[#13527](https://github.com/rancher/rancher/issues/13527)], a new field was added to all workloads on `update`, which will cause any pods in workloads from previous versions to re-create. 

> **Note:** When rolling back, we are expecting you to rollback to the state at the time of your upgrade. Any changes post upgrade would not be reflected. In the case of rolling back using a [Rancher single-node install](https://rancher.com/docs/rancher/v2.x/en/installation/single-node-install/), you must specify the exact version you want to change the Rancher version to, rather than using the default `:latest` tag.

## Known Major Issues
The known issues for this release remain unchanged from v2.0.10:
* Sometimes new Kubernetes version doesn't get updated right away on the upgraded Kubernetes clusters; it gets fixed as soon as user application gets deployed on the node [[15831](https://github.com/rancher/rancher/issues/15831)]

## Major Bug Fixes since v2.0.10
*  Addressed CVE-2018-20321 that allowed users in the Default project of a cluster to escalate privileges to that of a cluster admin through a service account. [[#17725](https://github.com/rancher/rancher/issues/17725)]
*  Addressed CVE-2019-6287 that allowed members to have continued access to create, update, read, and delete namespaces in a project after they had been removed from it.  [[#17724](https://github.com/rancher/rancher/issues/17724), [#17244](https://github.com/rancher/rancher/issues/17244)]

## Rancher CLI Downloads

https://github.com/rancher/cli/releases/tag/v2.0.4