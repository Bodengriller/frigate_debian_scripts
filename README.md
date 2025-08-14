# Google Coral TPU Installation
## Install the TPU driver and library ###

### 1. Installing prerequisites...
```
apt update && apt upgrade -y
apt install -y git devscripts dh-dkms dkms proxmox-headers-$(uname -r)
```

### 2. Adding Coral Edge TPU repository...
```
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | gpg --dearmor -o /etc/apt/keyrings/coral-edgetpu.gpg
echo "deb [signed-by=/etc/apt/keyrings/coral-edgetpu.gpg] https://packages.cloud.google.com/apt coral-edgetpu-stable main" | tee /etc/apt/sources.list.d/coral-edgetpu.list
apt update
apt install -y libedgetpu1-std
```

### 3. Installing driver from source...
Download Git-repo and switch to PullRequest #50 [https://github.com/google/gasket-driver/pull/50]
```
mkdir -p /home/coral-build
git clone https://github.com/google/gasket-driver.git /home/coral-build/gasket-driver
cd /home/coral-build/gasket-driver
git fetch origin pull/50/head:pr-50
git checkout pr-50
debuild -us -uc -tc -b
dpkg -i ../gasket-dkms_*_all.deb
apt update
```
if no error => reboot !

### 4. Verify the drivers are functioning properly
```
lspci -nn | grep 089a
ls /dev/apex_0
```

### For unprivileged LXC containers
```
nano /etc/udev/rules.d/99-chmod777.rules
```
```
SUBSYSTEM=="apex", MODE="0777", GROUP="apex"
KERNEL=="renderD128", MODE="0777"
```
```
udevadm control --reload-rules && udevadm trigger
```

## Reinstall the TPU driver after kernel-update ###
```
apt remove --purge -y gasket-dkms
cd /home/coral-build/gasket-driver
debuild -us -uc -tc -b
dpkg -i ../gasket-dkms_*_all.deb
apt update
```
if no error => reboot !
