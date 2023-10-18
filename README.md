<!-- TOC -->

- [Manage SUSE Enterprise Linux Systems with Rancher System-Upgrade-Controller](#manage-suse-enterprise-linux-systems-with-rancher-system-upgrade-controller)
    - [Setup SLE Micro 5.5+ install ISO with appended combustion configuration](#setup-sle-micro-55-install-iso-with-appended-combustion-configuration)
        - [Setup your combustion configuration](#setup-your-combustion-configuration)
            - [elemental register and agent components](#elemental-register-and-agent-components)
        - [Instructions to creating your appended combustion SLE Micro installation image](#instructions-to-creating-your-appended-combustion-sle-micro-installation-image)
    - [Setup SLE Micro 5.x install ISO with embedded ignition/combustion partition](#setup-sle-micro-5x-install-iso-with-embedded-ignitioncombustion-partition)
        - [Setup your ignition/combustion configuration](#setup-your-ignitioncombustion-configuration)
            - [First, the ignition configuration](#first-the-ignition-configuration)
                - [elemental register and agent components](#elemental-register-and-agent-components)
            - [Next, the combustion configuration](#next-the-combustion-configuration)
                - [elemental register and agent components](#elemental-register-and-agent-components)
        - [Instructions to creating your embedded ignition/combustion SLE Micro installation image](#instructions-to-creating-your-embedded-ignitioncombustion-sle-micro-installation-image)
    - [Installing SLE Micro using SelfInstall ISO](#installing-sle-micro-using-selfinstall-iso)
        - [Register the SLE Micro system to Rancher with elemental-register](#register-the-sle-micro-system-to-rancher-with-elemental-register)
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

The goal of the information contained in this repository is to allow you to manage the operating system, and in this case SUSE Linux Enterprise (SLE Micro, SUSE Linux Enterprise Server, and SUSE ALP) using the Rancher system-upgrade-controller as the mechanism. There are many reasons why you might want to use it in this way. Some of the top reasons are the barrier to entry is very low and does not require a lot of setup. You don't have to learn another tool. It fits perfectly in the realm of cloud native declarative workload management. Creating a descriptive plan that eventually achieves the result as it has been defined. 

There is one dependency in this setup when your working with a production environment and that is "Where do you get the updates?". You could source them from the SUSE Customer Center and register all of your nodes there, but then everything is going over the internet for its updates. In a production setup you will want to keep things isolated on premise to ensure proper provinence of your patches. Using a tool from SUSE called the Repository Mirroring Tool (RMT) will allow you to achieve this level of security with your updates and keep everything on premise. There is a guide published with each new service pack of SUSE Linux Enterprise Server (SLES) that walks you through how to set this up and mirror your software repositories. You can access this guide here [Repository Mirroring Tool Guide] (https://documentation.suse.com/sles/15-SP4/html/SLES-all/book-rmt.html) Another method for anyone looking for a cloud native approach is installing RMT on Kubernetes which you can reference in the same guide under "Deploying RMT on top of the Kubernetes cluster". Once you have RMT setup, then walk through the steps to mirror the SUSE Linux Enterprise distributions you will be using.

To continue you can follow the next steps or refer to the table of contents.

In the next steps you have a choice of what method of deployment you might want to use with SLE Micro, and there are pros and cons to each, but one will be better for your given situattion. The two options are to either append the combustion configuration or to embed the ingnition/combustion as a partition to the SelfInstall ISO image. The first option can only be used with SLE Micro 5.5+ because of the requirement for a new version of combustion that comes with it. The later can work with any of the versions of SLE Micro. The append option will give you greater flexibility and will be much easier to work with. 

## Setup SLE Micro 5.5+ install ISO with appended combustion configuration
This option to append the combustion configuration is limited to only using combustion, but you could call ignition by pulling its configuration from the network through a kernel boot line option or the added partition that you learn about in the next section. This would defeat the purpose though and I can't think of a reason why you would want to do this. Combustion is a powerful tool that will allow you to accomplish the same configuration options that are available to ignition and more. The added benefit of combustion is the fact that the configuration is a shell script that you can manipulate any way that you want to in order to accomplish your desired outcome. There are many documents out there that talk about combustion and I will list those below. Many of these documents will get you to the right spot, but don't give you a clean step by step instruction. I will walk you through that step by step below.

* [OpenSUSE Portal:MicroOS/Combustion](https://en.opensuse.org/Portal:MicroOS/Combustion)

* [SLE Micro Documentation](https://documentation.suse.com/sle-micro/5.4/html/SLE-Micro-all/cha-images-ignition.html)

Download your SLE Micro 5.5+ **SelfInstall** ISO [here](https://www.suse.com/download/sle-micro/) and make sure you choose the file with the name that looks like this with **SelfInstall** in the name:
```SLE-Micro.x86_64-5.5.0-Default-SelfInstall-GM.install.iso```

### Setup your combustion configuration
In this setup we will focus completely on using combustion for the configuration of your system. 

There is a nifty tool created by the openSUSE team to help you create your first combustion configuration. Just use [Fuel Ignition.](https://opensuse.github.io/fuel-ignition/)

Combustion uses a single file called **script** which is a bash shell script. This can be used to change many advanced things like default partitioning, install packages, required drivers, manipulating services, and even some of the same things as ignition such as adding files, passwords, creating users, and many more. Combustion can be very powerful and you should explore the many options. 

Attached to this repository you will find a sample script for your use and modification. It has some ideas in it for getting started and many other annotated comments. Some things included are regisering to an RMT server, installing and enabling docker service for use with RKE1 if required, and disabling the transactional-update.timer for use with this solution so the timer does not run daily pulling updates automatically. 

#### elemental register and agent components
In the combustion folder there are two additional files that will need to be copied into the folder. The file config.yaml and an rpm called yq. The config.yaml needs to be retrieved from your OS Management Registration Endpoint. Once you have your Registration Endpoint created you will see something similar to this "OS Management -> Registration Endpoint" ![OS Management > Registration Endpoint](/doc/OSManage-RegEndpoint.png?raw=true)
When you have a proper registration endpoint setup you will see the Registration URL. Copy that URL and open it in a new browser tab/window and you will see the output of your config.yaml file that you require for elemental-register. It will look like this > ![Registration URL](/doc/Registration-URL.png?raw=true)
Copy or download the file and drop it into the combustion folder with the script file.
The next thing required which is temporary until we have something more solid developed (The next release of elemental-register will fix this) is the use of the yq binary. If you don't have the PackageHub repository attached then you can grab this package from the build service and download directly from https://download.opensuse.org/repositories/openSUSE:/Backports:/SLE-15-SP4/standard/x86_64/yq-4.18.1-bp154.1.17.x86_64.rpm Once you have the rpm package downloaded then move it to the combustion folder where the script file is located. There is a command in the script which installs the rpm from that folder. Read through the annotated script file since it also has other instructions or modifications that you could make. 
### Instructions to creating your appended combustion SLE Micro installation image
This option has significantly less steps to setup.
To create your SLE Micro appended installation ISO I use the xorriso command included with the Linux OS I use (openSUSE TW).

Step 1, prep/create your combustion folder and **Script**

Step 2, execute xorriso to append the combustion folder to the root of the SelfInstall ISO:

```xorriso -indev ./SLE-Micro.x86_64-5.5.0-Default-SelfInstall-Build10.1.install.iso -outdev ./SLE-Micro-Selfinstall.iso -map $(pwd)/combustion /combustion -boot_image any replay -changes_pending yes```


## Setup SLE Micro 5.x install ISO with embedded ignition/combustion partition
SLE Micro is a powerful Linux distribution targetted at specialized workloads for containers and virtual machines, which comes deployed as an immutable infrastructure with a security profile using SELinux that is locked down from the very start. Since we are working with an immutable deployment you will want to change the personality of that deployment during the installation process. There are two methods built into SLE Micro that allow you to do this, and those are Ignition and Combustion. You can use one or the other, or combine them to achieve your results. Either way, it is a powerful combination together to achieve anything you require to give your system the personality it needs. There are many documents out there that talk about these two tools and I will list those below. Many of these documents will get you to the right spot, but don't give you a clean step by step instruction. I will walk you through 
that step by step below. 

* [OpenSUSE Portal:MicroOS/Ignition](https://en.opensuse.org/Portal:MicroOS/Ignition)

* [OpenSUSE Portal:MicroOS/Combustion](https://en.opensuse.org/Portal:MicroOS/Combustion)

* [SLE Micro Documentation](https://documentation.suse.com/sle-micro/5.4/html/SLE-Micro-all/cha-images-ignition.html)

Download your SLE Micro **SelfInstall** ISO [here](https://www.suse.com/download/sle-micro/) and make sure you choose the file with the name that looks like this with **SelfInstall** in the name:
```SLE-Micro.x86_64-5.4.0-Default-SelfInstall-GM.install.iso```
### Setup your ignition/combustion configuration
Before tackling the image creation we need to first have a basic understanding of its configuration, and before you do the creation you will first want to make a decsion about whether you want to use ignition or combustion, or use them both together. I'm going to use them both together in this setup just to give you an idea of how that is done. They both have different configurations. 

There is a nifty tool created by the openSUSE team to help you create your first ignition and combustion configuration. Just use [Fuel Ignition.](https://opensuse.github.io/fuel-ignition/)
#### First, the ignition configuration
This configuration uses a file called **config.ign** which is a json file format. You can add users, create passwords, create files, add ssh keys, network configuration, create and enable services, and much more. For a full explanation read the [Ignition Docs.](https://coreos.github.io/ignition/)

Attached to this repository you will find a sample config.ign for your use and modification. It includes a few users with a section for authorized keys, a hostname, and the password hash is for **linux** as the password on both users. The network is setup for dhcp. Go ahead and change it to fit your setup.
##### elemental register and agent components
There are two sections near the bottom of the config.ign file. One component is the elemental-system-agent.service enablement so that it automatically gets turned on and connects to the Rancher server. Another one is the config.yaml for elemental-register, which as of this writing it might not be required because combustion takes care of it. 
#### Next, the combustion configuration
For combustion it uses a single file called **script** which is a bash shell script. This can be used to change many advanced things like default partitioning, install packages, required drivers, and even some of the same things as ignition such as adding files, passwords, creating users, and many more. Combustion can be very powerful and used various use cases. Explore the many options. 

Attached to this repository you will find a sample script for your use and modification. It has some ideas in it for getting started and many other annotated comments. Some things included are regisering to an RMT server, installing and enabling docker service for use with RKE1 if required, and disabling the transactional-update.timer for use with this solution so the timer does not run daily pulling updates automatically. 
##### elemental register and agent components
Just like in the ignition configuration there are a few areas that need to be setup so that the elemental-register can work properly. In the combustion folder there are two additional files that will need to be copied into the folder. The file config.yaml and an rpm called yq. The config.yaml needs to be retrieved from your OS Management Registration Endpoint. Once you have your Registration Endpoint created you will see something similar to this "OS Management -> Registration Endpoint" ![OS Management > Registration Endpoint](/doc/OSManage-RegEndpoint.png?raw=true)
When you have a proper registration endpoint setup you will see the Registration URL. Copy that URL and open it in a new browser tab/window and you will see the output of your config.yaml file that you require for elemental-register. It will look like this > ![Registration URL](/doc/Registration-URL.png?raw=true)
Copy or download the file and drop it into the combustion folder with the script file.
The next thing required which is temporary until we have something more solid developed is the use of the yq binary. If you don't have the PackageHub repository attached then you can grab this package from the build service and download directly from https://download.opensuse.org/repositories/openSUSE:/Backports:/SLE-15-SP4/standard/x86_64/yq-4.18.1-bp154.1.17.x86_64.rpm Once you have the rpm package downloaded then move it to the combustion folder where the script file is located. There is a command in the script which installs the rpm from that folder. Read through the annotated script file since it also has other instructions or modifications that you could make. 
### Instructions to creating your embedded ignition/combustion SLE Micro installation image
To create your ignition/combustion image easily I use a Linux OS (openSUSE TW) with a few tools like truncate, mkfs.ext4, e2label, and xorriso.

Step 1, create an image file of 20Mib size:

```# truncate -s +20M ignition.img```

Step 2, write an ext4 filesystem to the image:

```# sudo mkfs.ext4 ignition.img```

Step 3, label your image with ignition:

```# sudo e2label ignition.img "ignition"```

Step 4, mount the image for writing:

```# sudo mount -o loop ignition.img /mnt```

Step 5, write your ignition and combustion directories to the image:

```# sudo mkdir ignition combustion /mnt```

Step 6, copy ignition configuration to the ignition directory:

```# cp config.ign /mnt/ignition```

Step 7, copy the script to the combustion directory:

```# cp script /mnt/combustion```

Step 8, unmount the image:

```# sudo umount /mnt```

Step 9, execute xorriso to attach a 3rd partition to the end of the SelfInstall ISO:

```# xorriso -indev SLE-Micro.x86_64-5.4.0-Default-SelfInstall-GM.install.iso -outdev SLE-Micro-Selfinstall.iso -boot_image any replay -append_partition 3 Linux ignition-micro.img -changes_pending yes```

## Installing SLE Micro using SelfInstall ISO
Depending on whether you used append or embed will dictate how you will be able to use the ISO. If your Selfinstall ISO has been extended with an additional partition on the end you can only use it as a boot media from USB. So when you are attaching it to a virtual platform like VMware, KVM, or VirtualBox then you need to attach it as a bootable USB media. The CDROM media will not mount or recognize the 3rd partition. When booting with the USB media type it will read the ignition label and mount it. If your Selfinstall ISO was appended with the combustion folder on the root of the ISO then you can boot the ISO from standard CDROM Media with any platform, or write it out to a USB thumb drive and boot it. There is more flexibility in the append method. 

When you have the media mounted properly You will boot to the grub install options and select "Install SLE Micro"

### Register the SLE Micro system to Rancher with elemental-register
This should now happen automatically if you followed the steps in the setup of the ignition or combustion and have included it in your SelfInstall ISO.
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