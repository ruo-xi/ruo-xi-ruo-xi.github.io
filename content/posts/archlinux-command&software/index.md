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
- daily
- archlinux

categories:
- archlinux

---

### CommandLine command

#### text 

##### view
* cat    (bat)
* more
* less
* head
* tail

##### edit
* neovim

##### processing
* sed
* awk
* grep
* column
* sort
* uniq
* wc
* shuf
* tee

#### system 

##### monitor
* ps
* top    (bpytop bottom(btm))

##### disk usage
* du (dust)


##### exec command
* xargs

#### File/Dir

##### manager
* ranegr
* if

##### file jump
* cd (zoxide)

##### manage
* rm
* touch
* mkdir
* chmod

##### find
* which
* find    (fd)


#### Net

##### connect
* ssh
* scp

##### info
* ifconfig   (ip)
* netstat   (ss)
* ping
* host   
* dig
* shuf     


#### MISC

##### video player
* mpv

##### audio player
* cmus

##### speedup coding speed
* gtypist

##### file convert
* yj      [toml yaml json HCL]
* mogrify        [image]

##### devices control
* usbUtils   (lsusb)

#### package manager

##### Archlinux
* yay
* paru


### desktop environment

#### xorg
* i3wm
* picom

##### application luncher
* rofi

#### wayland 

##### desktop environment
* sway (waybar, swayidle)
* wayfire 

##### graph env info
* wlr-randr   get the output info

##### screenshot
* grim slurp grimshot

##### terminal emulator
* foot
* kitty
* alacritty

##### application luncher
* wofi

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
