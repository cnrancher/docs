---
title: 5 - helm HA离线升级
weight: 5
---

The following instructions will guide you through upgrading a high-availability Rancher Server installed in an air gap environment.

## Prerequisites

- **Populate Images**

    Follow the guide to [Prepare the Private Registry]({{< baseurl >}}/rancher/v2.x/en/installation/air-gap-installation/prepare-private-reg/) with the images for the upgrade Rancher release.

- **Backup your Rancher Cluster**

    [Take a one-time snapshot]({{< baseurl >}}/rancher/v2.x/en/backups/backups/ha-backups/#option-b-one-time-snapshots)
    of your Rancher Server cluster. You'll use the snapshot as a restoration point if something goes wrong during upgrade.

- **kubectl**

    Follow the kubectl [configuration instructions]({{< baseurl >}}/rancher/v2.x/en/faq/kubectl) and confirm that you can connect to the Kubernetes cluster running Rancher server.

- **helm**

    [Install or update](https://docs.helm.sh/using_helm/#installing-helm) Helm to the latest version.

## Upgrade Rancher

1. Update your local helm repo cache.

    ```
    helm repo update
    ```

2. Get the [repository name that you installed Rancher]({{< baseurl >}}/rancher/v2.x/en/installation/server-tags/#helm-chart-repositories) with.

    ```
    helm repo list

    NAME          	      URL
    stable        	      https://kubernetes-charts.storage.googleapis.com
    rancher-<CHART_REPO>	https://releases.rancher.com/server-charts/<CHART_REPO>
    ```

    > **Note:** If you want to switch to a different Helm chart repository, please follow the [steps on how to switch repositories]({{< baseurl >}}/rancher/v2.x/en/installation/server-tags/#switching-to-a-different-helm-chart-repository). If you switch repositories, make sure to list the repositories again before continuing onto Step 3 to ensure you have the correct one added.


3. Fetch the latest chart to install Rancher from the Helm chart repository.

    This command will pull down the latest chart and save it in the current directory as a `.tgz` file. Replace `<CHART_REPO>` with the name of the repository name that was listed (i.e. `latest` or `stable`).


    ```plain
    helm fetch rancher-<CHART_REPO>/rancher
    ```

3. Render the upgrade template.

    Use the same `--set` values that you used for the install. Remember to set the `--is-upgrade` flag for `helm`. This will create a `rancher` directory with the Kubernetes manifest files.

    ```plain
    helm template ./rancher-<version>.tgz --output-dir . --is-upgrade \
    --name rancher --namespace cattle-system \
    --set hostname=<RANCHER.YOURDOMAIN.COM> \
    --set rancherImage=<REGISTRY.YOURDOMAIN.COM:PORT>/rancher/rancher
    ```

4. Copy and apply the rendered manifests.

    Copy the files to a server with access to the Rancher server cluster and apply the rendered templates.

    ```plain
    kubectl -n cattle-system apply -R -f ./rancher
    ```

## Rolling Back

Should something go wrong, follow the [HA Rollback]({{< baseurl >}}/rancher/v2.x/en/upgrades/rollbacks/ha-server-rollbacks/) instructions to restore the snapshot you took before you preformed the upgrade.
