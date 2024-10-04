## Instructions how to fix realtek rtl8111 8168 8411 pciexpress gigabit ethernet controller - proxmox-ve_8.2-2
## In My case Mini PC SOYO M2PLUS 16 GB RAM, 512 GB, Intel Celeron N100

### 1. Connect usb-lan adapter or mobile (with enable USB tethering) to USB, monitor by HDMI and boot from USB
Download [Proxmox VE ISO Installer](https://www.proxmox.com/en/downloads/proxmox-virtual-environment/iso "Proxmox VE ISO Installer") and prepare a USB  by [balenaEtcher](https://etcher.balena.io/#download-etcher "balenaEtcher"), after that boot from USB

mobile phone with android [Android USB Internet SHARING](https://support.google.com/android/answer/9059108?hl=en#zippy=%2Ctether-by-usb-cable)

### 2. Connect via ssh or plug keyboard to device and login

Open and edit
```
nano /etc/apt/sources.list
```

Modify your file to:
```
deb http://ftp.pl.debian.org/debian bookworm main contrib non-free non-free-firmware

deb http://ftp.pl.debian.org/debian bookworm-updates main contrib non-free non-free-firmware

# security updates
deb http://security.debian.org bookworm-security main contrib
deb http://download.proxmox.com/debian/pve bookworm pve-no-subscription

# add testing repo
deb http://deb.debian.org/debian testing main contrib non-free non-free-firmware
```
Update repository
```apt update```

Install tools to build and compilations

```apt install build-essential pve-headers dkms```

Install driver

```apt install r8168-dkms```

Blacklist r8169 

```echo "blacklist r8169" >> /etc/modprobe.d/blacklist.conf```
Remove module
```rmmod r8169```

Load r8168
```modprobe r8168```

Check is driver loaded
```lsmod | grep r8168```

```lspci -k```
02:00.0 Ethernet controller: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (rev 2b)
	Subsystem: Realtek Semiconductor Co., Ltd. RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller
	Kernel driver in use: r8168
	Kernel modules: r8168

See new ethernet interface

```ip a```

```root@proxmox:~# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: enp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq master vmbr0 state UP group default qlen 1000
    link/ether 00:e0:4c:0a:17:9d brd ff:ff:ff:ff:ff:ff
3: wlp1s0: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether f0:42:1c:6e:26:66 brd ff:ff:ff:ff:ff:ff
4: vmbr0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:e0:4c:0a:17:9d brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.140/24 scope global vmbr0
       valid_lft forever preferred_lft forever
    inet6 fe80::2e0:4cff:fe0a:179d/64 scope link 
       valid_lft forever preferred_lft forever
```

In my case it is enp2s0

Change Interface

```nano /etc/network/interfaces```

```
auto lo
iface lo inet loopback

iface enx8eea5477f7ee inet manual  # this is my default mobile -> usb interface can be removed

auto vmbr0
iface vmbr0 inet static
        address 192.168.1.140/24
        gateway 192.168.1.1
        bridge-ports enx8eea5477f7ee enp2s0 # -> enp2s0 was added, enx8eea5477f7ee can be removed
        bridge-stp off
        bridge-fd 0

iface wlp1s0 inet manual


source /etc/network/interfaces.d/*

#below section was added
auto enp2s0
iface enp2s0 inet manual
```

If we are connecting via tethering like me you should change the gateway ip address to one that will be supported by your network devices. You should also update the file

```
nano /etc/hosts
```
dns can be change here:
```
nano /etc/resolv.conf
```

and update address

```reboot```

### 3. Disable LID to work without an HDMI cable
If you do not disable LID (like on notebook **L(° O °L**) in the config, it will go to sleep when the HDMI cable is disconnected, and will not boot without a connected monitor. Also will not boot with or not VGA if not disable

	echo -e "HandleLidSwitch=ignore" | tee -a /etc/systemd/logind.conf
### Check
	cat /etc/systemd/logind.conf
![](https://raw.githubusercontent.com/dante1613/Motorcomm-YT6801/main/Screenshots/Proxmox/disabled%20lid.png)
### Reboot
	reboot



