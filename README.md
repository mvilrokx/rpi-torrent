# Ansible Playbooks to Setup Raspberry Pis

This repository contains a few Ansible Playbooks that you can use to setup Raspberry Pis as an always-on torrent box, complete with Private Internet Access VPN protection ([BYO PIA VPN Account](https://www.privateinternetaccess.com/pages/buy-vpn/vilrokx)) and a [Kill Switch](https://www.privateinternetaccess.com/blog/2018/12/understanding-a-vpn-kill-switch/).

## Prerequisites

- [ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
- [ssh-copy-id](https://www.ssh.com/ssh/copy-id)
- [an SSH Key](https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#adding-your-ssh-key-to-the-ssh-agent)
- a RPi, connected to your network (you know it's IP address) with ssh enabled. If you use [NOOBS](https://www.raspberrypi.org/downloads/noobs/), the easiest way to do this is to follow [this section of the NOOBS README.md](https://github.com/raspberrypi/noobs#how-to-automatically-install-an-os). To enable ssh on the RPi, you need to additionally add an empty file called `ssh` in the root folder of NOOBS, basically the same location as where you put the `wpa_supplicant.conf`.
- a Private Internet Access Account

## Getting Started

1. clone this repository
1. copy your ssh key to the RPi using `ssh-copy-id`
1. run `ansible-playbook new_default.yml` to prepare your RPI. This will prompt you for:
   1. a hostname for your new RPi (use only alphanumeric characters or a dash)
   1. the SSID (name) of the WiFi you want your RPi to connect to
   1. the password of the WiFi you want your RPi to connect to
1. run `ansible-playbook setup.yml --extra-vars "pia_username=<enter-yours> pia_password=<enter-yours> usb_vendor_id='<enter-yours>' usb_model_id='<enter-yours>'"` to setup your RPI as a Torrent Box

## User Guide

### new-default.yml playbook

This playbook can be used for setting up any new RPi; it has nothing specific related to transforming the RPi into a Torrent Box.

When you start the playbook using `ansible-playbook new_default.yml`, it will ask for:

- the hostname you want to give your RPi
- the WiFi SSID you want to connect your RPi to
- the WiFi password you want to connect your RPi to

If you do not have a WiFi, or you don't want to use it, just enter anything for those 2 prompts.

This playbook will perform the following tasks:

- Set Internationalization Options using the values in the new_defaults.yml playbook (you can change them in their if you need to)
- Configure the WiFi using the values you provided at the prompt
- Run apt-get update and apt-get upgrade (this can take a while)
- Set the hostname using the values you provided at the prompt
- re-enable IPTables (used for the Kill Switch later)

It will then reboot the RPi.

### setup.yml playbook

This playbook can be used for setting up your newly configured RPi (using the new-default.yml playbook) as a full blown, always on Torrent Server. It is highly configurable which means it requires quite a few variables. These can be defined in variable files. Check the playbook for the names of these files.

This playbook will perform the following tasks:

- switch on the VNC server
- give the RPi a static IP address
- install all necessary packages
- install and configure Private Internet Access VPN ([you need an account with them](https://www.privateinternetaccess.com/pages/buy-vpn/vilrokx))
- install and configure deluge
- install and configure samba
- install and configure miniDLNA
- configure a usb device as poweroff switch
- configure a VPN kill switch with IPTables

It will then reboot the RPi.

#### Variables

This playbook is configurable and requires some variables. These variables can be places in [ansible varfiles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#defining-variables-in-files). The setup.yml playbook accepts the following var_files (all in the `vars` directory):

- vars/static_ips.yml
- vars/pia_credentials.yml
- vars/deluge.yml
- vars/samba.yml
- vars/usb_ids.yml

##### static_ips variables

| name                      | description                                                                                                        | default            |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------ |
| eth0.ip_address           | The static IP Address you want to give your RPi on the ethernet interface (eth0). Make sure this is not used yet.  | `192.168.0.225/24` |
| eth0.routers              | Your Router's IP Address. This is usually `192.168.0.1`.                                                           | `192.168.0.1`      |
| eth0.domain_name_servers  | The list of DNS Servers you want to use on your RPi.                                                               | `1.1.1.1 1.0.0.1`  |
| wlan0.ip_address          | The static IP Address you want to give your RPi on the ethernet interface (wlan0). Make sure this is not used yet. | `192.168.0.225/24` |
| wlan0.routers             | Your Router's IP Address. This is usually `192.168.0.1`.                                                           | `192.168.0.1`      |
| wlan0.domain_name_servers | The list of DNS Servers you want to use on your RPi.                                                               | `1.1.1.1 1.0.0.1`  |

You can set the variables for `eth0` and `wlan0` to be the same _or_ different (I set them to be the same).

`1.1.1.1` and `1.0.0.1` are the [Cloudflare public DNS](https://www.cloudflare.com/learning/dns/what-is-1.1.1.1/) IP address.

Alternatively, you can also use the PIA DNS Servers: `209.222.18.222` and `209.222.18.218`

> If you want to avoid DNS leaks (test with https://dnsleaktest.com/), do not use the Google DNS (8.8.8.8) as those are [transparent DNS Proxies](https://dnsleaktest.com/what-is-transparent-dns-proxy.html)!

##### pia_credentials variables

| name         | description                                        | default |
| ------------ | -------------------------------------------------- | ------- |
| pia_username | Your Private Internet Access VPN Account User Name |         |
| pia_password | Your Private Internet Access VPN Account Password  |         |

##### deluge variables

| name                  | description | default                                                                               |
| --------------------- | ----------- | ------------------------------------------------------------------------------------- |
| deluge_system_user    |             | `deluge`                                                                              |
| deluge_user_home_dir  |             | `/var/lib/deluge`                                                                     |
| download_location     |             | `torrents/downloading`                                                                |
| move_completed        |             | `torrents/completed`                                                                  |
| regex_include         |             | `.\*`                                                                                 |
| subscription_name     |             | `ShowRSS Subscription`                                                                |
| rssfeed_key           |             | `0`                                                                                   |
| rssfeed_name          |             | `ShowRSS Feed`                                                                        |
| rssfeed_url           |             |                                                                                       |
| rssfeed_site          |             | `showrss.info`                                                                        |
| torrentfiles_location |             | `torrents/torrent-backups`                                                            |
| enabled_plugins       |             | `YaRSS2`                                                                              |
| autoadd_location      |             | `torrents/watch`                                                                      |
| plugins_location      |             | `.config/deluge/plugins`                                                              |
| yarss_egg             |             | `https://bitbucket.org/bendikro/deluge-yarss-plugin/downloads/YaRSS2-1.4.3-py2.7.egg` |

##### samba variables

| name                    | description | default                |
| ----------------------- | ----------- | ---------------------- |
| external_hdd            |             | `/dev/sda1`            |
| external_hdd_samba_name |             | `RaspberryExternalHDD` |
| mount_point             |             | `/media/USBHDD1`       |
| share_directory         |             | `shares`               |

##### usb_ids variables

| name          | description | default  |
| ------------- | ----------- | -------- |
| usb_vendor_id |             | `"7392"` |
| usb_model_id  |             | `"7811"` |

#### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using ansible-playbook's `--tags` flag. The tags available are `update`, `vnc`, `static_ip`, `packages`, `pia`, `deluge`, `samba`, `minidlna`, `usb_poweroff`, `iptables`, `nftables` and `reboot`.

```shell
ansible-playbook setup.yml --tags "pia,deluge,samba,minidlna"
```

TEST KILL SWITCH WITH `sudo systemctl stop openvpn`
