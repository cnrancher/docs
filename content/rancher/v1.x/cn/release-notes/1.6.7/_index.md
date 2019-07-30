---
title: Release v1.6.7
weight: 3000
---

## Important

This release addresses some bugs found in v1.6.6. Those fixes are available in v1.6.6 by upgrading the appropriate infrastructure stacks (Rancher Scheduler and Rancher Container Crontab). The 1.6.7 release is primarily for users in air-gapped networks that cannot dynamically update their catalogs and must rely on the catalogs packaged in the server image.

## Versions
- rancher/server:v1.6.7
- rancher/agent:v1.2.5
- rancher/lb-service-haproxy:v0.7.6
- [rancher-v0.6.3](https://github.com/rancher/cli/releases/tag/v0.6.3)
- [rancher-compose-v0.12.5](https://github.com/rancher/rancher-compose/releases/tag/v0.12.5)

### Supported Docker Versions

* Docker 1.12.3-1.12.6
* Docker 1.13.1
* Docker 17.03.0-ce/ee 
* Docker 17.06.0-ce/ee 

> Note: Kubernetes only supports up to Docker 1.12.6 

## Rancher Server Tags

Rancher Server has 2 different tags. For each major release tag, we will provide documentation for the specific version.
- `rancher/server:latest` tag will be our latest development builds. These builds will have been validated through our CI automation framework. These releases are not meant for deployment in production.
- `rancher/server:stable` tag will be our latest stable release builds. This tag is the version that we recommend for production.  

Please do not the releases with a `rc{n}` suffix. These `rc` builds are meant for the Rancher team to test out builds.

### Beta - v1.6.7 - `rancher/server:latest`
### Stable - v1.6.7 - `rancher/server:stable` (as of 8/14/17) 

## Important - Upgrade
* **Users on a version prior to Rancher v1.5.0:** We will automatically upgrade the `network-services` infrastructure stack as without this upgrade, your release will not work. 
* **Users on a version prior to Rancher v1.6.0**: If you make any changes to the default Rancher library setting for your catalogs and then roll back, you will need to reset the branch used for the default Rancher library under **Admin** -> **Settings** -> **Catalog**. The current default branch is `v1.6-release`, but the old default branch is `master`. 

* **Rollback Versions**: We support rolling back to Rancher v1.6.5 from Rancher v1.6.7.
  * **Steps to Rollback**:
    1. In the upgraded version the **Admin** -> **Advanced Settings** -> API values, update the `upgrade.manager` value to `all`. 
    2. "Upgrade" Rancher Server but pointing to the older version of Rancher (v1.6.5). This should include backing up your database and launching Rancher to point to your current database.  
    3. Once Rancher starts up again, all infrastructure stacks will automatically rollback to the applicable version in v1.6.5. 
    4. There will be a couple of services in certain infrastructure stacks that need to be deleted. 
       a. Delete the `cni-driver` service in the IPsec stack.
       b. Delete the `cloud-controller-manager` service in the Kubernetes stack.
    5. After your setup is back to its original state, update the `upgrade.manager` value back to the original value that you had (either `mandatory` or `none`). 

## Infrastructure Service Updates
When upgrading infrastructure services, please make sure to [upgrade in the recommended order](http://www.cnrancher.com/docs/rancher/v1.6/en/upgrading/#infrastructure-services).

* **Network Services - v0.2.5**
  - _New image: `rancher/metadata:v0.9.3`_
  -  Fixed to use a fixed listen address of 169.254.169.250, instead of all available IP addresses. [#9548]
* **IPsec - 0.1.3**
  - _New image: `rancher/net:v0.11.7`_
  - Fixed an issue with the CNI driver where sometimes networking setup for a container would fail [#9418]
  - Fixed the container fetch logic to address service upgrade issues [#9559]
* **Kubernetes 1.7.2 - v1.7.2-rancher7**
  - _New images: `rancher/k8s:v1.7.2-rancher7`, `rancher/etcd:v2.3.7-13`_
  - When upgrading to this version of k8s, you will need to delete the `cloud-controller-manager` service after upgrade. Due to a k8s bug, we've had to disable this service. [#9599]
  - Fixed an issue where heapster was failing due to `NodeLegacyHostIp`, which was removed in k8s 1.7 [#9552]
  >Note: If upgrading from a k8s version prior to k8s v1.6, then you will need to re-generate any remote kubeconfig due to RBAC support.
* **Scheduler - 0.6.3** 
  - Fixed to use a fixed listen address of 169.254.169.250, instead of all available IP addresses.


## Known Major Issues
 
## Major Bug Fixes since v1.6.6
- See the list of infrastructure services to see major bug fixes

## [Rancher CLI](https://docs.rancher.com/rancher/v1.6/en/cli/) Downloads

https://github.com/rancher/cli/releases/tag/v0.6.3

## [Rancher-Compose](https://docs.rancher.com/rancher/v1.6/en/cattle/rancher-compose/) Downloads

https://github.com/rancher/rancher-compose/releases/tag/v0.12.5
