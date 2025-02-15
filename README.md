








## Minimal Debian Xfce performance tuning Guide




                                              

This tutorial guides you through optimizing and install a Debian 12 XFCE system.
It covers system installation, essential utilities, Overclocking and CPU tuning.


This is for experienced Linux users who want to testing the performances and understanding the system modifications. 
This guide focuses on performance, potentially at the cost of system stability and security.



**Prerequisites:**



*   Basic knowledge of Linux command-line operations.

*   Understanding of system configuration files.

*   Willingness to accept potential risks associated with advanced system modifications.





## Step 1: Base System Installation and Configuration





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


```NOPASSWD:```  This means no password will be required when using sudo to run the following command.

- Reboot

  

## Step 2: Essential Utilities and Desktop Configuration





### 2.1 Install Essential Utilities
```
sudo apt install acpid dbus-x11 accountsservice apt-transport-https ca-certificates software-properties-common -y

sudo apt install mousepad xfce4-terminal -y
```

acpid:                        Handles Advanced Configuration and Power Interface (ACPI) events (e.g., lid close, power button).

dbus-x11:                     Essential inter-process communication for X11 applications.

accountsservice:              Manages user accounts and login sessions.

apt-transport-https:          Enables APT to download packages over HTTPS for secure updates.

ca-certificates:              Provides root certificates for verifying the authenticity of SSL/TLS connections.

software-properties-common:   Tools for managing software repositories (PPAs).

mousepad:                     Lightweight text editor.

xfce4-terminal:               Terminal emulator for running commands.



### 2.2 Set Default Terminal

Open Settings > Default Applications > Utilities

*   Set "Terminal Emulator" to "Xfce Terminal".



### 2.3 Disable Apparmor

**WARNING:** Disabling AppArmor reduces system security.  Understand the implications before proceeding.  Consider configuring AppArmor profiles instead of disabling it entirely.
```
sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo aa-teardown
```


### 2.4 Dark Mode (Xfce)

Open  Applications > Settings > Appearance

*   choose "Adwaita Dark".


### 2.5 Set Terminal

Go to Settings > Default Applications > Utilities

*   Set "Terminal Emulator" to "Xfce Terminal".



### 2.6 Install and Configure Firefox ESR
```
sudo apt install firefox-esr
```

- Firefox Dark Mode


Go to Settings > General > Manage Colors...

*   Set Text: White
*   Set Background: dark grey
*   Set Unvisited Links: White
*   Set Visited Links: light gray
*   Choose "Always"


- Tweaks Firefox ESR

