Updated driver for rtl88x2bu wifi adaptors based on rtl88x2BU_WiFi_linux_v5.3.1_27678.20180430_COEX20180427-5959 originally downloaded from [D-Link's download page for the DWA-182 Rev D](https://support.dlink.com/ProductInfo.aspx?m=DWA-182).

Build confirmed on:

```
Linux version 4.18.0-0.bpo.1-amd64 (debian-kernel@lists.debian.org) (gcc version 6.3.0 20170516 (Debian 6.3.0-18+deb9u1)) #1 SMP Debian 4.18.6-1~bpo9+1 (2018-09-13)

gcc (Debian 6.3.0-18+deb9u1) 6.3.0 20170516
```
```
Linux version 4.19.0-rc2-amd64 (debian-kernel@lists.debian.org) (gcc version 8.2.0 (Debian 8.2.0-5)) #1 SMP Debian 4.19~rc2-1~exp1 (2018-09-03)

gcc (Debian 8.2.0-9) 8.2.0
```

# DKMS installation
```bash
cd rtl88x2BU_WiFi_linux_v5.3.1_27678.20180430_COEX20180427-5959
VER=$(sed -n 's/\PACKAGE_VERSION="\(.*\)"/\1/p' dkms.conf)
sudo rsync -rvhP ./ /usr/src/rtl88x2bu-${VER}
sudo dkms add -m rtl88x2bu -v ${VER}
sudo dkms build -m rtl88x2bu -v ${VER}
sudo dkms install -m rtl88x2bu -v ${VER}
sudo modprobe 88x2bu
```

# build for raspberry pi
```bash
sudo apt update
sudo apt full-upgrade
sudo apt install bc
sudo apt install raspberrypi-kernel-headers
make
sudo make install
```

# configure wifi ap and bridge
## Step 1: Upgrading Raspberry Pi

First we will update package list available from repositories using

```bash
sudo apt-get update 
```

Once done, we can install these latest packages using

```bash
sudo apt-get upgrade
```

This might take a while depending on your internet connection speed.

Add TipAsk QuestionCommentDownload
## Step 2: Installing Hostadp and Bridge-utils

Once raspberry pi is upgraded.

we need to install a user space background process called hostapd, used for wireless access points and authentication servers. We will also need a package called bridge-utils to manage bridge devices.

```bash
sudo apt-get install hostapd bridge-utils
```

We need to turn off some of the new services that we just installed do it using

```bash
sudo systemctl stop hostapd
```

Debug- Some times raspbian will display message saying hostapd and bridge-utils not found for install command. Do not worry. Run 'sudo apt-get update' once more and it should get resolved.

Add TipAsk QuestionCommentDownload

## Step 2: Disable DHCP Config for Wlan0 and Eth0

Now, we set dhcp background process not to automatically configure wlan0 and eth0 interfaces. We do this by putting following two lines

```nano
denyinterfaces wlan0
denyinterfaces eth0
```

at the end of /etc/dhcpcd.conf file, open it using.

```bash
sudo nano /etc/dhcpcd.conf
```

Add TipAsk QuestionCommentDownload
## Step 3: Creating Bridge Br0

Next, we create a bridge br0 using brctl command which is an Ethernet bridge administrator

```bash
sudo brctl addbr br0
```

and using

```bash
sudo brctl addif br0 eth0
```

command we add eth0 as one of the ports for bridge br0.

Add TipAsk QuestionCommentDownload

## Step 4: Edit /etc/network/interfaces

Now open up a file called interfaces in /etc/network directory

```bash
sudo nano /etc/network/interfaces
```

and add these five lines.

```nano
allow-hotplug wlan0
iface wlan0 inet manual

auto br0
iface br0 inet dhcp
bridge_ports eth0 wlan0
```

First line starts wlan0 interface on a hotplug event. Second line creates a network interface without an IP address which is normally done for bridge elements. Third line starts br0 interface on boot up. Forth line helps in automatic assignment of IP address to br0 interface using DHCP server and finally fifth line connects eth0 interface with wlan0. Save this file and close it.

Add TipAsk QuestionCommentDownload

## Step 5: Edit /etc/hostapd/hostapd.conf

Next, we will configure our wireless access point, we can do this using a file called hostapd.conf in /etc/hostapd folder. Open it up

```bash
sudo nano /etc/hostapd/hostapd.conf
```

and paste these lines.

```nano
interface=wlan0
bridge=br0
ssid=miniProjects
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase=subscribe
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

Value assigned to ssid is the name that access point will use to broadcast its existence. Last five lines are focused on authentication and security of access point. Value of wpa_passsphrase is used as login password which is subscribe in our case. This is a link to document, where you can find definition of each variable that we have used here.

Add TipAsk QuestionCommentDownload

## Step 6: Final Edit /etc/default/hostapd

Finally, open up hostapd file in /etc/default directory

```bash
sudo nano /etc/default/hostapd
```

uncomment DAEMON_CONF line and provide path to file we just created.

```nano
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

This completes setup for raspberry pi to act as router.

# Disable native wifi

edit the file
```bash
sudo nano /etc/modprobe.d/raspi-blacklist.conf
```

and add

```nano
blacklist brcmfmac
blacklist brcmutil
```
