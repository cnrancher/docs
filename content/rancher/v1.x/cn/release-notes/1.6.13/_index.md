---
title: Release v1.6.13
weight: 3003
---

## Versions
- rancher/server:v1.6.10
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

### Beta - v1.6.10 - `rancher/server:latest`
### Stable - v1.6.10 - `rancher/server:stable` (as of 9/20/17)

## Important - Upgrade
* **Users on a version prior to Rancher v1.5.0:** We will automatically upgrade the `network-services` infrastructure stack as without this upgrade, your release will not work. 
* **Users on a version prior to Rancher v1.6.0**: If you make any changes to the default Rancher library setting for your catalogs and then roll back, you will need to reset the branch used for the default Rancher library under **Admin** -> **Settings** -> **Catalog**. The current default branch is `v1.6-release`, but the old default branch is `master`. 

* **Rollback Versions**: We support rolling back to Rancher v1.6.7 from Rancher v1.6.10.
  * **Steps to Rollback**:
    1. In the upgraded version the **Admin** -> **Advanced Settings** -> API values, update the `upgrade.manager` value to `all`. 
    2. "Upgrade" Rancher Server but pointing to the older version of Rancher (v1.6.7). This should include backing up your database and launching Rancher to point to your current database.  
    3. Once Rancher starts up again, all infrastructure stacks will automatically rollback to the applicable version in v1.6.7. 
    4. After your setup is back to its original state, update the `upgrade.manager` value back to the original value that you had (either `mandatory` or `none`). 

> **Note on Rollback:** If you are rolling back and have authentication enabled using Active Directory, any new users/groups added to site access on the Access Control page after the upgrade will not be retained upon rolling back. Any users added before the upgrade will continue to remain. [#9850]

## Important - Please read if you currently have authentication enabled using Active Directory with TLS enabled prior to upgrading to v1.6.10.

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

## Known Major Issues
- Kubernetes Users with a auth protected private registry: Add-on starter is unable to pull the `pause-amd64:3.0` image, which causes add-ons to not start. Workaround: If the `pause-amd64:3.0` image is pre-pulled onto the hosts, then add-ons will be able to start as expected. [#9790]

## Major Bug Fixes since v1.6.9
- Fixed an issue with AD authentication where users cannot configure AD auth if login domain is used as prefix for test username [#9889]

## [Rancher CLI](https://docs.rancher.com/rancher/v1.6/en/cli/) Downloads

https://github.com/rancher/cli/releases/tag/v0.6.4

## [Rancher-Compose](https://docs.rancher.com/rancher/v1.6/en/cattle/rancher-compose/) Downloads

https://github.com/rancher/rancher-compose/releases/tag/v0.12.5