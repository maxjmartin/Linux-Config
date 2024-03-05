# Fedora SilverBlue / Kinoite G14 2022 Install
System: ROG Zephyrus G14 GA402RJ

Most of instruction came from [Asus Linux](asus-linux.org).

The machine is AMD bases, so no custom kernel will need to be installed.  

I refuse to install RPM Fusion on the machine.  Everything should be a ToolBox, DistroBox or a Flatpak.  This setup will describe how to configure a Distrobox or Flatpak to handle this.  Unless stated otherwise all RPMs should be managed in either environment.  

If pCloud is not installed in the directed folder, the application will not synch.  I think this is an SELinux thing.  haven't figured that out yet.  

Note that the Wayland support configuration for VS Code, is only needed for Gnome.  

## Installation partitioning.
```
512 mb    /boot/efi    fs:efi
1024 mb   /boot        fs:ext4
lvm
  42.5 gb /            fs:btrfs
  <rest>  /var/home    fs:btrfs
```
LVM encrypted

## Set Gnome Fractional Scaling
`gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"`

## Check if update available.
`rpm-ostree upgrade --check`

## To run the update.
`rpm-ostree upgrade`

## Open a terminal window and enter, to configure the `asusctl` and `supergfxctl` repos.
`sudo nano /etc/yum.repos.d/asus.repo`

### Then paste in the following with the key combination 'control + shift + v':
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

## Get Windows VirtIO Drivers.
```
sudo wget https://fedorapeople.org/groups/virt/virtio-win/virtio-win.repo -O /etc/yum.repos.d/virtio-win.repo
```

## Change from Stable to latest VirtIO Drivers.
```
sudo nano /etc/yum.repos.d/virtio-win.repo

[virtio-win-latest]
name=Latest virtio-win builds
baseurl=https://fedorapeople.org/groups/virt/virtio-win/repo/latest
enabled=1    # Change from 0 to 1
skip_if_unavailable=1
gpgcheck=0
```

##  Place the MS Key in /etc/pki/rpm-gpg.
```
cd /etc/pki/rpm-gpg
sudo wget https://packages.microsoft.com/keys/microsoft.asc
```

##  Install the VS Code Repo:
```
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
```

## Open Software Center and ensure the repos are loaded.  Then reboot.
`systemctl reboot`

## Install the following packages: (libosinfo) Already present in Gnome.
```
sudo rpm-ostree install asusctl supergfxctl asusctl-rog-gui distrobox qemu-kvm libvirt virt-install virt-manager virt-viewer edk2-ovmf swtpm qemu-img guestfs-tools libosinfo virtio-win gnome-tweaks gnome-shell-extension-pop-shell gnome-shell-extension-pop-shell-shortcut-overrides gnome-shell-extension-user-theme gnome-shell-extension-dash-to-dock code code-insiders

systemctl reboot
```

## Enable `supergfxctl` and set graphics mode.
```
systemctl enable --now supergfxd.service

systemctl reboot

supergfxctl -m Integrated
```

## Enable Systemctl resources.
```
for drv in qemu interface network nodedev nwfilter secret storage; do \
    sudo systemctl enable virt${drv}d.service; \
    sudo systemctl enable virt${drv}d{,-ro,-admin}.socket; \
done

systemctl reboot
```

## Add yourself to the libvirt group.
`sudo usermod -aG libvirt $USER`

## Confirm KVM installation.
`sudo virt-host-validate qemu`

## Pin the initial setup, and the layered setup.
```
rpm-ostree status

sudo ostree admin pin 0  
sudo ostree admin pin 1 
```

## To unpin later.
`sudo ostree admin pin --unpin  0`

## Install pCloud.
1 - Download app image file.
`https://www.pcloud.com/download-free-online-cloud-file-storage.html`

2 - Place it in the `~/.local/share/applications` directory.

3 -Execute the file, and complete the setup.

## Install Android Studio.
1 - Download app zip file file.
`https://developer.android.com/studio?gad_source=1&gclid=EAIaIQobChMIoaOfhrvahAMVHQ6tBh0YuQjpEAAYASAAEgLI5PD_BwE&gclsrc=aw.ds`

2 - Place it in the `~/.local/share/applications` directory.

3 - Execute the `studio.sh` file, located in the app's bin folder.

4 - Customize the install of the Android SDK to `~/.local/share/applications/Android/SDK`

5 - Once app launches in settings select `Create Desktop Entry...` to add the app to the startup menu.  

## Install the Flathub repo.
`flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo`

Use either Flathub or Fedora Flatpaks as you see fit.  Fedora Flatpaks run on Fedora.  Flathub will be hit or miss, and can affect your system resources.  

## Install the Flathub apps.
`Flatseal, Warehouse, Flatsweep, Main Menu, Steam`

## Configure VS Code to use it own app title header.  
In VS Code press `ctrl+shift+p`, type `settings`, open `settings (JSON)`.  Add `"window.titleBarStyle": "custom"`, to the JSON file.  

In Gnome, create a `~/.config/code-flags.conf` file.  Then insert in the file `--enable-features=UseOzonePlatform --ozone-platform-hint=auto`.  Once done VS Code Insiders will display with Wayland support.  If only operating in a Wayland environment you can use `--ozone-platform=wayland` instead.  

## Optional Gnome Auto Login
`sudo nano /etc/gdm/custom.conf`

Under the [daemon] section, add or modify the following lines:
```
AutomaticLoginEnable=True
AutomaticLogin=[YourUsername]
```

## Configure Distrobox
Place in `~/.config/distrobox`
```
container_manager="podman"
container_image_default="registry.fedoraproject.org/fedora-toolbox:39"
````
That way by default created boxes will share a Fedora runtime.  This will need to be changes with OS upgrades.  

## Create a Calibre Distrobox.
Note: making a distrobox using opensuse, instead of the Flathub or Fedora Flathub packages, due to update delays.  Also I appreciate the openSUSE QA.    

Reference Docs: `https://distrobox.it/usage/distrobox-create/`

```
distrobox create --name openSUE_TW --image registry.opensuse.org/opensuse/distrobox-packaging:latest
```
Run updates and install Calibre.
```
sudo zypper dup
sudo zypper install calibre
distrobox-export --app Calibre
```

You can create a systemd service to perform distrobox-upgrade automatically, this example shows how to run it daily.
```
sudo nano ~/.config/systemd/user/distrobox-upgrade.service
```
Then enter:
```
[Unit]
Description=distrobox-upgrade Automatic Update

[Service]
Type=simple
ExecStart=distrobox-upgrade --all
StandardOutput=null
```
Then create:
```
sudo nano ~/.config/systemd/user/distrobox-upgrade.timer
```
And enter:
```
[Unit]
Description=distrobox-upgrade Automatic Update Trigger

[Timer]
OnBootSec=1h
OnUnitInactiveSec=1d

[Install]
WantedBy=timers.target
```
Then Invoke
```
systemctl --user daemon-reload && systemctl --user enable --now distrobox-upgrade.timer
```
