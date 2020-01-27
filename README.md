# Ansible Playbooks to Setup Raspberry Pis

This repository contains a few Ansible Playbooks that you can use to setup Raspberry Pis as an always-on torrent box, complete with Private Internet Access VPN protection (BYO PIA VPN Account) and a [Kill Switch](https://www.privateinternetaccess.com/blog/2018/12/understanding-a-vpn-kill-switch/).

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
1. run `ansible-playbook deploy.yml --extra-vars "pia_username=<enter-yours> pia_password=<enter-yours> usb_vendor_id='<enter-yours>' usb_model_id='<enter-yours>'"` to setup your RPI as a Torrent Box

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

### deploy.yml playbook

This playbook can be used for setting up your newly configured RPi (using the new-default.yml playbook) as a full blown, always on Torrent Server. It is highly configurable which means it requires quite a few variables. These can be defined in variable files. Check the playbook for the names of these files.

This playbook will perform the following tasks:

- switch on the VNC server
- give the RPi a static IP address
- install all necessary packages
- install and configure Private Internet Access VPN (you need an account with them)
- install and configure deluge
- install and configure samba
- install and configure miniDLNA
- configure a usb device as poweroff switch
- configure a VPN kill switch with IPTables

It will then reboot the RPi.
