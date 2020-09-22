# vSphere Container Storage Interface (CSI)

[vSphere Container Storage Interface](https://github.com/kubernetes-sigs/vsphere-csi-driver) handles storage specific functionality for Kubernetes running on VMware vSphere infrastructure.

## Introduction

This chart deploys all components required to run the vSphere CSI as described on it's [GitHub page](https://vsphere-csi-driver.sigs.k8s.io/). This Helm chart has been created to work with CSI driver version 2.0, released with vSphere 7.0. This means that it supports both block and file Persistent Volumes. Block PVs can be dynamically created on VMFS, NFS, vSAN and vVol datastores. File PVs can be dynamically created on vSAN datastores with vSAN File Services enabled.

## Prerequisites

- Has been tested on Kubernetes v1.18.3
- This Helm chart assumes that your Kubernetes cluster has not been configured to use the external vSphere CPI cloud provider. The vSphere CSI driver has a dependency on the vSphere CPI cloud provider and this Helm chart will automatically deploy is as part of the CSI deployment. For further information on the vSphere CPI driver, please refer to the following documentation: [Kubernetes documentation](https://kubernetes.io/docs/tasks/administer-cluster/running-cloud-controller/#running-cloud-controller-manager).

## Adding this helm repository

To add the helm repository for the vSphere CSI driver, run the following commands:

```bash
helm repo add cormachogan https://cormachogan.github.io/vsphere-csi-helmchart
helm repo update
```

## Testing the Helm Chart

To test the helm chart before installing it , the following commands can be run:

```bash
helm template --debug vsphere-csi cormachogan/vsphere-csi
helm install --dry-run  --debug vsphere-csi cormachogan/vsphere-csi
```

## Installing the Helm Chart for Block based Persistent Volumes

To install this chart for block based PVs, you will need to provide additional vCenter information/credentials. Run the following command (but replace the placeholder values with the ones for your environment):

```bash
helm install vsphere-csi cormachogan/vsphere-csi \
--namespace kube-system \
--set config.enabled=true \
--set config.vcenter=<vCenter IP> \
--set config.username=<vCenter Username> \
--set config.password=<vCenter Password> \
--set config.datacenter=<vCenter Datacenter> \
--set config.clusterId='changeme' \
--set vsphere-cpi.config.enabled=true \
--set vsphere-cpi.config.vcenter=<vCenter IP> \
--set vsphere-cpi.config.username=<vCenter Username> \
--set vsphere-cpi.config.password=<vCenter Password> \
--set vsphere-cpi.config.datacenter=<vCenter Datacenter> \
--set vsphere-cpi.namespace=<target-namespace> \
--set netconfig.enabled=false
```

A full example can be seen here. If you need to make changes to any of the configuration options (other than the secret), you can use the `upgrade --install` option shown here. If you need to change the secret, you will have to delete and reinstall the chart.

> **Caution**: The clusterId is a unique identifier for the Kubernetes cluster, chosen at the time the helm chart is installed. The same clusterId should not be used for different Kubernetes clusters managed by the same vCenter Server.

```bash
helm upgrade --install vsphere-csi cormachogan/vsphere-csi
--namespace kube-system \
--set config.enabled=true \
--set config.vcenter=vcsa-01.rainpole.com \
--set config.password=VMware123 \
--set config.datacenter=Datacenter \
--set config.clusterId=MyCluster1 \
--set vsphere-cpi.config.enabled=true \
--set vsphere-cpi.config.vcenter=vcsa-01.rainpole.com \
--set vsphere-cpi.config.password=VMware123 \
--set vsphere-cpi.config.datacenter=Datacenter \
--set vsphere-cpi.namespace=kube-system \
--set netconfig.enabled=false
```

> **Tip**: List all releases using `helm list --all`

## Installing the Helm Chart for Block and File based Persistent Volumes

To install this helm chart for both block and file PVs, the chart includes a `netconfig` parameter set to support CSI file shares. This allows vSAN File Shares to be used as read-write-many Persistent Volumes. To enable a certain IP address range to access the file shares, select the datastore where file shares can be created, set specific file share permissions and control the root squash parameter, run the following command:

```bash
helm upgrade --install vsphere-csi cormachogan/vsphere-csi
--namespace kube-system \
--set config.enabled=true \
--set config.vcenter=vcsa-01.rainpole.com \
--set config.password=VMware123 \
--set config.datacenter=Datacenter \
--set config.clusterId=MyCluster1\
--set vsphere-cpi.config.enabled=true \
--set vsphere-cpi.config.vcenter=vcsa-01.rainpole.com \
--set vsphere-cpi.config.password=VMware123 \
--set vsphere-cpi.config.datacenter=Datacenter \
--set vsphere-cpi.namespace=kube-system \
--set netconfig.enabled=true \
--set netconfig.datastore=ds:///vmfs/volumes/vsan:52e2cfb57ce8d5d3-c12e042893ff2f76/ \
--set netconfig.ips='*' \
--set netconfig.permissions=READ_WRITE \
--set netconfig.rootsquash=true
```

The datastore URL entry above may be found in the vSphere client of the vCenter Server managing the Kubernetes Cluster. Select the Datastores view, then the vSAN datastore that supports vSAN File Services. In the Summary view, the URL will be displayed.

## Still on vSphere 6.7U3 (csi-resizer crashing)

If you are still on vSphere 6.7U3, you will need to use `--set csiResizer.enabled=false` to disable the csi-resizer container, which requires vSphere 7.0 or newer.

## Override image source (air gapped)

If your Kubernetes clusters are in a network that is firewalled from the Internet, and need to override the image sources to something internal, please see the values.yaml.

## Manually installing the vSphere CSI driver

If you want to provide your own `csi-vsphere.conf`, for example, to handle multple datacenters/vCenters or for using zones, you can learn how to manually deploy the CSI driver by reading the following [documentation](https://vsphere-csi-driver.sigs.k8s.io/driver-deployment/installation.html).

## Uninstalling the vSphere CSI Helm Chart

To uninstall/delete the `vsphere-csi` deployment via Helm:

```bash
helm delete vsphere-csi --namespace kube-system
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

> **Tip**: To permanently remove the release using Helm, run `helm delete --purge vsphere-csi --namespace kube-system`
