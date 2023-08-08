<!-- TOC -->

- [Manage SUSE Enterprise Linux Systems with Rancher System-Upgrade-Controller](#manage-suse-enterprise-linux-systems-with-rancher-system-upgrade-controller)
    - [Setup SLE Micro install ISO with embedded ignition/combustion partition](#setup-sle-micro-install-iso-with-embedded-ignitioncombustion-partition)
    - [Install downstream cluster using Rancher](#install-downstream-cluster-using-rancher)
        - [Install system-upgrade-controller to downstream cluster](#install-system-upgrade-controller-to-downstream-cluster)
    - [Setup system upgrade plan](#setup-system-upgrade-plan)
        - [Apply system upgrade plan to downstream cluster](#apply-system-upgrade-plan-to-downstream-cluster)
        - [Apply labels to control execution](#apply-labels-to-control-execution)
    - [Inventory Kubernetes nodes using labels](#inventory-kubernetes-nodes-using-labels)
    - [Staging downstream clusters for upgrades](#staging-downstream-clusters-for-upgrades)
        - [Install staging snapshot to downstream clusters](#install-staging-snapshot-to-downstream-clusters)
    - [Install live patches to downstream clusters no reboots](#install-live-patches-to-downstream-clusters-no-reboots)
    - [Snapshot rollback](#snapshot-rollback)
    - [Upgrade 3rd party drivers using system upgrade plan](#upgrade-3rd-party-drivers-using-system-upgrade-plan)
    - [Upgrade firmware using system upgrade plan](#upgrade-firmware-using-system-upgrade-plan)

<!-- /TOC -->

# Manage SUSE Enterprise Linux Systems with Rancher System-Upgrade-Controller

The goal of the information contained in this repository is to allow you to manage the operating system, and in this case SUSE Enterprise Linux (SLE Micro, SUSE Linux Enterprise Server, and SUSE ALP) using the Rancher system-upgrade-controller as the mechanism. There are many reasons why you might want to use it in this way. Some of the top reasons are the barrier to entry is very low and does not require a lot of setup. You don't have to learn another tool. It fits perfectly in the realm of cloud native declarative workload management. Creating a descriptive plan that eventually achieves the result as it has been defined. 

## Setup SLE Micro install ISO with embedded ignition/combustion partition


## Install downstream cluster using Rancher

### Install system-upgrade-controller to downstream cluster

## Setup system upgrade plan

### Apply system upgrade plan to downstream cluster

### Apply labels to control execution


## Inventory Kubernetes nodes using labels

## Staging downstream clusters for upgrades

### Install staging snapshot to downstream clusters

## Install live patches to downstream clusters (no reboots)

## Snapshot rollback

## Upgrade 3rd party drivers using system upgrade plan

## Upgrade firmware using system upgrade plan