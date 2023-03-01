---
layout: default
title: "Virtual Machine Clone & Template Creation" 
date: 2023-02-24
categories: [Virtual Machine, Hypervisor]
tags: [machine-id, ssh host-keys, proxmox]
---

Important sanitation procedure for creating secure, performant, and glitch free virtual machines, clones and templates.

![Prox Logo]({{ "/assets/img/postimages/xlogo.png" | relative_url }})

## Intro

This article aims to highlight and walk through important sanitation techniques when working with Virtual Machines. Namely, stopping the duplication of Machine-ID and SSH Host-Key during Virtual Machine creation from a template. We will also explore customisation of OS distribution, installing applications and utilities, and optimising system configurations for use case requirements.

Evolving specialised setups and configurations can be organic or prescriptive as desired. You can strip down or bulk up whatever the use-case requirements but there are important baseline delineations to know and adhere to as a given.

# Baseline Delineation

A template should produce a VM that meets a known baseline setup, configuration, and resource allocation. This will help in advance to produce an optimal, secure, and glitch free VM from your templates.

## Principle objectives

These are the principal objectives highlighted in this article but we auction them towards the end of the article to reflect a logical workflow.  

- **Clear Machine-ID:** Every VM should have a unique Machine-ID.  Leaving Machine-ID designated in the template results in duplicate Machine-IDs for every consequent VM. [Removing Machine-IDs](#remove-machine-id) prior to template creation is a must.

![Kity Color]({{ "/assets/img/postimages/kitcattwo.png" | relative_url }})

- **Clear all SSH Host-Keys:** A fresh VM when started for the first time will generate a new set of SSH Host Keys. SSH Host-Keys need to be cleared from the system prior to template creation or risk multiple machines using identical SSH Host-Keys.

![Puple Double]({{ "/assets/img/postimages/icantsee.gif" | relative_url }})

- **Distribution-Upgrade controls:** Maintain prescribed distribution version during system updates. For rolling versions perform Full-Update and Distribution-Upgrade.

## Supplementary & optional objectives

Based on a known or expected use case: Install and customize applications, utilities, security features, network and connectivity, desktop, and display setting.

- Install Applications and utilities.
- Enable Security features.
- Configure Network & Connectivity.
- Customise Desktop and Display setting.

# Scope
This article and workflow correlates to PROXMOX however the fundamental principles will be similar regardless of hypervisor. Cloning and template creation reference PVE (PROXMOX Virtual Environment) being the control centre API for admins and users encompassing access control, resource management, and the entirety of hypervisor features. UX will vary across hypervisors, but their features will be fairly consistent and recognizable.

The Kali Linux ISO-image example used in this article is a Debian based Linux distribution however Ubuntu and most other Linux based distros will be in the ballpark of this article's workflow. Non-Linux operating systems will adhere to the same principals highlighted in this article but implementation will vary.

There are additional techniques for pre-configuring and automating virtual machine cloning and template creation that are well worth learning but they are out of scope for this article.

![Face Reflect]({{ "/assets/img/postimages/norasmall.png" | relative_url }})

# ISO-image
In this section we will download our ISO-image of choice then upload it to the PROXMOX image library.

## Download ISO-image
Download chosen ISO-image to your local machine. In this case I am downloading Kali Linux to my downloads folder using the link below. 

|   |   |
|---|---|
| OS | 64-bit Kali Linux 2022.4 |
| Installer | Complete offline installation with customization, 3.5G |
| ISO Image | https://cdimage.kali.org/kali-2022.4/kali-linux-2022.4-installer-amd64.iso |
| SHA256sum | aeb29db6cf1c049cd593351fd5c289c8e01de7e21771070853597dfc23aada28 |

## Upload ISO-image
Upload the local copy of  your ISO-image to the PROXMOX image library. Within PROXMOX VE navigate to: local node > click on ISO-mage > select Upload and follow popup prompts to complete.

*There are numerous ways to download, upload and add images that are well worth learning and understanding. This example uses the most simple and transparent process.*  


# Create VM from ISO-image

Create the VM that will be customized for cloning and template creation. Within PROXMOX API click on ‘Create VM’ button and follow the wizard, populating fields as detailed in below example. Change input as required in accordance with your use-case, resources availability, and conventions. 

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

# VM initial setup and OS install

## Setup newly created VM from server-view in PROXMOX VE
Select the VM: > Select: Hardware from Summary drop down: > click Remove.

|   |   |
|---|---|
| Hardware | Remove the CD/DVD Drive |

Select the VM: > Select: Options from Summary drop down: > Select QEMU Guest Agent: > Click Edit: > Check box (Use QEMU Agent) and click OK.

|   |   |
|---|---|
| Options | QEMU Guest Agent: (Enable) |

# OS install - First Boot

From PROXMOX VE 

- Start the VM and begin OS installation
- Select the VM: > Go to: >_ Console: > START

The VM will boot up for the first time and present an instillation menu/wizard. 

- Select **Manual Install** and follow prompts populating fields as detailed in below example. i.e. Change input as required, according to your preference.

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
| Finish the instillation | Continue |

Now can choose to continue to OS login where you are prompted for User Name and Password. Use the credentials you configured above to log in for the fist time.

## Validate OS in PVE

From terminal session, validate operating system is successfully running in proxmox virtual environment.  

```bash
# Confirm VM sucsessfully running in proxmox environment
cat /proc/cpuinfo
```

## Update & Upgrade OS distribution

Complete a full Update & Distribution Upgrade.

```bash
# Full update and distrobution upgrade
sudo apt update && sudo apt dist-upgrade
```

## Status check Qemu-Guest-Agent

From terminal session, confirm Qemu-Guest-Agent is installed, alive, and exited.

```bash
# Check status of qemu-guest-agent and start start if required
systemctl status qemu-guest-agent.service
systemctl start qemu-guest-agent.service

# Install and start if required
sudo apt install qemu-guest-agent 
```

# Install & Configure Applications and Utilities

## Install Firewall Utilities

UFW (Uncomplicated Fire Wall), quick and user friendly utility to confirm firewall defaults are correct and enabled. To install, complete the following from terminal session.

### **Install UFW**

```bash
sudo apt-get install ufw
sudo ufw status verbose
sudo ufw enable
sudo ufw enable 
```

### **Install GUFW** GUI extension for UFW Firewall utility

```bash
sudo apt-get install gufw
sudo gufw
```

## Network & Connectivity

### Configure proxychains4

*(Optional depending on use case)*

Navigate to configuration file **proxychains4.conf.**  

```bash
sudo nano /etc/proxychains4.conf
```

Edit configurations file **proxychains4.conf**

|   |   |
|---|---|
| Comment  | # strict_chain |
| UnComment | dynamic_chain |
| [ProxyList]: Add | socks5  127.0.0.1 9050 |

## Enable IP forwarding

Optional depending on use case and is only advised for specific use cases. Understand the security considerations of enabling this feature.

Navigate to configuration file **sysctl.conf.**

```bash
sudo nano /etc/sysctl.conf
```

Edit configuration file **sysctl.conf**

|   |   |
|---|---|
| UnComment lines | net.ipv4.ip_forward = 1 |
|  | net.ipv6.conf.all.forwarding = 1 |

Save and apply changes from terminal session.

```bash
# Save changes then run command to apply change
sysctl -p
```

## Install Tor Tor Browser-Launcher

*(Optional depending on use case)*

From terminal session run the following commands.

```bash
sudo apt update
sudo apt install -y tor torbrowser-launcher
```

```bash
# Run as user, torbrowser-launcher for first time
torbrowser-launcher
```

Do NOT run as root!

Initial instantiation will download and install Tor Browser including the signature verification.

Subsequent instantiations using the same command will update and launch Tor Browser.

## Install alien

**Description:** Useful utility that converts RPM packages to DEB packages

```bash
sudo apt-get install alien
```

## Install icedtea

**Description:** Enhanced performance for java packages in web browser

```bash
sudo apt-get install icedtea-netx
```

## Install Terminator

**Description:** User friendly and highly customizable terminal emulator

```bash
sudo apt-get install terminator
```

## Install WireGuard VPN

Description:  Fast and simple opensource VPN implemented with built in security guarantees

```bash
apt install wireguard resolvconf
```

**Wireguard setup and configuration is out of scope for this article however there are many great resources and tutorials available online.** 

Ensure you do NOT have any public keys, private private keys, pre-shared keys, IP addressing, or any other identifiers in WireGuard folder of configuration file. You can generate keys and populate the configuration when required in a production VM but not in advance and do not include in the template. 

May be acceptable for a direct full clone of a VM in use that you intend to discard for the new full clone.      

## Display settings

**Configure from GUI:** Go to Settings > Display > 1920x1080   16:9

Will vary depending on your hardware. The default actually works well but I get excellent use of my screen real estate with this particular configuration.

# Remove ssh host_keys!

This is required because each new vm creates new ssh host_keys when they first start. This VM is being prepared for template transformation so existing ssh host_keys must be removed otherwise subsequent VMs will share duplicate ssh host_keys. 

|   |   |
|---|---|
| Navigate to ssh folder | cd /etc/ssh |
| Remove host keys | sudo rm ssh_host_* |
| Confirm | ls -l |

# Remove machine-id!

Every VM must have a unique machine-id. Machine IDs are generated on fist boot of the VM. This VM is being prepared for template transformation so existing machine-id must be removed otherwise subsequent VMs will share duplicate machine-ids.  

From Terminal session remove machine-id and build symlink if required.

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

# Clean up

```bash
sudo apt clean
```

```bash
sudo apt autoremove
```

**Shutdown and do NOT restart prior to Full-Cloning or Conversion to Template!**

**If restart occurs prior to Full-Cloning or Conversion to Template; Repeat PART 4 (Remove ssh Host_Keys, Remove machine-id, Clean up)**

# Backup clone creation

**Create a backup Clone prior to conversion to template.**

Template creation is a destructive process so as a precaution, I first make a backup Clone of the VM intended for conversion to template. This way if for some reason the templating process fails or is interrupted, my origin VM remains intact. 

**From Proxmox server view:** Select VM prepared for cloning > Right click and select clone > Follow the wizard prompts > Click on Clone to finish.

The cloning is complete and you now have an identical backup clone of the VM you intend to transform to a template.

# VM Template Creation

**From Proxmox server view:** Select VM to be converted to a Template > Right click and select Convert to Template > Click Yes to confirm.

You now have a template of your fully setup, configured and customized VM. Create identical VMs in moments from this template that are fully featured, secure, and customized to be fit for purpose. Application, Utilities, connectivity and security already in place with next to no effort. It’s all in the preparation. 

As your machine set up and configuration changes or diversifies over time, you can create a template library of your go to VMs for a given use case or scenario. 

# Final note

I hope you have enjoyed this article, gained some benefit, or nudged further thoughts and interest for the subject. This is the basic process I use to ensure I can instantiate a new VM for immediate use with minimal effort. 

This article is a general summary of the process and workflow I use. It is not a formal instructional document. It is intended to highlight a few important things to be attentive to when transforming VMs into templates. 

## References

The VM install, configuration, cloning, template creation, and workflow outlined in this article are fairly standard and covered in many of online sources.  

These are some key resources I use to advance my knowledge and understanding of the subject. I’d encourage you to explore their work and channels and support their incredibly valuable and generous contribution to the community. 

**Learn Linux TV** for their brilliant video series on PROXMOX (And so much more!)

**Sheridan Computers** for an excellent video walk through of bare metal encrypted disk install of Kali Linux.