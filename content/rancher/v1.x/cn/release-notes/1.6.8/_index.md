---
title: Release v1.6.8
weight: 3001
---

## Versions
- rancher/server:v1.6.8
- rancher/agent:v1.2.6
- rancher/lb-service-haproxy:v0.7.9
- [rancher-v0.6.4](https://github.com/rancher/cli/releases/tag/v0.6.4)
- [rancher-compose-v0.12.5](https://github.com/rancher/rancher-compose/releases/tag/v0.12.5)

### Supported Docker Versions

* Docker 1.12.3-1.12.6
* Docker 1.13.1
* Docker 17.03.0-ce/ee 
* Docker 17.06.0-ce/ee 

> Note: Kubernetes 1.7 only supports up to Docker 1.12.6 

## Rancher Server Tags

Rancher Server has 2 different tags. For each major release tag, we will provide documentation for the specific version.
- `rancher/server:latest` tag will be our latest development builds. These builds will have been validated through our CI automation framework. These releases are not meant for deployment in production.
- `rancher/server:stable` tag will be our latest stable release builds. This tag is the version that we recommend for production.  

Please do not the releases with a `rc{n}` suffix. These `rc` builds are meant for the Rancher team to test out builds.

### Beta - v1.6.8 - `rancher/server:latest`
### Stable - v1.6.7 - `rancher/server:stable` 

## Important - Upgrade
* **Users on a version prior to Rancher v1.5.0:** We will automatically upgrade the `network-services` infrastructure stack as without this upgrade, your release will not work. 
* **Users on a version prior to Rancher v1.6.0**: If you make any changes to the default Rancher library setting for your catalogs and then roll back, you will need to reset the branch used for the default Rancher library under **Admin** -> **Settings** -> **Catalog**. The current default branch is `v1.6-release`, but the old default branch is `master`. 

* **Rollback Versions**: We support rolling back to Rancher v1.6.7 from Rancher v1.6.8.
  * **Steps to Rollback**:
    1. In the upgraded version the **Admin** -> **Advanced Settings** -> API values, update the `upgrade.manager` value to `all`. 
    2. "Upgrade" Rancher Server but pointing to the older version of Rancher (v1.6.7). This should include backing up your database and launching Rancher to point to your current database.  
    3. Once Rancher starts up again, all infrastructure stacks will automatically rollback to the applicable version in v1.6.7. 
    4. After your setup is back to its original state, update the `upgrade.manager` value back to the original value that you had (either `mandatory` or `none`). 

## Important - Please read if you currently have authentication enabled using Active Directory with TLS enabled prior to upgrading to v1.6.8.

Starting with v1.6.8, Rancher has updated the Active Directory auth plugin and moved it into the new authentication framework.  We have also further secured the AD+TLS option by ensuring that the hostname/IP of the AD server matches with the hostname/IP of the TLS certificate.  Please see [[#9459](https://github.com/rancher/rancher/issues/9459)] for details.

Due to this new check, you should be aware that if the hostname/IP does not match your TLS certificate, you will be locked out of your Rancher Server if you do not correct this prior to upgrading.  To ensure you have no issues with the upgrade, please execute the following to verify your configuration is correct.

- Verify the hostname/IP you used for your AD configuration.  To do this, log into Rancher using a web browser as an admin and click **Admin** -> **Access Control**.  Note the `server` field to determine your configured hostname/IP for your AD server.
- To verify your the configure hostname/IP for your TLS cert, you can execute the following command to determine the CN attribute:
```openssl s_client -showcerts -connect domain.example.com:443```
You should see something like:
```subject=/OU=Domain Control Validated/CN=domain.example.com```
Verify that the CN attribute matches with your configured `server` field from the above step.

If the fields match, you are good to go.  Nothing else is required.

If the fields **_do not match_**, please execute the following steps to correct it.

- Open a web browser and go to Rancher's `settings` URL.  This can be done by logging into Rancher as an admin and click **API**->**Keys**.  You should see an `Endpoint (v2-beta)` field.  Take the value of that field and append `/settings`.  The final URL should look something like `my.rancher.url:8080/v2-beta/settings`.  Launch this URL in your browser and you should see Rancher's API browser.  
- Search for `api.auth.ldap.server` and click that setting to edit it.  On the top right, you should be able to click an `edit` button.  Change the value of that to match the hostname/IP of the value found in your cert as identified by the CN attribute and click **Show Request**->**Send Request** to persist the value into Rancher's DB.  The response should show your new value.

Once this is completed and the hostname/IP matches your certs' CN attribute, you should have no issues with AD login after upgrading to 1.6.8.

## Enhancements
* **Support for RHEL 7.4 [#9674]** - Support for RHEL 7.4 requires you to upgrade your `network-services` to version `0.2.5`.
* **Ability to update LDAP without disabling access control [#8115]** - When the AD server has been updated, you can update your access control without disabling access control in Rancher.
* **Rancher CLI Enhancements**
  - Ability to use the environment name as a variable inside compose files. [#8791]
  - Support for `stop_grace_period` [#7715]
* **Support for auto-labeling of environment UUID for containers [#8442]** - Each container will now be labeled with its corresponding environment UUID.

## Infrastructure Service Updates
When upgrading infrastructure services, please make sure to [upgrade in the recommended order](http://www.rancher.cn/docs/rancher/v1.6/en/upgrading/#infrastructure-services).

* **Network Services - v0.2.6**
  - _New image: `rancher/network-manager:v0.7.8`, `rancher/dns:v0.15.3`, `rancher/metadata:v0.9.4`_
  - Fixed to honor upstream TTL [#8805]
  - Fixed nil pointer cache
  - Fixed to check locally before searching in global cache [#8504]
  - Fixed for go panic during json encoding [#9637]

* **IPsec - 0.1.4**
  - _New image: `rancher/net:v0.11.9`_
  - Changed to use Rancher Metadata IP directly to avoid name resolution [#9521]
  - Fixed to initiate IPSec tunnels from only one end [#8204]
  - Allowed ability to configure replay window size [#9377]
  - Allowed ability to configure rekey intervals [#9705]

* **Healthcheck - v0.3.3**
  - _New image: `rancher/healthcheck:v0.3.3`_
  - Added health check for sidekick service containers using networkFrom primary [#6305]
  - Changed to use Rancher Metadata IP directly to avoid name resolution [#9521]

* **VXLAN - 0.2.1**
  - _New image: `rancher/net:v0.11.9`_
  - Changed to use Rancher Metadata IP directly to avoid name resolution [#9521]

* **Kubernetes 1.7.4 - v1.7.4-rancher2**
  - _New images: `rancher/k8s:v1.7.4-rancher2`, `rancher/kubernetes-agent:v0.6.5`, `rancher/etcd:v2.3.7-13`, `rancher/kubectld:v0.8.3`, `rancher/etc-host-updater:v0.0.3`, `rancher/kubernetes-auth:v0.0.8`, `rancher/lb-service-rancher:v0.7.10`_
  - Changed to use Rancher Metadata IP directly to avoid name resolution
  - Added ability to support `kubernetes.io/ingress.class` annotation [#6344]
  - Fixed an issue where pods using host network had broken networking [#9678]
  >Note: If upgrading from a k8s version prior to k8s v1.6, then you will need to re-generate any remote kubeconfig due to RBAC support.

## Known Major Issues
- Kubernetes Users with a auth protected private registry: Add-on starter is unable to pull the `pause-amd64:3.0` image, which causes add-ons to not start. Workaround: If the `pause-amd64:3.0` image is pre-pulled onto the hosts, then add-ons will be able to start as expected. [#9790]
- Issue with AD authentication where certain AD setups were not able to accurately select the unique LDAP user from AD and show up as "User not Found" [#9813]
- Issue with AD authentication where usernames/passwords with certain characters were not able to be log in [#9815, #9830]
- Issue with AD authentication where users with escaped commas in their DN are not able to log in or use API keys [#9827, #9828]

## Major Bug Fixes since v1.6.7
- Fixed an issue where service account password was being sent back from API during access control authentication [#8625]
- Fixed an issue where haproxy logs would be redirected to file only.  Logs will also be sent to stdout/stderr [#9616]
- Fixed an issue where private catalogs were no longer alphabetically sorted [#9446]
- Fixed an issue where the load balancer wouldn't work if there were more than 2 backends in the custom haproxy config [#8226]
- Fixed an issue where `rancher up` didn't respect `--wait` and `--wait-state` [#8492]
- Fixed an issue where the default timeout for ldap connections was too small [#8944]
- Added the ability to change the log level of rancher-auth-service when Rancher starts for debugging [#9390]
- Fixed an issue where user stacks starting before infra stacks end up in a deadlock [#9680]
- Fixed an issue where allocation errors were not displaying the resource constraints in some scenarios [#9554]
- Fixed an issue where catalog templates would take a long time to populate from the cached catalog on an air-gapped setup [#9669]

## [Rancher CLI](https://docs.rancher.com/rancher/v1.6/en/cli/) Downloads

https://github.com/rancher/cli/releases/tag/v0.6.4

## [Rancher-Compose](https://docs.rancher.com/rancher/v1.6/en/cattle/rancher-compose/) Downloads

https://github.com/rancher/rancher-compose/releases/tag/v0.12.5
