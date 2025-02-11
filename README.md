








## Tweaks Debian Xfce Nvidia



                                              

This tutorial guides you through optimizing a Debian 12 XFCE system with an NVIDIA GPU for maximum performance.
It covers system installation, essential utilities, performance tuning, NVIDIA Oveclocking and CUDA setup.


This is for experienced Linux users who want to testing the performances and understanding the system modifications. 
This guide focuses on achieving the highest possible performance, potentially at the cost of system stability and security.



**Prerequisites:**


*   A computer with an NVIDIA graphics card.

*   Basic knowledge of Linux command-line operations.

*   Understanding of system configuration files.

*   Willingness to accept potential risks associated with advanced system modifications.





## Step 1: Base System Installation and Configuration

In this step, you will install a minimal Debian 12 system, install XFCE, and configure basic system settings.




### 1.1 Install Debian 12 Netinst

- Download lastest Netinst Debian 12 :

https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.9.0-amd64-netinst.iso


- Create usb bootable :

*   From linux : https://gist.github.com/araujo88/6943d4e789c890d9749d19f73bc8c596
*   From Windows : https://devtutorial.io/how-to-create-bootable-usb-installer-for-debian-12-p3123.html


*   Boot from the Debian 12 Netinst image. During the installation process, *only* select "standard system utilities". This ensures a minimal installation with fewer unnecessary packages.



### 1.2 Install Xfce4

- After the base system installation, log in as root and install XFCE:
```
su -
apt install xfce4
```

- Reboot



### 1.3 Configure apt sources

- Edit the apt sources list to include `contrib`, `non-free`, and `non-free-firmware` repositories:
```
su -
nano /etc/apt/sources.list
```

- Replace the entire content of the file with the following:
```
deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

deb http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware
```


### 1.4 Update Apt

- Update the package lists:
```
sudo apt update
```


### 1.5 Set Timezone

- List available timezones:
```
sudo timedatectl list-timezones
```

- Set your timezone (replace `Your/Timezone` with your actual timezone, e.g., `Europe/Paris`):
```
sudo timedatectl set-timezone Your/Timezone
```


### 1.6 Add user to sudoers

- Install `sudo`:
```
su -
apt install sudo
exit
```

- Add your current user to the `sudo` group:
```
sudo usermod -aG sudo $USER
```


### 1.7 Add rights

- Add your user to the `video` and `audio` groups:
```
sudo usermod -aG video $USER
sudo usermod -aG audio $USER
```


### 1.8 Allow members of group sudo to execute any command

**WARNING:** This step significantly reduces system security.  Understand the implications before proceeding.  It allows any user in the `sudo` group to execute *any* command without a password.
```
sudo nano /etc/sudoers
```

- Locate the line:
```
%sudo   ALL=(ALL:ALL) ALL
```

- And replace with:
```
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```
- Reboot





## Step 2: Essential Utilities and Desktop Configuration

Install essential utilities and configure the desktop environment.




### 2.1 Install Essential Utilities
```
sudo apt install acpid dbus-x11 accountsservice apt-transport-https ca-certificates curl software-properties-common -y

sudo apt install mousepad xfce4-terminal nodejs npm lshw net-tools gmtp -y
```


### 2.2 Set Default Applications

- Open Settings > Default Applications > Utilities.  Set "Terminal Emulator" to "Xfce Terminal".



### 2.3 Disable Apparmor

**WARNING:** Disabling AppArmor reduces system security.  Understand the implications before proceeding.  Consider configuring AppArmor profiles instead of disabling it entirely.
```
sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo aa-teardown
```


### 2.4 Dark Mode (Xfce)

Go to Applications > Settings > Appearance and choose "Adwaita Dark".


### 2.4.1 Set Terminal

Go to Settings > Default Applications > Utilities. Set "Terminal Emulator" to "Xfce Terminal".



### 2.5 Install and Configure Firefox ESR
```
sudo apt install firefox-esr
```

#### 2.5.1 Firefox Dark Mode


Go to Settings > General > Manage Colors...

*   Set Text: White
*   Set Background: dark grey
*   Set Unvisited Links: White
*   Set Visited Links: light gray
*   Choose "Always"


#### 2.5.2 Tweaks Firefox ESR

