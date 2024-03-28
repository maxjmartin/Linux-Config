# Fedora SilverBlue / Kinoite G14 2022 Install
System: ROG Zephyrus G14 GA402RJ

Most of instruction came from [Asus Linux](asus-linux.org), or the Fedora OS documentation.

The machine is AMD bases, so no custom kernel will need to be installed.  

The intention is that RPM Fusion will not be installed on the machine.  Everything should be a ToolBox, DistroBox or a Flatpak.  This setup will describe how to configure a Distrobox or Flatpak to handle this.  Unless stated otherwise all RPMs should be managed in either environment.  

If pCloud is not installed in the directed folder, the application will not synch.  I think this is an SELinux thing.  haven't figured that out yet.  

VS Code and Edge repos installed due to requirements for my development environment.  Though admittedly Edge can be handled through a Distrobox export.  Choose one and not the other.  Else Edge will crash randomly, and not start back up.  Not certain why this is.  But some programs will not export from a Fedora image Distrobox either.  But they will from an openSUSE box.

Use either Flathub or Fedora Flatpaks as you see fit, though. Fedora Flatpaks run on Fedora. Flathub will be hit or miss, and can affect your system resources.  

Note update frequency concerns between the two Flathub and Fedora Flatpaks, can be quite different.  Because of this I will be defaulting to Flathub, then Fedora Flatpaks.  Otherwise a Fedora Distrobox can used.  

Note that the Wayland support configuration for VS Code, is only needed for Gnome.  

## Installation partitioning.
```
512 mb     /boot/efi    fs:efi
1024 mb    /boot        fs:ext4
lvm
  42.24 gb /            fs:btrfs
  <rest>   /var         fs:btrfs
```
LVM encrypted

## Set Gnome Fractional Scaling
```
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

## Check if update available.
`rpm-ostree upgrade --check`

## To run the update.
`sudo rpm-ostree upgrade --reboot`

## Open a terminal window and enter, to configure the `asusctl` and `supergfxctl` repos.
`sudo nano /etc/yum.repos.d/asus.repo`

## Then paste in the following with the key combination 'control + shift + v':
```
[copr:copr.fedorainfracloud.org:lukenukem:asus-linux]
name=Copr repo for asus-linux owned by lukenukem
baseurl=https://download.copr.fedorainfracloud.org/results/lukenukem/asus-linux/fedora-$releasever-$basearch/
type=rpm-md
skip_if_unavailable=True
gpgcheck=1
gpgkey=https://download.copr.fedorainfracloud.org/results/lukenukem/asus-linux/pubkey.gpg
repo_gpgcheck=0
enabled=1
enabled_metadata=1
```

## Install the initial hardware layer.
- `sudo rpm-ostree install asusctl supergfxctl asusctl-rog-gui --reboot`
- Add ROG-GUI to the autostart list.
- In ROG-GUI settings, set check start minimized.
- Enable supergfxctl and set graphics mode.
```
sudo systemctl enable --now supergfxd.service && \
systemctl reboot
```

### Pin the initial setup, and the layered setup.
- Check the status of the tree `rpm-ostree status`
- Pin the branch you want.  Probably 0.
`sudo ostree admin pin 0`

### To unpin later.
`sudo ostree admin pin --unpin  0`  
Note the number will probably be different later.  So look at the layered packages to identify the right branch.  

## Get Windows VirtIO Drivers.
```
sudo wget https://fedorapeople.org/groups/virt/virtio-win/virtio-win.repo -O /etc/yum.repos.d/virtio-win.repo
```

### Change from Stable to latest VirtIO Drivers.
`sudo nano /etc/yum.repos.d/virtio-win.repo`

then change the latest drivers to enabled.
```
[virtio-win-latest]
name=Latest virtio-win builds
baseurl=https://fedorapeople.org/groups/virt/virtio-win/repo/latest
enabled=1    # Change from 0 to 1
skip_if_unavailable=1
gpgcheck=0
```

## Place the MS Key in /etc/pki/rpm-gpg.
- `cd /etc/pki/rpm-gpg`

- `sudo wget https://packages.microsoft.com/keys/microsoft.asc`

## Install the MS Edge Repo 
```
sudo sh -c 'echo -e "[edge-yum]\nname=edge-yum\nbaseurl=https://packages.microsoft.com/yumrepos/edge/\n
repo_gpgcheck=0\ngpgcheck=0\nenabled=1\ngpgkey=https://packages.microsoft.com/yumrepos/edge/repodata/repomd.xml.key" > /etc/yum.repos.d/packages.microsoft.com_yumrepos_edge.repo'
```

## Install the VS Code Repo:
```
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
```

## Cleanup the Repo Cache
`sudo rpm-ostree cleanup -m`

## Install the following packages: (libosinfo) Already present in Gnome.
```
sudo rpm-ostree install \
qemu-kvm libvirt virt-install virt-manager virt-viewer libosinfo edk2-ovmf swtpm qemu-img guestfs-tools virtio-win \
code code-insiders microsoft-edge-stable \
distrobox \
--reboot
```

