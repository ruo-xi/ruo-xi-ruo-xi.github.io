---
title: "Archlinux Command&software"
date: 2022-05-11T01:15:40+08:00
weight: 0

toc:
  enable: true

math:
  enable: false

comment: true
hiddenFromHomePage: false
hiddenFromSearch: false

lightgallery: true
seo:
  images: []

summary: "dialy uesd linux command"
description: ""
keywords: ""

resources:
- name: featured-image
  src: featured-image.jpg
- name: featured-image-preview
  src: featured-image-preview.jpg

draft: true

subtitle: ""

tags:

categories:

---

### CommandLine command

#### text resolve
* sed
* awk
* grep
* column
* sort
* uniq
* wc
* shuf     
* tee

#### System Monitor
* ps

#### File Manage
* rm
* touch

#### text view
* cat
* more
* less
* head
* tail

#### find file
* which
* find

#### Command
* xargs

#### Net
* ssh
* scp
* ifconfig   ip
* netstat   ss
* ping
* host   dig
* shuf     

### file convert
* yj      (toml yaml json HCL)
* mogrify        (image)



### desktop environment

#### xorg
* i3wm
* picom
* rofi

#### wayland 
* sway
* wayfire
* foot
* kitty
* wofi
##### screenshot
* grim slurp grimshot


### emulator

#### anbox
##### install
```bash
yay -S anbox-git android-tools anbox-modules-dkms
```
##### enable and mount the kernel modules
```bash
sudo modprobe binder_linux devices=binder,hwbinder,vndbinder,anbox-binder,anbox-hwbinder,anbox-vndbinder
sudo modprobe ashmem_linux
sudo mkdir -p /dev/binderfs
sudo mount -t binder binder /dev/binderfs

echo "ashmem_linux
binder_linux" | sudo tee -a /etc/modules-load.d/anbox.conf 
```
##### enable network via NetwarkManager

NetworkManager will automatically set up the bridge every reboot, so you only need to execute the command once.

```bash
nmcli con add type bridge ifname anbox0 -- connection.id anbox-net ipv4.method shared ipv4.addresses 192.168.250.1/24
```
##### enable anbox service
```bash
sudo systemctl enable --now anbox-container-manager.service
```
##### install app
```bash
adb install path/to/app.apk
```


#### qemu
#### virtual box