Refer to the Betterfox project for potential Firefox ESR tweaks: [https://github.com/yokoffing/Betterfox/tree/esr128](https://github.com/yokoffing/Betterfox/tree/esr128)





## Step 3: Performance Tuning





### 3.1 Disable Suspend
```
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```


### 3.2 Create and Configure xset_noblank Script

- Create a script to disable screen blanking and DPMS:
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


- Make script executable
```
sudo chmod +x /home/$USER/xset_noblank.sh
```


- Add to autostart

Open to Applications > Settings > Session and Startup > Application Autostart.

*   Click "+ Add".
*   Set Name: xset_noblank
*   Set Command: /home/$USER/xset_noblank.sh
*   Uncheck "screen-locker"

- Reboot



### 3.3 Install cpufrequtils and sysfsutils
```
sudo apt install -y cpufrequtils sysfsutils
```

- Enable Services
```
sudo systemctl enable cpufrequtils
sudo systemctl enable sysfsutils
```


### 3.4 Disable Core Dumps

- Prevent large core dump files from consuming disk space:
```
sudo nano /etc/sysctl.d/50-coredump.conf
```

- Add the following line:
```
kernel.core_pattern=|/bin/false
```


### 3.5 Enable the service
```
sudo sysctl -p /etc/sysctl.d/50-coredump.conf
```


### 3.6 Configure CPU Governor and Frequencies

- Note: It's generally recommended to use /etc/default/cpufrequtils for configuration if it exists.  This tutorial uses /etc/init.d/cpufrequtils for consistency with the original, but check for the preferred file first.
```
sudo nano /etc/init.d/cpufrequtils
```

- Find and modify these lines:
```
ENABLE="true"
GOVERNOR="ondemand"
MAX_SPEED="XXXXXXX"  # Replace with your CPU's maximum speed (use cpufreq-info to find it)
MIN_SPEED="XXXXXXX"  # Replace with approximately half of your CPU's maximum speed
```

- Use cpufreq-info in the terminal to determine your CPU's maximum speed.
  
- Recommendation:
Even with C-states disabled, the ondemand governor might still be a better choice for gaming due to its ability to manage Turbo Boost and frequency scaling within the available range.  However, the best way to know for sure is to benchmark.



### 3.7 Sysfs Tuning
```
sudo nano /etc/sysfs.conf
```

- Add the following lines:
```
devices/system/cpu/cpufreq/ondemand/up_threshold = 99 # Percentage CPU utilization before scaling up frequency. Higher values mean more responsiveness.
devices/system/cpu/cpufreq/ondemand/sampling_down_factor = 6 # How aggressively the CPU scales down frequency. Higher values mean less aggressive scaling down.
devices/system/cpu/cpufreq/ondemand/sampling_rate = 20000000 # How often the CPU usage is checked. Higher values mean more frequent checks (more responsive).
vm.swappiness = 10 # Represents the kernel's preference (or avoidance) of swap space.(better performance if enough RAM).
vm.dirty_background_ratio = 5 # Percentage of "dirty" memory before background write to disk starts.
vm.dirty_ratio = 10 # # Percentage of "dirty" memory before writes to disk are forced.
vm.vfs_cache_pressure = 50 # How aggressively the kernel reclaims memory from the VFS cache. Lower values mean less aggressive reclaiming.
```
- Check Temperatures and CPU Frequencies
```
sudo sensors
sudo cpufreq-info
```

### 3.8 Grub Configuration

WARNING: Disabling security mitigations (mitigations=off kernel.randomize_va_space=0) is not recommended and significantly reduces system security.  Understand the risks before proceeding.
```
sudo nano /etc/default/grub
```

- Modify the GRUB_CMDLINE_LINUX_DEFAULT line to:
```
GRUB_CMDLINE_LINUX_DEFAULT="intel_pstate=passive intel_idle.max_cstate=0 idle=poll nosmt=force pcie_aspm=off mitigations=off kernel.randomize_va_space=0 ipv6.disable=1 zswap.enabled=0"
```
- Parameters :

```intel_pstate=passive```          Use the older intel_pstate driver in passive mode (might reduce performance on newer CPUs).
```intel_idle.max_cstate=0```       Disable all CPU C-states (low-power states).
```idle=poll```                     Use polling for idle instead of interrupts (can increase power consumption but reduce latency).
```nosmt=force```                   Disable Symmetric Multi-Threading (Hyperthreading may hurt the performance, do benchmarks).
```pcie_aspm=off```                 Disable PCIe Active State Power Management (can improve latency but increase power consumption).
```mitigations=off```               Disable kernel mitigations for CPU vulnerabilities (increases risk but might improve performance).
```kernel.randomize_va_space=0```   Disable address space layout randomization (security risk, might slightly improve performance).
```ipv6.disable=1```                Disable IPv6 networking.
``` zswap.enabled=0```              Disable kernel feature that provides a compressed RAM cache for swap pages (better performance if enough RAM).

- Reload Grub
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
- Reboot



## Step 4: NVIDIA Tweaks



### 4.1 Disable compositor

Open to Applications > Settings > Windows Manager Tweaks > Compositor

*   Uncheck  "Enable display compositing"

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


- CUDA Rights and Group Configuration
```
sudo groupadd cuda
sudo usermod -aG cuda $USER
```

- Allow members of group cuda to execute nvidia-smi

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


- Configuring the 10-nvidia-drm-outputclass :
```
sudo nano /usr/share/X11/xorg.conf.d/nvidia-drm-outputclass.conf
```

```
Section "OutputClass"
    Identifier     "nvidia"
    MatchDriver    "nvidia-drm"
    Driver         "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "DPMS" "false"
    ModulePath "/usr/lib/nvidia/current"    # Verify this path exists and adjust if necessary
    ModulePath "/usr/lib/xorg/modules"
    Option     "Coolbits" "28" # Enable Overclocking and FanControl
EndSection
```

- Important: Verify that the ModulePath entries exist for your system.  Incorrect paths can prevent X from starting.


#### 4.4.2 Modprobe conf
```
sudo nano /etc/modprobe.d/nvidia-options.conf
```

- Add the following lines:

```
options nvidia NVreg_UsePageAttributeTable=1 # Can improve performance, especially for graphics-intensive applications, by optimizing memory access patterns.
options nvidia NVreg_EnableMSI=1 # Reduces CPU overhead associated with handling interrupts from the graphics card, potentially improving system responsiveness.
options nvidia NVreg_EnableStreamMemOPs=1 # Can improve performance in applications that involve significant data transfer between the CPU and GPU.
options nvidia NVreg_RegistryDwords="PerfLevelSrc=0x2222" # Performance level
options nvidia NVreg_RegistryDwords="OverrideMaxPerf=0x1" # Max performance (caution: heat/power)
options nvidia-drm modeset=1 # Enable kernel mode setting (modeset)
options nvidia-current NVreg_RestrictProfilingToAdminUsers=0 # Allow non-administrative users to use NVIDIA profiling tools.  This can be useful for debugging and performance analysis but might pose a security risk in some environments
```



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
memory transfer rate to test stability after each overclocking step.


#### 4.6.2 Overclock

EXTREME CAUTION: Increase offset values incrementally and RUN BENCHMARK AFTER EACH CHANGES.  Start with small increases (e.g., +10 MHz for graphics clock, +100 MHz for memory clock).


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
sudo nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=XXX' # Where XXX is % of Fan Speed


##Graphics overclocking

sudo nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffsetAllPerformanceLevels=XXX' #  Where XXX is the value added to the maximum graphics clocks

##Memory overclocking

sudo nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=XXXX' #  Where XXXX is the value added to the maximum Memory clocks

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




### 5.1 Set noatime and Tweak fstab for Ext4
```
sudo nano /etc/fstab
```

UUID=YOUR_ROOT_PARTITION_UUID:  Identify your partition, don't touch it

ext4: This specifies the filesystem type.

noatime: This mount option disables updating the access time (atime) for files 
whenever they are read.  This can improve performance, especially on 
solid-state drives (SSDs), as it reduces unnecessary writes.

barrier=0:  This option disables write barriers. Write barriers ensure data integrity by
 forcing writes to be completed in a specific order. Disabling them can 
significantly improve performance, especially on SSDs, but it comes with
 a risk: if your system crashes or loses power during a write operation,
 data corruption can occur.  Use with caution!  If you're unsure, leave barrier=1 (the default).


- Add options
```
UUID=YOUR_ROOT_PARTITION_UUID / ext4 noatime,errors=remount-ro 0 1
```


- Reload systemctl
 
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


#### 5.2.2 Disable journalling on Ext4

WARNING: This can lead to data loss!  Ensure you have a complete backup before proceeding.
```
lsblk    # Identify your root partition (e.g., /dev/nvme0n1p2)

sudo tune2fs -O ^has_journal /dev/nvme0nXXX    # Replace /dev/nvme0nXXX with your actual root partition! (you can find it with : " lsblk ")
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


#### 5.4.1 Disable idle
```
sudo nano /etc/pulse/system.pa
```

- Add and Comment this lines:
```
load-module module-udev-detect tsched=0 # disables the timer-based scheduling feature of the module.  This can improve performance, but might introduce latency
#load-module module-suspend-on-idle
```


### 5.5 Disable Services

- Disable unnecessary services to reduce boot time and resource usage. 
```
sudo systemctl disable systemd-networkd.service  # Only if using a static network configuration
sudo systemctl disable upower.service  # Only if you don't need battery monitoring or power management
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



## Troubleshootings


### Sound output is wrong (headphones/lineout...)

- Use pavucontrol to change the port to your desired one. Then find the internal name of the port with this command:
```
$ pacmd list | grep "active port"
    active port: <hdmi-output-0>
    active port: <analog-output-lineout>
    active port: <analog-input-linein>
```
- Using this information about the internal name of the port, change it with :
```
pacmd set-sink-port 0 analog-output-lineout
```
If you have multiple cards, try changing the 0 to a 1.

- If this works, you can put:
```
set-sink-port 0 analog-output-lineout
```
in your /etc/pulse/default.pa file to have it across reboots.