## Include these for Gnome
```
gnome-tweaks gnome-shell-extension-pop-shell gnome-shell-extension-pop-shell-shortcut-overrides gnome-shell-extension-user-theme gnome-shell-extension-dash-to-dock 
```

## Add yourself to the libvirt group.
`sudo usermod -a -G libvirt $USER`

## Enable Systemd resources for Virt Manager.
```
for drv in qemu interface network nodedev nwfilter secret storage; do \
    sudo systemctl enable virt${drv}d.service; \
    sudo systemctl enable virt${drv}d{,-ro,-admin}.socket; \
done \
&& systemctl reboot
```

## Confirm KVM installation.
`sudo virt-host-validate qemu`

## Pin the latest OStree Branch
`sudo ostree admin pin 0`

## Install pCloud.
- Download app image file. https://www.pcloud.com/download-free-online-cloud-file-storage.html
- Place it in the `~/.local/share/applications` directory.
- Execute the file, and complete the setup.

## Install Android Studio.
- Download app zip file file at: https://developer.android.com/studio?gad_source=1&gclid=EAIaIQobChMIoaOfhrvahAMVHQ6tBh0YuQjpEAAYASAAEgLI5PD_BwE&gclsrc=aw.ds
- Place the expanded folder in the `~/.local/share/applications` directory.
- Execute the studio.sh file, located in the app's bin folder.
- Customize the install of the Android SDK to `~/.local/share/applications/Android/SDK`
- Once app launches in settings select 'Create Desktop Entry...' to add the app to the startup menu.

## Install the Flathub repo.
`sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo`


## Install the Flatpaks:
Flatseal, Flatsweep, Warehouse, Steam

## Configure VS Code to use its own app title header.
 In VS Code press `ctrl+shift+p`, type `settings`, open `user settings (JSON)`. Add `"window.titleBarStyle": "custom"`, to the JSON file.

### Optional for Gnome Wayland Support
Create a `~/.config/code-flags.conf` file. Then insert in the file `--enable-features=UseOzonePlatform --ozone-platform-hint=auto`. Once done VS Code Insiders will display with Wayland support. If only operating in a Wayland environment you can use `--ozone-platform=wayland` instead.  This only work for VS Code Insiders.  

# Optional Steps

## Gnome Auto Login
- `sudo nano /etc/gdm/custom.conf`
- Under the `[daemon]` section, add or modify the following lines:
```
AutomaticLoginEnable=True
AutomaticLogin=[YourUsername]
```

## Configure Distrobox
- Place in `~/.config/distrobox/distrobox.conf`
```
container_manager="podman"
container_image_default="registry.fedoraproject.org/fedora-toolbox:39"
#container_image_default="registry.opensuse.org/opensuse/distrobox-packaging:latest"
```
That way by default created boxes will share a Fedora runtime. This will need to be changes with OS upgrades.  Or go with the openSUSE image, which is rolling.

## You can create a systemd service to perform distrobox-upgrade automatically, this example shows how to run it daily.
- `sudo nano ~/.config/systemd/user/distrobox-upgrade.service`

- Then enter:
```
[Unit]
Description=distrobox-upgrade Automatic Update

[Service]
Type=simple
ExecStart=distrobox-upgrade --all
StandardOutput=null
```
- Create:
`sudo nano ~/.config/systemd/user/distrobox-upgrade.timer`

- And enter:
```
[Unit]
Description=distrobox-upgrade Automatic Update Trigger

[Timer]
OnBootSec=1h
OnUnitInactiveSec=1d

[Install]
WantedBy=timers.target
```

### Then Invoke - From the same directory.
```
systemctl --user daemon-reload && systemctl --user enable --now distrobox-upgrade.timer
```

## Install Fluedo Codec Pak
`sudo rpm-ostree install oneplay-codec-pack-22-1.x86_64.rpm`


## Enable RPM Fusion Repo
```
sudo rpm-ostree install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm --reboot
```

## Create a Sandboxed Fedora Box
`distrobox create --unshare-all --name sand_fox.`

## Create a Sandboxed openSUSE Box
This assumes Distrobox is configured for Fedora by default.
```
distrobox create --name tumbleSUSE --unshare-all --image registry.opensuse.org/opensuse/distrobox-packaging:latest
```

##   Install RPM Fusion within a Distrobox App
```
sudo rpm-ostree install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

### Add Gstreamer Codecs
```
sudo dnf install @multimedia @sound-and-video ffmpeg-libs gstreamer1-plugins-{bad-*,good-*,base} gstreamer1-plugin-openh264 gstreamer1-libav lame*
```
Alternativley you could also use:
```
dnf install gstreamer1-devel gstreamer1-plugins-base-tools gstreamer1-doc gstreamer1-plugins-base-devel gstreamer1-plugins-good gstreamer1-plugins-good-extras gstreamer1-plugins-ugly gstreamer1-plugins-bad-free gstreamer1-plugins-bad-free-devel gstreamer1-plugins-bad-free-extras
```

## Link to MicroSoft Linux Repos
`https://learn.microsoft.com/en-us/linux/packages`

Usefull for refrencing changes to repos.  
