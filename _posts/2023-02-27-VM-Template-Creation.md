---
layout:
title: "Virtual Machine Clone & Template Creation" 
date: 2023-02-24
categories: [Virtual Machine, Hypervisor]
tags: [machine-id, ssh host-keys, proxmox]
---

### Achieving definitive [Virtual Machine Rendition](#create-vm-from-iso-image) for [Clone](#backup-clone-creation) and [Template Creation](#vm-template-creation).  

<!-- Insert imagae portraying VM/clone rendition -->
![copyclone]({{ "/assets/img/postimages/indiclone.gif" | relative_url }}){:width="60%"}

# Prerequisites and Scope
Familiarity with Type 1 Hypervisors, in particular Proxmox will help give context to some of this articles content. Proxmox Virtual Environment and Kali Linux OS are enlisted for our subject exploration but with allowance for some variation, this content will translate well to other hypervisor platforms, ISO-images, and distributions. There are many other techniques for pre-configuring and automating virtual machine cloning and template creation that are well worth learning but they are out of scope for this article.

<!-- Insert imagae portraying VM/clone rendition -->
![copyclone]({{ "/assets/img/postimages/topintroimage.png" | relative_url }})


# Intro
This article aims to demonstrate and highlight important sanitation techniques when working with Virtual Machines. Namely, stopping the duplication of [Machine-IDs](#remove-machine-id) and [SSH Host-Keys](#remove-ssh-host_keys) during Virtual Machine creation from a template. We will also explore customisation of OS distribution, installing applications and utilities, and optimising system configuration for use case.

Evolving specialised setups and configurations can be organic or prescriptive as desired. You can strip down or bulk up whatever the use-case requirements but there are important baseline delineations to know and adhere to as a given.

# Baseline Delineation
A template should produce a VM that meets a known baseline setup, configuration, and resource allocation. This will help to produce an optimal, secure, and glitch free VM from your templates.

# Principle Objectives
These are the principal objectives highlighted but we action them downstream to reflect a logical workflow.  

- **Clear Machine-ID:** Leaving Machine-IDs designated in a template results in duplicate Machine-IDs for every consequent VM. Removing Machine-IDs prior to template creation is a must.

<!-- Insert imagae portraying mashines with differnt IDs -->
<!-- ![kitycolor]({{ "/assets/img/postimages/.png" | relative_url }}) -->

- **Clear all SSH Host-Keys:** A fresh VM when started for the first time will generate a new set of SSH Host Keys. SSH Host-Keys need to be cleared prior to template creation or risk multiple machines using identical SSH Host-Keys.

<!-- Insert imagae portrayingz:Multiple machines attempting to use the same ssh host key ??? -->
<!-- ![eyesdouble]({{ "/assets/img/postimages/.gif" | relative_url }}) -->
 

- **Distribution-Upgrade controls:** Maintain prescribed distribution version during system updates. For rolling versions perform Full-Update and Distribution-Upgrade.

<!-- Insert imagae portraying: Distribution Versioning ??? -->
<!-- ![sescriptivename]({{ "/assets/img/postimages/.png" | relative_url }})
_????_ -->

# Supplementary & optional objectives

Based on a known or expected virtual machine use case: Install and customize applications, utilities, security features, network and connectivity, desktop, and display setting.

- [UFW (Uncomplicated Fire Wall)](#install-firewall-utilities)
- [proxychains4](#configure-proxychains4)
- [IP forwarding](#configure-proxychains4)
- [Tor Tor Browser-Launcher](#install-tor-tor-browser-launcher)
- [alien](#install-alien)
- [icedtea](#install-icedtea)
- [Terminator](#install-terminator)
- [WireGuard VPN](#install-wireguard-vpn)
- [Display settings](#display-settings)

<!-- Insert imagae portraying: Applications and Utilities -->
<!-- ![apptoolkit]({{ "/assets/img/postimages/.png" | relative_url }}) -->
<!-- _????_ -->

# ISO-image
In this section we will download our ISO-image of choice and upload it to the PROXMOX image library.

<!-- Insert imagae portraying: ISO-image icon -->
<!-- ![isoimageicon]({{ "/assets/img/postimages/.png" | relative_url }}) -->
<!-- _????_ -->

## Download ISO-image
Download chosen ISO-image to your local machine. In this case I am downloading Kali Linux to my downloads folder using the link below.

<!-- Insert imagae portraying: Screen shot of Source download -->
![imagesource]({{ "/assets/img/postimages/isoimagedownload.png" | relative_url }}){:width="60%"}
<!-- _????_ -->

|   |   |
|---|---|
| OS | 64-bit Kali Linux 2022.4 |
| Installer | Complete offline installation with customization, 3.5G |
| ISO Image | https://cdimage.kali.org/kali-2022.4/kali-linux-2022.4-installer-amd64.iso |
| SHA256sum | aeb29db6cf1c049cd593351fd5c289c8e01de7e21771070853597dfc23aada28 |

## Upload ISO-image
Upload the local copy of  your ISO-image to the PROXMOX image library. Within PROXMOX VE navigate to: local node > click on ISO-mage > select Upload and follow popup prompts to complete.

<!-- Insert imagae portraying: Screen shot of PVE image upload -->
![uploadisoimage]({{ "/assets/img/postimages/uploadisocrop.png" | relative_url }}){:width="60%"}
_There are numerous methods used to download, upload and add images that are well worth learning and understanding. This example uses a simple and straightforward method._  


# Create VM from ISO-image
Create the VM that will be customized for cloning and template creation. Within PROXMOX VE click on the ‘Create VM’ button and follow the wizard, populating fields as detailed in the below example. Change input as required in accordance with your use case, resources availability, and conventions. 

<!-- Insert image portraying: Screen shot of ISO setup within PVE  -->
![vmcreation]({{ "/assets/img/postimages/vmcreateshow.png" | relative_url }}){:width="60%"}
<!-- _????_ -->

|   |   |
|---|---|
| General | Node: Proxmox |
|  | VM ID: 101 (Any number: VM IDs must be unique but can be changed) |
|  | Name: kaliOriginOne (Chose any name in line with your naming conventions) |
|  | Resource Pool: (Leave default if no resource pool setup) |
| OS | Storage: local   |
|  | ISO image: kali-linux-2022.4-installer-amd64.iso |
|  | Type: Linux |
|  | Version: latest |
| System | Leave as default |
| Disks | Enable Discard |
|  | All else leave default |
| CPU | Sockets: 2    (Ensure allocation is within your resource availability) |
|  | Cores: 2       (Ensure allocation is within your resource availability) |
| Memory (MIB) | 32128          (Ensure allocation is within your resource availability) |
| Network | Leave as default |

# VM Initial Setup and OS Install

## Setup newly created VM from Server-View in PVE
Select the VM: > Select: Hardware from Summary drop down: > click Remove.

|   |   |
|---|---|
| Hardware | Remove the CD/DVD Drive |

<!-- Insert imagae portraying: Screen shot process from within PVE -->
![removecd]({{ "/assets/img/postimages/remocecd.png" | relative_url }}){:width="60%"}
<!-- _????_ -->

Select the VM: > Select: Options from Summary drop down: > Select QEMU Guest Agent: > Click Edit: > Check box (Use QEMU Agent) and click OK.

|   |   |
|---|---|
| Options | QEMU Guest Agent: (Enable) |

<!-- Insert imagae portraying: Screen shot process from within PVE -->
![qemuenable]({{ "/assets/img/postimages/qemyenable.png" | relative_url }}){:width="60%"}
<!-- _????_ -->

# OS install - First Boot
From PVE 

- Start the VM and begin OS installation.
- Select the VM: > Go to: >_ Console: > START.

<!-- Insert imagae portraying: Screen shot process from within PVE -->
![startvm]({{ "/assets/img/postimages/startforstboot.png" | relative_url }}){:width="60%"}
<!-- _????_ -->
The VM will boot up for the first time and present an installation menu/wizard. 

<!-- Insert imagae portraying: Screen shot of icon vm startup -->
<!-- ![vmstartup]({{ "/assets/img/postimages/.png" | relative_url }}) -->
<!-- _????_ -->

- Select **Manual Install** and follow the prompts populating fields as detailed in the below example. Change input as required, according to your preference.

|   |   |
|---|---|
| Language | Select language of choice |
| Location | Select country of choice |
| Key map | Select key map of choice |
| Host name | Enter suitable host name use case |
| Domain Name | .local (or other suitable Domain Name for use case) |
| Full name for new user | FirstName LastName |
| Name for new user | UserName (Can be first name from Full Name) |
| Password for new user | Suitable password of choice |
| Configure clock | locations time zone |
| Language | Language of choice |
| Partition disks | (Default) Guided: use entire disk              |
|  | This will be the 34 GB allocated earlier |
|  | All files in one partition |
|  | Finish and write to disk |
| Software selection | Use defaults |
| Install the GRUB boot loader | Select device and enter |
| Finish the installation | Continue |

Next, choose to continue to the OS login where you are prompted for your User Name and Password. Use the credentials you configured above to log in for the first time.

## Validate OS in PVE
From a terminal session, validate the operating system is successfully running in PVE. 

<!-- Insert imagae portraying: Screen shot process within PVE and running VM -->
<!-- ![descriptivename]({{ "/assets/img/postimages/.png" | relative_url }}) -->
<!-- _????_ -->

```bash
# Confirm VM sucsessfully running in PVE.
cat /proc/cpuinfo
```

## Update & Upgrade OS Distribution
Complete a full Update & Distribution Upgrade.

<!-- Insert imagae portraying: Screen shot process within PVE and running VM -->
<!-- ![descriptivename]({{ "/assets/img/postimages/.png" | relative_url }}) -->
<!-- _????_ -->

```bash
# Full update and distribution upgrade.
sudo apt update && sudo apt dist-upgrade
```

## Status Check Qemu-Guest-Agent
From the terminal session, confirm Qemu-Guest-Agent is installed, alive, and exited.

<!-- Insert imagae portraying: Screen shot process within PVE and running VM (Agent rinning and excited) -->
<!-- ![descriptivename]({{ "/assets/img/postimages/.png" | relative_url }}) -->
<!-- _????_ -->

```bash
# Check status of qemu-guest-agent and start if required.
systemctl status qemu-guest-agent.service
systemctl start qemu-guest-agent.service

# Install and start if required
sudo apt install qemu-guest-agent 
```

# Install & Configure Applications and Utilities

# Install Firewall Utilities

Uncomplicated Fire Wall (UFW), is an efficient and user friendly utility to confirm firewall defaults are correct and enabled. To install, complete the following from terminal session.

## **Install UFW**

```bash
sudo apt-get install ufw
sudo ufw status verbose
sudo ufw enable

```

## **Install GUFW** GUI extension for UFW utility
```bash
sudo apt-get install gufw
sudo gufw
```

# Network & Connectivity

# Configure proxychains4

*(Optional depending on use case)*

Navigate to configuration file **proxychains4.conf.**  

```bash
sudo nano /etc/proxychains4.conf
```

Edit the configurations file **proxychains4.conf**

|   |   |
|---|---|
| Comment  | # strict_chain |
| UnComment | dynamic_chain |
| [ProxyList]: Add | socks5  127.0.0.1 9050 |

# Enable IP Forwarding
Optional and only advised for specific use cases. Understand the security considerations of enabling this feature.

Navigate to configuration file **sysctl.conf.**

```bash
sudo nano /etc/sysctl.conf
```

Edit the configuration file **sysctl.conf**

|   |   |
|---|---|
| UnComment lines | net.ipv4.ip_forward = 1 |
|  | net.ipv6.conf.all.forwarding = 1 |

Save and apply changes from the terminal session.

```bash
# Save changes then run command to apply change.
sysctl -p
```

# Install Tor Tor Browser-Launcher

*(Optional depending on use case)*

From the terminal session run the following commands.

```bash
sudo apt update
sudo apt install -y tor torbrowser-launcher
```

```bash
# Run as user, torbrowser-launcher for first time
torbrowser-launcher
```

Do NOT run as root!

Initial instantiation will download and install Tor Browser, including signature verification.

Subsequent instantiations using the same command will update and launch Tor Browser.

# Install alien
**Description:** Useful utility that converts RPM packages to DEB packages.
```bash
sudo apt-get install alien
```

# Install icedtea
**Description:** Enhanced performance for java packages in web browser.
```bash
sudo apt-get install icedtea-netx
```

# Install Terminator
**Description:** User friendly and highly customizable terminal emulator.
```bash
sudo apt-get install terminator
```

# Install WireGuard VPN
Description:  Fast and simple opensource VPN implemented with built in security guarantees.
```bash
apt install wireguard resolvconf
```

**Wireguard setup and configuration is out of scope for this article however there are many great resources and tutorials available online.** 
Ensure you do NOT have any public keys, private keys, pre-shared keys, IP addresses, or any other identifiers in the WireGuard folder and configuration file. You can generate keys and populate the configuration file when required but not include them in a template. 

This may be acceptable cloning a VM you intend to discard for the new clone.      

# Display Settings

**Configure from GUI:** Go to Settings > Display > 1920x1080   16:9
This will vary depending on your hardware. The default actually works well but I get excellent use of my screen real estate with this particular configuration.

# Remove SSH host_keys!
This is required because every VM generates new SSH host_keys when they boot up for the first time. This VM is being prepared for template transformation so existing SSH host_keys must be removed otherwise subsequent VMs will share duplicate SSH host_keys. 

|   |   |
|---|---|
| Navigate to ssh folder | cd /etc/ssh |
| Remove host keys | sudo rm ssh_host_* |
| Confirm | ls -l |

# Remove Machine-ID!
The VM Machine-ID is generated when they boot up for the first time and every VM must have a unique Machine-ID. This VM is being prepared for template transformation so the existing Machine-ID must be removed otherwise subsequent VMs will share duplicate Machine-IDs.  

From a terminal session remove the Machine-ID and build symlink if required.

```bash
# View machine-id
cat /etc/machine-id
```

```bash
# Delete contents of machine-id file
sudo truncate -s 0 /etc/machine-id
```

```bash
# Check for machine-id symlink
ls -l /var/lib/dbus/machine-id
```

```bash
# If no symlink then create one
sudo ln -sf /etc/machine-id /var/lib/dbus/machine-id
```

```bash
# Confirm symlink creation
ls -l /var/lib/dbus/machine-id
```

```bash
# Symlink will looks like
lrwxrwxrwx 1 root root 15 Feb 5 15:33 /var/lib/dbus/machine-id —> /etc/machine-id
```

```bash
# Confirm the machine-id file is empty
cat /etc/machine-id
```

# Clean Up
```bash
sudo apt clean
```

```bash
sudo apt autoremove
```

**Shutdown and do NOT restart prior to cloning or conversion to template!**

**If restart occurs prior to Cloning or conversion to template; Repeat PART 4 (Remove SSH host_keys, Remove Machine-ID, Clean up)**

# Backup Clone Creation

**Create a backup Clone prior to conversion to template.**

Template creation is a destructive process so as a precaution, make a backup clone of the VM intended for conversion to template. This way, if for some reason the templating process fails or is interrupted, the origin VM remains intact. 

**From PROXMOX server view:** Select VM prepared for cloning > Right click and select clone > Follow the wizard prompts > Click on Clone to finish.

<!-- Insert imagae portraying: template creation and concepts -->
![backupclone]({{ "/assets/img/postimages/clonecreatetwo.png" | relative_url }}){:width="60%"}
<!-- _????_ -->

<!-- Insert imagae portraying: template creation and concepts -->
![backupclone]({{ "/assets/img/postimages/clonecreateone.png" | relative_url }}){:width="60%"}
<!-- _????_ -->

The cloning is complete and you now have an identical backup clone of the VM you intend to transform to a template.

# VM Template Creation


**From PROXMOX server view:** Select the VM to be converted to a Template > Right click and select Convert to Template > Click Yes to confirm.

<!-- Insert imagae portraying: template creation and concepts -->
![template-created]({{ "/assets/img/postimages/converttemplatetwo.png" | relative_url }}){:width="60%"}
<!-- _????_ -->

<!-- Insert imagae portraying: template creation and concepts -->
![template-created]({{ "/assets/img/postimages/converttemplateone.png" | relative_url }}){:width="60%"}
<!-- _????_ -->

You now have a template of your fully setup, configured and customized VM. You can create identical VMs that are fully featured, secure, and customized to be fit for purpose in moments from this template. Application, utilities, connectivity and security are already in place with next to no effort. It’s all in the preparation. 

<!-- Insert imagae portraying: template creation and concepts -->
![template-created]({{ "/assets/img/postimages/templatepencilcropone.png" | relative_url }}){:width="40%"}
<!-- _????_ -->

As your machine set up and configuration changes or diversifies over time, you can create a template library of your go to VMs for a given use case or scenario. 

## Final Note
I hope you have enjoyed this article, gained some benefit, or nudged further thoughts and interest on the subject. This is the basic process I use to ensure I can instantiate a new VM for immediate use with minimal effort. This article is a general summary of the process and workflow I use and is not a formal or official instructional document. It is intended to highlight some important elements for considderation when transforming VMs into templates. 

## References
The VM install, configuration, cloning, template creation, and workflow outlined in this article are fairly standard and covered in many online resources. These are some key resources I use to advance my knowledge and understanding of the subject. I’d encourage you to explore their work and channels and support their incredibly valuable and generous contributions to the community. 

[**Learn Linux TV** for their brilliant video series on PROXMOX. (And so much more!)](https://youtu.be/LCjuiIswXGs?list=PLT98CRl2KxKHnlbYhtABg6cF50bYa8Ulo)

[**Sheridan Computers** for an excellent video walk through of bare-metal encrypted disk install of Kali Linux.](https://youtu.be/8N9jKWm-cKY)

This artical is still being finalised so please let me know if I have used any content that requires source atribution. I will ettempt to acknowledge and support the creators of any content I have used or referenced. 