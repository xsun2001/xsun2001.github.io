+++
title = "Play with NAS $ Part 1: Initial configuration"
date = 2025-09-16

[taxonomies]
categories = ["NAS"]
tags = ["NAS", "linux", "network"]
+++

### Do I need a NAS?

After losing all my data in an external Samsung T7 SSD, I finally decided to buy a NAS (Network Attached Storage). The main motivations include:

<!-- more -->

1. Sync my (and maybe my family's) photos and replace the Apple Cloud.
2. TimeMachine backup for my MacBook Pro. The old 2TB WD Element SE may be broken because its IO speed drops severely. I want a plug-and-play experience and external drives cannot safely do that.
3. Save my videos (mainly game recording) and movies to free the space in my gaming laptop.
4. Also I just want to have a NAS.

### The hardware configuration of my NAS

In fact, I have searched and learnt about building a NAS for several mouths. From M-ATX or ITX chassis and motherboard, to efficient CPU and memory combination; from best HDD for NAS (even refreshed server disks), to network bandwidth. I just cannot find the perfect configuration to fulfill all of my imagination to a NAS, including TrueNAS OS with high-performance ZFS (which recommends to use a lot of system memory and also ECC support) and also a powerful CPU for future workloads. An old supermicro X10SDV supports ECC memory but has very poor single thread performance. A wired taobao motherboard which equipped with a special ES AMD Zen processor seems to support ECC memory, but it may not be stable enough for all my data.

So I just gave up and turn to pre-built NAS system. Finally, I chose a UGREEN DXP4800 with 4x 4T Seagate IronWolf NAS HDD. All of them only need 4325.65¥ with 15% national discount. It equips with an Intel N100 CPU, 8GB memory and 2x 2.5Gbps network ports. I have to admit that this CPU is a bit shabby when compared with the candidates I mentioned above. And of course it doesn't support fancy ZFS, ECC or other cool feature. However, it does come up with additional advantages:

1. It is pre-built. It is challenging to build a compact ITX server for the first time. Not to mention the additional SATA and power cables for NAS.
2. It has a decent user app. I don't need to choose, install and configure softwares for most of my requirements, including photo sync, TimeMachine, SMB and WebDAV.
3. It is beautiful. Yes, I prefer its good look, especially when it needs to be place on my working table.
4. Full control. I still have full ssh and root permission to access the terminal and install any debian packages. And docker support is smooth too.

I think I have had enough things to do in my life. I want a solution for my requirements with reasonable price and smooth experience. I may devote more time for the main devices, and NAS is not included in this list. It just needs to hold all the data and perform its background jobs with reliable stability.

### Tough setup process with campus network

The setup process of UGREEN NAS should be simple. Install the disks, connect the power cable and internet cable, press the boot button, open its APP to start initialization. However, in campus environment, authentication is required to grant access to the network. After hours of investigation, I finally find the method to overcome this issue. I need to connect my MBP to phone's hotspot network, then connect it to the NAS with the "Internet Sharing" enabled. Then my MBP will become a simple router and assign a local IP to the NAS for accessing. It allows me to finish its initialization using the APP. I used normal ext4 format and RAID5 array for the 4 disks.

After that, I need to config the NAS to correctly connect to campus wifi and cabled network. To achieve it, I need to ssh into the NAS and install `wpasupplicant`:

```bash
sudo apt update && sudo apt install wpasupplicant -y
```

UGREEN's NAS OS is modified from Debian 12 (bookworm). Good choice.

UGREEN OS uses legacy `if-up`/`if-down` and `/etc/network` to manage networks. To maintain the functionality of its UI and better compatibility, I do not install NetworkManager or other tools to replace it. Instead, I will use `wpasupplicant`'s systemd service to handle the authentication. Create `/etc/wpa_supplicant/wpa_supplicant-wired-eth0.conf` with following content:

```
ctrl_interface=/var/run/wpa_supplicant
update_config=1
ap_scan=0

network={
    key_mgmt=WPA-EAP
    eap=PEAP
    identity="<username>"
    password="<password>"
    phase2="auth=MSCHAPV2"
}
```

Then start the corresponding service:

```bash
systemctl enable --now wpa_supplicant-wired@eth0.service
```

The config file is automatically used for `eth0` network device with `wired` driver. For wireless network, create `/etc/wpa_supplicant/wpa_supplicant-wlan0.conf` with additional `ssid="<ssid>"` config and use `wpa_supplicant@eth0.service`. However, for my system, the USB wifi stick is very unstable. So currently I only use the `eth0`.

That all for the initial configuration of the UGREEN NAS. The photo sync is easily handled by its iOS app. All my recoding footages and movies are transmitted via SMB and can be viewed through its "Movie Center". TimeMachine backup is archived by enabling Bonjour service and setting up the corresponding shared directory with given capacity (3TB for me). The next blog will focus on the Docker services on my NAS.