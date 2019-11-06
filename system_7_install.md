This document is targeted at CentOS 7 edition.

# 1. Update fresh system install (CentOS edition)
**Run these steps as root or use `sudo` for all commands**

Update to latest packages
```
yum upgrade
```


# 2. Create certificate for module signing
**Run these steps as root or use `sudo` for all commands**

Create certificate for module signing
```
openssl req -new -x509 -newkey rsa:2048 -keyout drivers.priv -outform DER -out drivers.der -nodes -days 36500 -subj "/CN=Driver signing/"
```

Import certificate into keyring. During import process you'll be asked to create password for import. Remember it, you'll used it in the next step.
```
mokutil --import /root/drivers.der
```

Restart system and load certificate into MOK database (pay attention - MOK prompt stays open only for 10s). You'll be asked for password you created in previous step.
```
reboot
```

Check, that certificate is succesfully loaded
```
keyctl list %:.system_keyring
```

# 3. Signing modules/drivers for 'SecureBoot' systems
**Run these steps as root or use `sudo` for all commands**

Sign existing modules
```
perl /usr/src/kernels/`uname -r`/scripts/sign-file sha256 /path/name.priv /path/name.der $(modinfo -n modulename)
```

# 4. Install Nvidia drivers
**Run these steps as root or use `sudo` for all commands**

Download Nvidia drivers into root home dir and set them as executable
```
chmod +x NVIDIA-$version.run
```

Blacklist nouveau module in GRUB by editing grub file
```
vim /etc/default/grub
```
Append the following to "GRUB_CMDLINE_LINUX"
```
rd.driver.blacklist=nouveau nouveau.modeset=0
```
Also disable module loading
```
echo 'blacklist nouveau' >> /etc/modprobe.d/blacklist.conf
```
Rebuild GRUB file
```
grub2-mkconfig -o /boot/efi/EFI/centos/grub.cfg
```
Back up and rebuild your `initramfs`
```
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img
dracut -f
```
Change the default `runlevel` to text mode
```
systemctl set-default multi-user.target
```
Reboot system
```
reboot
```

Install dependencies (replace "Server with GUI" with "Workstation" for RHEL 8 Workstation)
```
yum -y groupinstall "GNOME Desktop" "Development Tools"
yum -y install kernel-devel
```

Install NVIDIA driver. If you have SecureBoot enabled, let NVIDIA generate and sign modules. Note certificate location and after installation is finished import it into MOK using `mokutil --import`
```
sh ./NVIDIA-Linux-x86_64-430.50.run
```
If setup won't be able to load libOpenGL.so.0, choose to reinstall libvnd 

Test new driver
```
systemctl isolate graphical.target
```

If everything works, set back `runlevel`
```
systemctl set-default graphical.target
```

Sources used:
- http://us.download.nvidia.com/XFree86/Linux-x86_64/430.50/README/installdriver.html
- https://access.redhat.com/solutions/4134381
- https://gist.github.com/geordanr/5018ad99b7466ac0a446dacdafe739ff
- https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Kernel_Administration_Guide/sect-signing-kernel-modules-for-secure-boot.html


# Installing Google Chrome (with repo)

Create repo file 
``` 
sudo vim /etc/yum.repos.d/google-chrome.repo
```

Add to file
```
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
```

Install chrome
```
sudo yum install google-chrome-stable
```

To update run
```
sudo yum update google-chrome-stable
```


# Installing Visual Studio Code (with repo)

Import Microsoft GPG key
```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
```

Create repo file
``` 
sudo vim /etc/yum.repos.d/vscode.repo
```

Add to file
```
[code]
name=Visual Studio Code
baseurl=https://packages.microsoft.com/yumrepos/vscode
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/keys/microsoft.asc
```

Install code
```
sudo yum install code
```


# Adding EPEL (Extra Packages for Enterprise Linux) repository (CentOS edition)

Enable the PowerTools repository since EPEL packages may depend on packages from it.
```
sudo yum config-manager --set-enabled PowerTools
```

Add EPEL repo
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```


# Adding RPM Fusion repository

Before adding RPM Fusion repository, make sure to add EPEL repo. Add both free and non-free repositories.

```
sudo yum install --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-8.noarch.rpm https://download1.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-8.noarch.rpm
```

*Optional:* add "tainted" RPM Fusion repos [Free](https://rpmfusion.org/FAQ#Free_Tainted)/[Nonfree](https://rpmfusion.org/FAQ#Nonfree_Tainted)
```
sudo yum install rpmfusion-free-release-tainted rpmfusion-nonfree-release-tainted
```

# Installing VLC

Install VLC
```
sudo yum install vlc
```


# Installing Git >2.0 (3rd party repo)

Remove existing git installation (1.8) and install git >2.0
```
sudo yum remove git
sudo rpm -U https://centos7.iuscommunity.org/ius-release.rpm
sudo yum install git2u
```


# Installing Sublime merge

Add sublime repo and install
```
sudo rpm -v --import https://download.sublimetext.com/sublimehq-rpm-pub.gpg
sudo yum-config-manager --add-repo https://download.sublimetext.com/rpm/stable/x86_64/sublime-text.repo
sudo yum install sublime-merge
```