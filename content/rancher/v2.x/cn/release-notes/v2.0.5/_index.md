---
title: Release v2.0.5
weight: 6
---

## Versions

> **NOTE** - Image Name Changes: Please note that as of v2.0.0, our images will be `rancher/rancher` and `rancher/rancher-agent`. If you are using v1.6, please continue to use `rancher/server` and `rancher/agent`. 

- rancher/rancher:v2.0.5
- rancher/rancher-agent:v2.0.5

## Rancher Server Tags

Rancher server has 2 different tags. For each major release tag, we will provide documentation for the specific version.

- `rancher/rancher:latest` tag will be our latest development builds. These builds will have been validated through our CI automation framework. These releases are not meant for deployment in production.
- `rancher/rancher:stable` tag will be our latest stable release builds. This tag is the version that we recommend for production.  

Please do not use releases with a `rc{n}` suffix. These `rc` builds are meant for the Rancher team to test builds.

### Latest - v2.0.5 - `rancher/rancher:latest`

### Stable - v2.0.4 - `rancher/rancher:stable`

## Upgrades and Rollbacks

Rancher supports both upgrade and rollback starting with v2.0.2.  Please note the version you would like to [upgrade](https://www2.cnrancher.com/docs/rancher/v2.x/cn/upgrades/) or [rollback](https://www2.cnrancher.com/docs/rancher/v2.x/cn/backups-and-restoration/restorations/) to change the Rancher version.

**Any upgrade after v2.0.3, when scaling up workloads, new pods will be created [[#14136](https://github.com/rancher/rancher/issues/14136)]** - In order to update scheduling rules for workloads [[#13527](https://github.com/rancher/rancher/issues/13527)], a new field was added to all workloads on `update`, which will cause any pods in workloads from previous versions to re-create.
> **Note:** When rolling back, we are expecting you to rollback to the state at the time of your upgrade. Any changes post upgrade would not be reflected. In the case of rolling back using a [Rancher single-node install](https://www2.cnrancher.com/docs/rancher/v2.x/cn/installation/server-installation/single-node-install/), you must specify the exact version you want to change the Rancher version to, rather than using the default `:latest` tag.
> **Note:** If you had the helm stable catalog enabled in v2.0.0, we've updated the catalog to start pointing directly to the kubernetes helm repo instead of an internal repo. Please delete the custom catalog that is now showing up and re-enable the helm stable. [[#13582](https://github.com/rancher/rancher/issues/13582)]

## Enhancements

- **Support for OpenLDAP [[#13814](https://github.com/rancher/rancher/issues/13814)]** - openLDAP has been implemented as an authentication provider and can be selected as an option.
- **Support for FreeIPA [[#13815](https://github.com/rancher/rancher/issues/13815)]** - FreeIPA has been implemented as an authentication provider and can be selected as an option.
- **Cordon [[#13623](https://github.com/rancher/rancher/issues/13623)]** - Added the ability to cordon/uncordon nodes. When you update a node to cordon, no pods will be able to be scheduled on the node.
- **Azure AKS Enhancements  [[#14164](https://github.com/rancher/rancher/issues/14164)]** - Added ability to select a different resource group for AKS advanced networking options.
- **Rancher CLI** - to support Infrastructure as Code (IaC), two new commands have been added to the rancher CLI.
  - `rancher export` - allows the export of a managed Kubernetes cluster as a YAML definition file.
  - `rancher up` - allows the import of a YAML file for cluster creation within Rancher

## Known Major Issues

- After upgrade, ingresses may have some issues with showing 503s or inability to target new workloads, the current workaround is to re-create the ingress [[#13611](https://github.com/rancher/rancher/issues/13611)]

## Major Bug Fixes since v2.0.4

- Fixed an issue where adding in nested groups for AD was causing slower login times by making an option to disable searching using nested groups. [[#14364](https://github.com/rancher/rancher/issues/14364)]
- Fixed an issue where when standard users created clusters, they were unable to download keys for nodes [[#14024](https://github.com/rancher/rancher/issues/14024)]
- Fixed an issue where you couldn't update the name for registry, configmaps or secrets [[#14094](https://github.com/rancher/rancher/issues/14094)]
- Fixed an issue where attempting to configure OpenStack as a cloud provider wasn't working [[#14197](https://github.com/rancher/rancher/issues/14191)]
- Fixed an issue where you were unable to use CLI to install a local helm chart [[#14251](https://github.com/rancher/rancher/issues/14251)]
- Fixed an issue where it looks as if you can must change the password for users logged in as an external auth provider [[#14162](https://github.com/rancher/rancher/issues/14162)]

## Migrating from Rancher 1.6

Our next major milestone will be our 2.1 release that will include a tool that will convert Rancher Compose to Kubernetes YAML.  This will help our existing Cattle users from having to start the migration from scratch.  However, we know the biggest challenge will be having to leverage Cattle functionality in a Kubernetes environment as you deploy new workloads.  We plan to also release a guide that will act as a cheatsheet for those that enjoy Cattle's simplicity and want to quickly create those workloads in Kubernetes. 

## Rancher CLI Downloads

[Downloads](https://github.com/rancher/cli/releases/tag/v2.0.3)