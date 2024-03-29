This document is targeted at RHEL/CentOS 8 edition.

# 1a. Update fresh system install (RHEL edition)
**Run these steps as root or use `sudo` for all commands**

Setup RedHat subscription
```
subscription-manager attach --auto
```

Verify enabled repos
```
subscription-manager repos --list-enabled
```

Update to latest packages
```
dnf upgrade
```


# 1b. Update fresh system install (CentOS edition)
**Run these steps as root or use `sudo` for all commands**

Update to latest packages
```
dnf upgrade
```


# 2. Create certificate for module signing
**Run these steps as root or use `sudo` for all commands**

Create configuration file
```
vim /root/nvidia_ssl_configuration.config
```

With file content
```
[ req ]
default_bits = 4096
distinguished_name = req_distinguished_name
prompt = no
string_mask = utf8only
x509_extensions = myexts

[ req_distinguished_name ]
O = Organization name
CN = NVIDIA Graphics Drivers
emailAddress = your@email.address

[ myexts ]
basicConstraints=critical,CA:FALSE
keyUsage=digitalSignature
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid
```

Create certificate
```
openssl req -x509 -new -nodes -utf8 -sha256 -days 36500 -batch -config /root/nvidia_ssl_configuration.config -outform DER -out /root/public_nvidia.der -keyout /root/private_nvidia.priv
```

Import certificate into keyring. During import process you'll be asked to create password for import. Remember it, you'll used it in the next step.
```
mokutil --import /root/public_nvidia.der
```

Restart system and load certificate into MOK database (pay attention - MOK prompt stays open only for 10s). You'll be asked for password you created in previous step.
```
reboot
```

Check, that certificate is succesfully loaded
```
keyctl list %:.platform
```


# 3. Install Nvidia drivers
**Run these steps as root or use `sudo` for all commands**

Download Nvidia drivers into root home dir and set them as executable
```
chmod +x NVIDIA-$version.run
```

Blacklist nouveau module
```
echo 'blacklist nouveau' >> /etc/modprobe.d/blacklist.conf
```

Install dependencies (replace "Server with GUI" with "Workstation" for RHEL 8 Workstation)
```
dnf groupinstall "Server with GUI" "base-x" "Legacy X Window System Compatibility" "Development Tools"
dnf install elfutils-libelf-devel "kernel-devel-uname-r == $(uname -r)"
```

Back up and rebuild your `initramfs`
```
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r)-nouveau.img
dracut -f
```

Change the default `runlevel`
```
systemctl set-default multi-user.target
```

Reboot system
```
reboot
```

Set environment variable `KBUILD_SIGN_PIN` with password you used to sign your key
```
export KBUILD_SIGN_PIN=your_password
```

Install the driver. If not sure, let driver set X configuration.
```
sh ./NVIDIA-Linux-x86_64-430.50.run --module-signing-secret-key=/root/private_nvidia.priv --module-signing-public-key=/root/public_nvidia.der
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
sudo dnf install google-chrome-stable
```

To update run
```
sudo dnf update google-chrome-stable
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
sudo dnf install code
```


# Adding EPEL (Extra Packages for Enterprise Linux) repository (CentOS edition)

Enable the PowerTools repository since EPEL packages may depend on packages from it.
```
sudo dnf config-manager --set-enabled PowerTools
```

Add EPEL repo
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```


# Adding EPEL (Extra Packages for Enterprise Linux) repository (RHEL edition)

Enable the codeready-builder-for-rhel-8-\*-rpms repository since EPEL packages may depend on packages from it
```
sudo subscription-manager repos --enable "codeready-builder-for-rhel-8-*-rpms"
```

Add EPEL repo
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
```

# Adding RPM Fusion repository

Before adding RPM Fusion repository, make sure to add EPEL repo. Add both free and non-free repositories.

```
sudo dnf install --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-8.noarch.rpm https://download1.rpmfusion.org/nonfree/el/rpmfusion-nonfree-release-8.noarch.rpm
```

*Optional:* add "tainted" RPM Fusion repos [Free](https://rpmfusion.org/FAQ#Free_Tainted)/[Nonfree](https://rpmfusion.org/FAQ#Nonfree_Tainted)
```
sudo dnf install rpmfusion-free-release-tainted rpmfusion-nonfree-release-tainted
```

# Installing VLC

Install VLC
```
sudo dnf install vlc
```
