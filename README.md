# fedora Post-install Guide 

It is primarily intended for my own use cases, so before copying any command in terminal please click on the sources for more information. You can skip certain steps as you like.  

**Note**: This is **not** a fedora Hardening guide

## Update

Before installing anything make sure to update your system.
To update the system you can paste the below command in the terminal:
```
sudo dnf -y update
```

After the update **reboot** your computer.

***

## Firmware updates

Source: [fwupd](https://fwupd.org/lvfs/vendors/)

Update device firmware, including UEFI, using `fwupd`. Use these commands to update:

```
fwupdmgr refresh
fwupdmgr get-devices
fwupdmgr get-updates
fwupdmgr update
```

***

## Add RPM-fusion Repository

source: [Rpm-fusion config](https://rpmfusion.org/Configuration)

RPM-fusion provides software that fedora doesn't want to ship. 
RPM-fusion have **free** and **non-free** repository. To enable *both* copy the below command in the terminal:

```
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```  

RPM-fusion default to use the openh264 library, so you need the repository to be explicitly enabled

```
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1
```

#### AppStream metadata

```
sudo dnf update @core
```

```
sudo dnf install rpmfusion-\*-appstream-data
```

After adding the repositories **Reboot** you computer.

***

## Multimedia (Codecs)

Source:[RPM-fusion multimedia](https://rpmfusion.org/Howto/Multimedia)

GNOME's default video player doesn't play certain videos in fedora because the supported codecs do not comes pre-installed with fedora. To install these additional codecs copy the following commands in terminal:

#### Switch to full ffmpeg

```
sudo dnf swap ffmpeg-free ffmpeg --allowerasing
```

#### Install additional codec

```
sudo dnf install @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin
```

***

## Hardware Accelerated Codec 

source: Source:[RPM-fusion multimedia](https://rpmfusion.org/Howto/Multimedia)

These codecs help decrease load on the CPU when watching videos online.

#### Intel (recent)

```
sudo dnf install intel-media-driver
```

#### Intel (older)

```
sudo dnf install libva-intel-driver
```

#### AMD

```
sudo dnf install mesa-va-drivers-freeworld
sudo dnf swap mesa-vulkan-drivers{,-freeworld}
```

**for steam or alikes** (AMD)

```
sudo dnf install mesa-va-drivers-freeworld.i686
sudo dnf swap mesa-vulkan-drivers{,-freeworld}.i686
```

After Installing the Multimedia (codecs) **Reboot** your computer.

***

## Flatpak

Source: [Flathub](https://flathub.org/en/setup/Fedora)

*Flathub* is pre-configured as a part of the Third-Party Repositories. If you forgot to click **"Enable third-party Repositories"** option on *Tour*, you can enable Flathub with this command:

```
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

***

## DNS

If you want to set DNS in fedora.

1. Open a Terminal
2. Make sure that systemd-resolved is enabled by running this command:

`sudo systemctl enable systemd-resolved`

3. Open the Settings app and go to Network/wifi. Click on the settings icon for your connected network. On the IPv4 and IPv6 tabs, turn off Automatic using the radio button next to DNS, and leave the DNS field blank, then click on Apply.  Disable and enable the network using the on/off button to make sure it takes effect.

![ipv4.png](/var/home/jackfruit/Documents/Markdown%20Notes/ipv4.png)

4. Edit the following file with nano or your favorite text editor:
	* First copy the config file: `cp /usr/lib/systemd/resolved.conf /etc/systemd/` and  then edit it: `sudo nano /etc/systemd/resolved.conf`
	* Add the following lines in the bottom under [Resolve]. 
		```
		#DNS=9.9.9.9#dns.quad9.net
		#DNS=149.112.112.112#dns.quad9.net
		#DNS=2620:fe::fe#dns.quad9.net
		#DNs=2620:fe::9#dns.quad9.net
		DNSOverTLS=yes	
		```
5. Save the file by pressing Ctrl + O and then Enter, and then Ctrl +X on your keyboard.
6. Create a symbolic link to make sure that /etc/resolv.conf uses systemd-resolved: 

`sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf`

7. Restart systemd-resolved by running this command:

`sudo systemctl restart systemd-resolved`

8. Restart NetworkManager with this command:

`sudo systemctl restart NetworkManager`

9. Verify the DNS settings with:

`resolvectl status`

**Confirm you're using Quad9 by visiting [on.quad9.net](https://on.quad9.net/) in your preferred browser.**

**In case it doesn't work, change the setting in** `/etc/systemd/resolved.conf`

***

## Privacy Tweaks

#### System counting

Source: [Privacy guides](https://www.privacyguides.org/en/os/linux-overview/#privacy-tweaks)

The Fedora Project [counts](https://fedoraproject.org/wiki/Changes/DNF_Better_Counting) how many unique systems access its mirrors by using a countme variable instead of a unique ID. Fedora does this to determine load and provision better servers for updates where necessary.

This [option](https://dnf.readthedocs.io/en/latest/conf_ref.html#options-for-both-main-and-repo) is currently off by default. We recommend adding `countme=false` to `/etc/dnf/dnf.conf` just in case it is enabled in the future.

1. Open a Terminal
2. Paste this in terminal

`sudo nano /etc/dnf/dnf.conf` 

3. Add the following lines in the bottom:

`countme=false`

4. Save the file by pressing Ctrl + O and then Enter, and then Ctrl +X on your keyboard.

#### MAC Address Randomization

Source: [Privacy guides](https://www.privacyguides.org/en/os/linux-overview/#privacy-tweaks)

This provides a bit more privacy on Wi-Fi networks as it makes it harder to track specific devices on the network you’re connected to. It does not make you anonymous.

1. Open the Terminal
2. Create a new file 

`sudo nano /etc/NetworkManager/conf.d/00-macrandomize.conf`

3. Add the following to it:

```
[device]
wifi.scan-rand-mac-address=yes

[connection]
wifi.cloned-mac-address=random
ethernet.cloned-mac-address=random
```

4. Save the file by pressing Ctrl + O and then Enter, and then Ctrl +X on your keyboard.
5. Then, restart NetworkManager:

`systemctl restart NetworkManager`

#### Fedora-maintained hardening for general use cases 

If you want a **security-focused distro** check out [Secureblue](https://secureblue.dev/) 

[Learn more](https://forge.fedoraproject.org/security/docs/src/branch/moduleseparation/modules/topics/pages/hardening.adoc)

The following command enables the **self-updating kernel hardening**, which changes several parameters of the kernel at the next boot:

```
sudo ln -s /usr/share/doc/systemd/99-kernel-hardening.conf /etc/sysctl.d/
```

**Note**: Do **not** copy (`cp`) this file and do not use hard links (`ln`)! The self-updating depends on using a symlink (`ln -s`)! Otherwise its reliability cannot be guaranteed!

You can **remove** this hardening again with:

```
sudo rm -fr /etc/sysctl.d/99-kernel-hardening.conf
```

The following commands enable the **Fedora-maintained SELinux & firewalld hardening**, which changes some settings of SELinux and the firewalld beginning at the next boot:

```
sudo setsebool -P deny_ptrace on
sudo firewall-cmd --set-default-zone=public
```

You can **remove** this hardening again with:

```
sudo setsebool -P deny_ptrace off
sudo firewall-cmd --set-default-zone=FedoraWorkstation
```

***

## Hardened Browser for fedora 

Source: [Secureblue](https://github.com/secureblue/Trivalent)

<img src="/var/home/jackfruit/Documents/Markdown%20Notes/trivalent.png" width="100"> 

By default fedora includes *Firefox* which is **not** recommended as they're currently much more vulnerable to exploitation and inherently add a huge amount of attack surface compared to **Chromium**. [Why?](https://grapheneos.org/usage#web-browsing)

[Trivalent](https://github.com/secureblue/Trivalent) Browser is Available for Fedora and other fedora based distros. To install it on fedora you need to add the secureblue [repository](https://repo.secureblue.dev/secureblue.repo) in fedora.

1. Add the following repository:

```
sudo dnf -y config-manager addrepo --from-repofile=https://repo.secureblue.dev/secureblue.repo
```

2. Install Trivalent

```
sudo dnf -y install trivalent
```

***

## Reboot 

After all the above make sure you have **rebooted** you computer to apply changes we made, before using it.

***

## Theming & extensions

You can theme you system if you prefer, There is an app called [Rewaita](https://flathub.org/en/apps/io.github.swordpuffin.rewaita) for that, also try to use few/popular extensions on GNOME as they tend to pose security risks.  

***

## Apps & Repositories

I recommend you install Apps from **trusted/official** repositories where possible and try to stick to *Flatpak* and *system-packages* first.

If you are going to use Flatpak you can install **[flatseal](https://flathub.org/en/apps/com.github.tchx84.Flatseal)** for permission management, **[Bazzar](https://flathub.org/en/apps/io.github.kolunmi.Bazaar)** for downloading flatpaks.  



