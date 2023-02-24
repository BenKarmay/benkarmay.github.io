---
title: Blog Interest
date:
category:
tags:
---

# Origin VM Templating

# Clean Custom VM for template production

This guide highlights important VM sanitation practices when preparing a VM for both cloning and template creation. You can customize, configure, strip down or bulk up your VM depending on its intended purpose but there are a few important alterations to include as a given.

### Baseline delineation

A template should produce a VM that meets a known baseline setup, configuration, and resource allocation. This will help in advance to produce an optimal, secure, and glitch free VM from your templates.

**Most importantly:**

- **Clear Machine ID**: Every VM needs to have a unique Machine-ID. Machine-ID’s must be cleared prior to conversion to template. If the Machine-ID is left designated in the template then every consequent VM will have identical Machine-ID’s.
- **Clear all SSH Host-Keys:** A fresh VM when started for the first time will generate a new set of SSH Host Keys. SSH Host-Keys should be completely cleared for the system prior to template conversion or risk multiple machines using identical SSH Host-Keys.



Further perform a Full-Update and Distribution-Upgrade, then based on your known or anticipated use case:

- Customize desktop and display configuration
- Install applications and utilities
- Configure security features
- Modify network configuration



This guide uses PROXMOX however the fundamental principles will be same regardless of hypervisor. Instructions for cloning and template creation are in the context of PVE (PROXMOX Virtual Environment) which essentially is the control center API for admins and users encompassing the entirety of PROXMOX management and features. UX will vary across hypervisors but their features will remain fairly consistent and recognizable.

Kali Linux is a Debian based Linux distribution however Ubuntu and most other Linux based distro’s will be in the ballpark of this guides workflow. Non Linux operating systems will adhere to the same principals highlighted here but implementation will vary.

There are additional techniques for pre-configuring and automating virtual machine cloning and template creation that are well worth learning but they are out of scope for this article.

## Preparation for Template Creation of 64-bit Kali Linux 2022.4

---

# Part 1

### ISO-image

Download the ISO-image and add to the image library of PROXMOX

| OS | 64-bit Kali Linux 2022.4 |
| --- | --- |
| Installer | Complete offline installation with customization, 3.5G |
| ISO Image | https://cdimage.kali.org/kali-2022.4/kali-linux-2022.4-installer-amd64.iso |
| SHA256sum | aeb29db6cf1c049cd593351fd5c289c8e01de7e21771070853597dfc23aada28 |

---

### VM creation from ISO-image

Within PROXMOX API, create the VM that will be customized for cloning and template creation. Select xxxx and follow the wizard populating fields as detailed in below example. i.e. Change input as required and according to your needs and resources.

| General | Node: Proxmox |
| --- | --- |
|  | VM ID: 500 |
|  | Name: kaliOriginOne |
|  | Resource Pool: Benjamin |
| OS | Storage: local   |
|  | ISO image: kali-linux-2022.4-installer-amd64.iso |
|  | Type: Linux |
|  | Version: latest |
| System | Leave as default |
| Disks | Enable Discard |
|  | All else leave default |
| CPU | Sockets: 2 |
|  | Cores: 2 |
| Memory (MIB) | 32128 |
| Network | Leave as default |

---

# Part 2

## Configure newly created VM settings from server-view in PROXMOX VE

Select the VM: > Select: **Hardware** from Summary drop down: > click Remove

| Hardware | Remove the CD/DVD Drive |
| --- | --- |

Select the VM: > Select: Options from Summary drop down: > Select QEMU Guest Agent: >Click Edit: > Check box (Use QEMU Agent) and click OK.

| Options | QEMU Guest Agent: (Enable) |
| --- | --- |

---

## First Boot - OS install

From PROXMOX VE

- Start the VM and begin OS installation
- Select the VM: > Go to: >_ Consol: > START

The VM will boot up for the first time and present an instillation menu/wizard.

- Select **Manual Install** and follow prompts populating fields as detailed in below example. i.e. Change input as required, according to your preference.

| Language | English |
| --- | --- |
| Location | Australia |
| Key map | American English |
| Host name | zyto |
| Domain Name | .local |
| Full name for new user | Piper Groff |
| Name for new user | piper |
| Password for new user | iterpop@roz |
| Configure clock | Victoria |
| Language | English |
| Partition disks | (Default) Guided: use entire disk              |
|  | This will be the 34 GB allocated earlier |
|  | All files in one partition |
|  | Finish and write to disk |
| Software selection | Use defaults |
| Install the GRUB boot loader | Select device and enter |
| Finish the instillation | Continue |

Now can choose to continue to OS login where you are prompted for User Name and Password. Use the credentials you configured above.

---

## Validate, Update && Upgrade OS distribution

**Open a terminal session terminal session from the OS GUI:**

Validation operating system is successfully running in proxmox virtual environment

```bash
# Confirm VM sucsessfully running in proxmox environment
cat /proc/cpuinfo
```

Complete a full Update & Distribution Upgrade

```bash
# Full update and distrobution upgrade
sudo apt update && sudo apt dist-upgrade
```

## Status check Qemu-Guest-Agent

**From terminal session in OS GUI:**

Confirm Qemu-Guest-Agent is installed, alive, and exited

```bash
# Check status of qemu-guest-agent and start start if required
systemctl status qemu-guest-agent.service
systemctl start qemu-guest-agent.service

# Install and start if required
sudo apt install qemu-guest-agent
```

---

# Part 3