Refer to the Betterfox project for potential Firefox ESR tweaks: [https://github.com/yokoffing/Betterfox/tree/esr128](https://github.com/yokoffing/Betterfox/tree/esr128)





## Step 3: Performance Tuning

Implement various system tweaks to improve performance.



### 3.1 Disable Suspend
```
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```


### 3.2 Create and Configure xset_noblank Script

- Create a script to disable screen blanking:
```
sudo nano /home/$USER/xset_noblank.sh
```

- Add the following content:
```
#!/bin/bash
xset s noblank s noexpose
xset dpms 0 0 0
xset s off
xset -dpms
```


### 3.3 Make script executable
```
sudo chmod +x /home/$USER/xset_noblank.sh
```


### 3.4 Add to autostart

- Go to Applications > Settings > Session and Startup > Application Autostart.

Click "+ Add".
Set Name: xset_noblank
Set Command: /home/$USER/xset_noblank.sh
Uncheck "screen-locker"

- Reboot



### 3.5 Install cpufrequtils and sysfsutils
```
sudo apt install -y cpufrequtils sysfsutils
```

#### 3.5.1 Enable Services
```
sudo systemctl enable cpufrequtils
sudo systemctl enable sysfsutils
```


### 3.6 Disable Core Dumps

- Prevent large core dump files from consuming disk space:
```
sudo nano /etc/sysctl.d/50-coredump.conf
```

- Add the following line:
```
kernel.core_pattern=|/bin/false
```


### 3.7 Enable the service
```
sudo sysctl -p /etc/sysctl.d/50-coredump.conf
```


### 3.8 Configure CPU Governor and Frequencies

- Note: It's generally recommended to use /etc/default/cpufrequtils for configuration if it exists.  This tutorial uses /etc/init.d/cpufrequtils for consistency with the original, but check for the preferred file first.
```
sudo nano /etc/init.d/cpufrequtils
```

- Find and modify (or add) these lines:
```
ENABLE="true"
GOVERNOR="ondemand"
MAX_SPEED="4400000"  # Replace with your CPU's maximum speed (use cpufreq-info to find it)
MIN_SPEED="3500000"  # Replace with approximately half of your CPU's maximum speed
```

- Use cpufreq-info in the terminal to determine your CPU's maximum speed.



### 3.9 Sysfs Tuning
```
sudo nano /etc/sysfs.conf
```

- Add the following lines:
```
devices/system/cpu/cpufreq/ondemand/up_threshold = 99
devices/system/cpu/cpufreq/ondemand/sampling_down_factor = 6
devices/system/cpu/cpufreq/ondemand/sampling_rate = 20000000
vm.swappiness = 10
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
vm.vfs_cache_pressure = 50
module/snd_hda_intel/parameters/power_save = 0
module/snd_hda_intel/parameters/power_save_controller = N
```


### 3.10 Grub Configuration

WARNING: Disabling security mitigations (mitigations=off kernel.randomize_va_space=0) is not recommended and significantly reduces system security.  Understand the risks before proceeding.
```
sudo nano /etc/default/grub
```

- Modify the GRUB_CMDLINE_LINUX_DEFAULT line to:
```
GRUB_CMDLINE_LINUX_DEFAULT="processor.ignore_ppc=1 intel_pstate=passive intel_idle.max_cstate=0 idle=poll nosmt=force pcie_aspm=off mitigations=off kernel.randomize_va_space=0 ipv6.disable=1"
```


### 3.11 Reload Grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
- Reboot



### 3.12 Check Temperatures and CPU Frequencies
```
sudo sensors
sudo cpufreq-info
```




## Step 4: NVIDIA and Tweaks

Install and configure NVIDIA drivers and CUDA, and apply further NVIDIA-specific tweaks.



### 4.2 Xorg Configuration
```
sudo usermod -a -G input $USER
sudo nano /etc/X11/Xwrapper.config
```

- Add the following lines:
```
allowed_users=anybody
needs_root_rights=yes
```

WARNING: Setting allowed_users=anybody is a major security risk.  Avoid this unless absolutely necessary.  Consider using allowed_users=console instead, which is significantly safer.



### 4.3 Install Nvidia Driver and CUDA
```
sudo apt install -y linux-headers-amd64 nvidia-driver firmware-misc-nonfree
```
- Reboot

```
sudo apt install -y nvidia-cuda-dev nvidia-cuda-toolkit
```
- Reboot


#### 4.3.1 CUDA Rights and Group Configuration
```
sudo groupadd cuda
sudo usermod -aG cuda $USER
```

#### 4.3.2 Allow members of group cuda to execute nvidia-smi

WARNING: The modification was overly permissive. 
```
sudo nano /etc/sudoers
```

- Add the following line:

```
%cuda ALL=(ALL) NOPASSWD: /usr/bin/nvidia-smi
```


### 4.4 Nvidia


#### 4.4.1 Nvidia Xorg conf

- Create or edit /etc/X11/xorg.conf.d/20-nvidia.conf (create the directory and file if they don't exist):

```
sudo nano /etc/X11/xorg.conf.d/20-nvidia.conf
```

- Add the following lines:

```
Section "OutputClass"
    Identifier     "nvidia"
    MatchDriver    "nvidia-drm"
    Driver         "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "DPMS" "false"
    ModulePath "/usr/lib/nvidia/current"    # Verify this path exists and adjust if necessary
    ModulePath "/usr/lib/xorg/modules"
    Option     "Coolbits" "28"
EndSection
```

- Important: Verify that the ModulePath entries are correct for your system.  Incorrect paths can prevent X from starting.


#### 4.4.2 Modprobe conf
```
sudo nano /etc/modprobe.d/nvidia-options.conf
```

- Add the following lines:

```
options nvidia NVreg_UsePageAttributeTable=1
options nvidia NVreg_EnableMSI=1
options nvidia NVreg_EnableStreamMemOPs=1
options nvidia NVreg_RegistryDwords="PerfLevelSrc=0x2222"
options nvidia NVreg_RegistryDwords="OverrideMaxPerf=0x1"
options nvidia-drm modeset=1
options nvidia-drm fbdev=1
#Uncomment the line below to allow non-admin users to use profiling tools:
options nvidia-current NVreg_RestrictProfilingToAdminUsers=0
```

- Note: The last line is commented out by default. 
Uncommenting it allows non-admin users to use NVIDIA profiling tools, 
which can be useful but also has security implications.


#### 4.4.4 Update Initramfs
```
sudo update-initramfs -u
```
- Reboot



### 4.5 Nvidia Patch (removes restriction on maximum number of simultaneous NVENC video encoding sessions)
```
git clone https://github.com/keylase/nvidia-patch.git
cd nvidia-patch
sudo bash ./patch.sh
```


### 4.6 Overclocking

WARNING: Overclocking can damage your hardware.  
Proceed with extreme caution, and increase values incrementally, testing
thoroughly after each change.  Monitor temperatures closely.


#### 4.6.1 Benchmark

- Download the Unigine Heaven benchmark: https://benchmark.unigine.com/heaven
```
cd Downloads
sudo chmod +x Unigine_Heaven-4.0.run
sudo ./Unigine_Heaven-4.0.run
cd Unigine_Heaven-4.0
./heaven
```

- Run the benchmark and monitor GPU temperature, clock speeds, and 
memory transfer rate.  Use this as a baseline and to test stability 
after each overclocking step.


#### 4.6.2 Overclock

EXTREME CAUTION: Increase offset values incrementally and run benchmark after each change.  Start with small increases (e.g., +10 MHz for graphics clock, +100 MHz for memory clock).


- Set maximum performance:
```
sudo nvidia-settings -a '[gpu:0]/GpuPowerMizerMode=1'
```

- Set fan control:
```
sudo nvidia-settings -a '[gpu:0]/GPUFanControlState=1'
sudo nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=100'  # 100% fan speed
```

- Set power limit (replace XXX with your GPU's maximum power limit, found using sudo nvidia-smi -q):
```
sudo nvidia-smi --power-limit=XXX
```

- Graphics overclocking (replace XXX with the desired offset, starting with small values like 10):
```
sudo nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffsetAllPerformanceLevels=XXX'  #  Where XXX is the value added to the maximum Memory clocks (exp:120 = +120MHZ)
```

- Memory overclocking (replace XXXX with the desired offset, starting with small values like 100):
```
sudo nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=XXXX' #  Where XXXX is the value added to the maximum Memory clocks (exp:1000 = +1000MHz) 
```

#### 4.6.3 Make changes permanents

- Create a script to apply overclocking settings on startup:
```
sudo nano /home/$USER/GpuTweaks.sh
```

- Add the following lines (replace XXX and XXXX with your stable overclocking values):

```
#!/bin/bash

##Set power limit to maximum
sudo nvidia-smi --power-limit=XXX #  Where XXX is value for power-limit

##Set maximum performance mode

sudo nvidia-settings -a '[gpu:0]/GpuPowerMizerMode=1'

##Set fan control

sudo nvidia-settings -a '[gpu:0]/GPUFanControlState=1'
sudo nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=100'


##Graphics overclocking

sudo nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffsetAllPerformanceLevels=XXX' #  Where XXX is the value added to the maximum graphics clocks

##Memory overclocking

sudo nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=XXXX' #  Where XXXX is the value added to the maximum Memory clocks

####################################################### END ###########################################################
```

- Make script executable:
```
sudo chmod +x /home/$USER/GpuTweaks.sh
```

- Add to autostart (Applications > Settings > Session and Startup > Application Autostart)

Click "+ Add".
Set Name: GpuTweaks
Set Command: /home/$USER/GpuTweaks.sh

- Reboot






## Step 5: Filesystem, Boot, and Service Configurations

Configure filesystem options, disable journaling (with extreme caution), set up autologin, and configure PulseAudio.



### 5.1 Set noatime and Tweak fstab for Ext4
```
sudo nano /etc/fstab
```

- Add the noatime option to your root filesystem entry.  The barrier=0 option is generally not recommended for modern filesystems and hardware.  Example (replace YOUR_ROOT_PARTITION_UUID with the actual UUID of your root partition, found using lsblk -f):
```
UUID=YOUR_ROOT_PARTITION_UUID / ext4 noatime,errors=remount-ro 0 1
```

#### 5.1.1 Relload systemctl
 
```
sudo systemctl daemon-reload
```


### 5.2 Disable Journaling on ext4

EXTREME CAUTION: Disabling journaling significantly increases the risk of data loss in case of a power failure or system crash.  This is not recommended for most users.  The following steps involve creating a backup of your system, which is essential before disabling journaling.


#### 5.2.1 Backup your system

- Download Rescuezilla ( https://github.com/rescuezilla/rescuezilla/releases/download/2.5.1/rescuezilla-2.5.1-64bit.noble.iso ).

Create a bootable USB drive with Rescuezilla :


- Find USB device name :
```
lsblk # Identify your Debian disk (e.g., /dev/nvme0n1)
```

- Unmount USB
```
sudo umount /dev/sdX # Where sdX is your USB
```

- Create the disk image with dd :
```
sudo dd if=~/Downloads /rescuezilla-2.5.1-64bit.noble.iso of=/dev/nvme0n1 bs=1M status=progress 
```
- Boot from it, and back up your entire Debian disk to another storage device 
(USB drive, external hard drive, etc.).  Do not proceed without a complete backup.


#### 5.2.5 Disable journalling on Ext4

WARNING: This can lead to data loss!  Ensure you have a complete backup before proceeding.
```
lsblk    # Identify your root partition (e.g., /dev/nvme0n1p2)

sudo tune2fs -O ^has_journal /dev/nvme0n1p2    # Replace /dev/nvme0n1p2 with your actual root partition!
```


### 5.3 Autologin Lightdm

WARNING: Enabling autologin is a security risk.  Anyone with physical access to your computer will be automatically logged in.
```
sudo nano /etc/lightdm/lightdm.conf
```

- Find or add the following lines within the appropriate sections:

```
[LightDM]
logind-check-graphical=true

[Seat:*]
autologin-user=$USER
autologin-user-timeout=0
```


### 5.4 PulseAudio Configuration
```
sudo nano /etc/pulse/daemon.conf
```

- Uncomment (remove the leading ;) and modify the following lines:

```
flat-volumes = no
avoid-resampling = yes
default-sample-rate = 48000
```


###  5.5 Disable idle
```
sudo nano /etc/pulse/system.pa
```

- Add and Comment this lines:
```
load-module module-udev-detect tsched=0
#load-module module-suspend-on-idle
```


### 5.6 Disable Services

- Disable unnecessary services to reduce boot time and resource usage. 
```
sudo systemctl disable cron.service
sudo systemctl disable avahi-daemon.service
sudo systemctl disable e2scrub_reap.service
sudo systemctl disable logrotate.service
sudo systemctl disable logrotate.timer
sudo systemctl disable systemd-networkd.service  # Only if using a static network configuration
sudo systemctl disable systemd-timesyncd.service
sudo systemctl disable upower.service  # Only if you don't need battery monitoring or power management
sudo systemctl disable apt-daily-upgrade.service
sudo systemctl disable apt-daily.timer
sudo systemctl disable packagekit.service
sudo systemctl disable nftables.service  # Only if you're not using nftables
```


## Further reading


  

  
Debian Xfce4 Minimal Install Guide - https://github.com/coonrad/Debian-Xfce4-Minimal-Install  
Debian Wiki - CpuFrequencyScaling: https://wiki.debian.org/CpuFrequencyScaling
Arch Linux Wiki - Improving performance: https://wiki.archlinux.org/title/Improving_performance
Arch Linux Wiki - Ext4: https://wiki.archlinux.org/title/Ext4
Arch Linux Wiki - LightDM: https://wiki.archlinux.org/title/LightDM
Arch Linux Wiki - PulseAudio/Troubleshooting: https://wiki.archlinux.org/title/PulseAudio/Troubleshooting
NVIDIA Developer Forums: https://forums.developer.nvidia.com/
pyenv github - https://github.com/pyenv/pyenv
Betterfox github - https://github.com/yokoffing/Betterfox/tree/esr128
nvidia-patch github - https://github.com/keylase/nvidia-patch.git
Unigine Heaven benchmark - https://benchmark.unigine.com/heaven
Rescuezilla - https://github.com/rescuezilla/rescuezilla/releases/download/2.5.1/rescuezilla-2.5.1-64bit.noble.iso