# Install Applications and Utilities

## UFW Firewall utilities

**Install UFW** (Uncomplicated Fire Wall), quick and user friendly utility to confirm firewall defaults are correct and enabled

```bash
sudo apt-get install ufw
sudo ufw status verbose
sudo ufw enable
sudo ufw enable
```

**Install GUFW** GUI extension for UFW Firewall utility

```bash
sudo apt-get install gufw
sudo gufw
```

---

## Network configuration

### Configure proxychains4

(Optional depending on use case)

Navigate to configuration file **proxychains4.conf**

```bash
sudo nano /etc/proxychains4.conf
```

Edit configurations file **proxychains4.conf**

| Comment  | # strict_chain |
| --- | --- |
| UnComment | dynamic_chain |
| [ProxyList]: Add | socks5  127.0.0.1 9050 |

---

### Enable IP forwarding

This is optional depending on use case and is only advised for specific use cases. Understand the  security considerations of enabling this feature.

Navigate to configuration file **sysctl.conf**

```bash
sudo nano /etc/sysctl.conf
```

Edit configuration file **sysctl.conf**

| UnComment lines | net.ipv4.ip_forward = 1 |
| --- | --- |
|  | net.ipv6.conf.all.forwarding = 1 |

**Apply changes from terminal session as follows**

```bash
# Save changes then run command to apply change
sysctl -p
```

---

## Install Tor Tor Browser-Launcher

(Optional depending on use case)

From terminal session run the following commands

```bash
sudo apt update
sudo apt install -y tor torbrowser-launcher
```

```bash
# Run as user, torbrowser-launcher for first time
torbrowser-launcher
```

Do NOT run as root

First time it will download and install Tor Browser including the signature verification

Subsequent instantiations using the same command will update and launch Tor Browser

---

## **Install alien**

Description: Useful utility that converts RPM packages to DEB packages

```bash
sudo apt-get install alien
```

---

## **Install icedtea**

**Description:** Enhanced performance for java packages in web browser

```bash
sudo apt-get install icedtea-netx
```

---

## **Install Terminator**

**Description:** User friendly and highly customizable terminal emulator

```bash
sudo apt-get install terminator
```

---

## Install WireGuard VPN

Description:  Fast and simple opensource VPN implemented with built in security guarantees

```bash
apt install wireguard resolvconf
```

**Wireguard setup and configuration is out of scope article however there are many great resources and tutorials available online.**

Ensure you do NOT have any public keys, private private keys, pre-shared keys, IP addressing, or any other identifiers in WireGuard folder of configuration file. You can generate keys and populate the configuration when required in a VM but not in advance and do not include in the template.

It’s allowed for a direct full clone of a VM in use that you intend to discard for the new full clone.

---

## Display settings

**Configure from GUI:** Go to Settings > Display > 1920x1080   16:9

Will vary depending on your hardware. The default actually works well but I get excellent use of my screen real estate with this particular configuration.

---

# Part 4

## Remove ssh host_keys

This is required because each new vm creates new ssh key when it first starts. This is going to be a template so they need to be removed so vm created from this template wont already have ssh keys. They would be ssh key that are already in use which would create further issues.

| Navigate to ssh folder | cd /etc/ssh |
| --- | --- |
| Remove host keys | sudo rm ssh_host_* |
| Confirm | ls -l |

---

## Remove machine-id

Every machine need a unique machine-id. This is generated when new machine is created.

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

### Clean up

```bash
sudo apt clean
```

```bash
sudo apt autoremove
```

### ! Shutdown and do NOT restart prior to Full-Cloning or Conversion to Template !

### If restart occurs prior to Full-Cloning or Conversion to Template: Repeat PART 4 (Remove ssh Host_Keys, Remove machine-id, Clean up)

---

# Part 5

## Create a a backup Clone prior to conversion to template

Template creation is a destructive process so as a precaution, I first make a backup Clone of the VM intended for conversion to template. This way if for some reason the templating process fails or is interrupted, my Origin VM remains intact.

From Proxmox server view

select VM to be full cloned

Right click and select clone

Follow prompts:

Click Clone

The cloning is complete and you now have an identical backup clone of the vm you intend to convert to a template.

---

## Create Template from Clone

From Proxmox server view

select VM to be converted to a Template

Right click and select Convert to Template

Click Yes to confirm

You now have a template for your fully configured and customized VM

You can create identical vm in moments and hit the ground running in an OS you are familiar with that will have all your desired features and utilities and connectivity and security already in place with next to no effort.

It’s all in the preparation

---

## Create full clone from template

Now bust it open and take it for a spin.

---

## ? Additional gear to consider adding to template ?

- Install Zen-map
- WireGuard

---

This is a basic process I use to ensure I can fire up a new VM for immediate use and minimal effort. As your machine set up and configuration changes or diversifies over time, you can great a library of your go to templates for a given use case or scenario. Template creation is a destructive process so as a precaution, I first make a Full-Clone of the VM to use for template creation. This way if for some reason the templating process fails or is interrupted, my Origin VM remains intact.

---

## References

The VM install, configuration, Template Creation, cloning and workflow outlined in this article are fairly standard and covered in a plethora of online sources.

### Honorable mentions go to:

- **Learn Linux TV** for their brilliant video series on PROXMOX
- **Sheridan Computers** for an excellent video walk through of bare metal encrypted disk install of Kali Linux.

I’d encourage you to check out their channels and support their incredibly valuable and generous work however you are able.
