










## Minimal Debian Xfce performance tuning Guide for benchmark




                                              

This tutorial guides you through optimizing and install a Debian 12 XFCE system for testing purpose.
It covers system installation, essential utilities, Overclocking and CPU tuning.


**WARNING:** This is intended for experienced Linux users who want to test performance and understand system modifications. This guide focuses on performance, potentially at the cost of system stability and **SECURITY**. Proceed with extreme caution and at your own risk.



**Prerequisites:**



*   Basic knowledge of Linux command-line operations.

*   Understanding of system configuration files.

*   Willingness to accept potential risks associated with advanced system modifications, including **SECURITY VULNERABILITIES** and hardware damage.






## Step 1: Base System Installation and Configuration





### 1.1 Install Debian 12 Netinst

- Download lastest Netinst Debian 12 :

https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-12.9.0-amd64-netinst.iso


- Create usb bootable :

*   From linux : https://gist.github.com/araujo88/6943d4e789c890d9749d19f73bc8c596
*   From Windows : https://devtutorial.io/how-to-create-bootable-usb-installer-for-debian-12-p3123.html


Only select "STANDARD SYSTEM UTILITIES". This ensures a minimal installation with fewer unnecessary packages.


### 1.2 Add user to sudoers

- Install `sudo`:
```
su -
apt install sudo
exit
```


### 1.3 Allow members of group sudo to execute any command

WARNING: This step significantly reduces system SECURITY. Understand the implications before proceeding. It allows any user in the sudo group to execute any command without a password. This is NOT RECOMMENDED for production systems.

```
su -
nano /etc/sudoers
```

- Add and replace with (don't forget to change the username):
```
your_user_name ALL=(ALL:ALL) NOPASSWD: ALL # Replace your_user_name with your username. 
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

*   Close with `Ctrl+x` then `Y` and `Enter`

`NOPASSWD:`  This means no password will be required when using sudo to run the following command.

- Reboot

  

### 1.4 Install Xfce4

- After the base system installation, log in as root and install XFCE:
```
su -
apt install xfce4
```

- Reboot


- Set Terminal
```
sudo apt install mousepad xfce4-terminal -y
```
- Go to Settings > Default Applications > Utilities

*   Set "Terminal Emulator" to "Xfce Terminal".



### 1.3 Configure apt sources

- Edit the apt sources list to include `contrib`, `non-free`, and `non-free-firmware` repositories:
```
sudo nano /etc/apt/sources.list
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




### 1.6 Add rights

- Add your user to the `video` and `audio` groups:
```
sudo usermod -aG video $USER
sudo usermod -aG audio $USER
```



  

## Step 2: Essential Utilities and Desktop Configuration





### 2.1 Install Essential Utilities
```
sudo apt install acpid dbus-x11 accountsservice apt-transport-https ca-certificates software-properties-common git -y

```


> ```acpid```:                        _Handles ACPI events (e.g., lid close, power button)._ 

> ```dbus-x11```:                     _Essential inter-process communication for X11 applications._ 

> ```accountsservice```:              _Manages user accounts and login sessions._ 

> ```apt-transport-https```:          _Enables APT to download packages over HTTPS for secure updates._ 

> ```ca-certificates```:              _Provides root certificates for verifying the authenticity of SSL/TLS connections._ 

> ```software-properties-common```:   _Tools for managing software repositories (PPAs)._ 




### 2.2 Disable Apparmor

WARNING: Disabling AppArmor reduces system SECURITY. Understand the implications before proceeding. Consider configuring AppArmor profiles instead of disabling it entirely.

```
sudo systemctl stop apparmor
sudo systemctl disable apparmor
sudo aa-teardown
```



### 2.3 Install and Configure Firefox ESR
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


### 3.2 disable screen blanking and DPMS


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
*   Set Command: Clic on File icon and find "/home/your_user_name/xset_noblank.sh"
*   Uncheck "screen-locker"

- Reboot

- Check if it work:
```
sudo xset -q
```

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


- Enable the service
```
sudo sysctl -p /etc/sysctl.d/50-coredump.conf
```


### 3.5 Configure CPU Governor and Frequencies

- Note: It's generally recommended to use /etc/default/cpufrequtils for configuration if it exists.  This tutorial uses /etc/init.d/cpufrequtils for consistency with the original, but check for the preferred file first.
```
sudo nano /etc/init.d/cpufrequtils
```

- Find and modify these lines (don't forget to change the ```XXXXXXX``` value):
```
ENABLE="true"
GOVERNOR="ondemand"
MAX_SPEED="XXXXXXX"  # Replace with your CPU's maximum speed (use cpufreq-info to find it)
MIN_SPEED="XXXXXXX"  # Replace with approximately half of your CPU's maximum speed
```

- Use cpufreq-info in the terminal to determine your CPU's maximum speed.
  
- Recommendation:
Even with C-states disabled, the ondemand governor might still be a better choice for performance due to its ability to manage Turbo Boost and frequency scaling within the available range.  However, the best way to know for sure is to benchmark.

- Check Temperatures and CPU Frequencies
```
sudo sensors
sudo cpufreq-info
```


### 3.6 Sysfs Tuning
```
sudo nano /etc/sysfs.conf
```

- Add the following lines:
```
devices/system/cpu/cpufreq/ondemand/up_threshold = 99
# Percentage CPU utilization before scaling up frequency. Higher values mean more responsiveness.
devices/system/cpu/cpufreq/ondemand/sampling_down_factor = 6
# How aggressively the CPU scales down frequency. Higher values mean less aggressive scaling down.
devices/system/cpu/cpufreq/ondemand/sampling_rate = 20000000
# How often the CPU usage is checked. Higher values mean more frequent checks (more responsive).
```

### 3.7 Sysctl Tuning

```
sudo nano /etc/sysctl.conf
```

- Add the following lines:
```
vm.swappiness = 10
# Represents the kernel's preference (or avoidance) of swap space.(better performance if enough RAM).
vm.dirty_background_ratio = 5
# Percentage of "dirty" memory before background write to disk starts.
vm.dirty_ratio = 10
# # Percentage of "dirty" memory before writes to disk are forced.
vm.vfs_cache_pressure = 50
# How aggressively the kernel reclaims memory from the VFS cache. Lower values mean less aggressive reclaiming.
```

### 3.7 Grub Configuration

**WARNING**: Disabling security mitigations (mitigations=off kernel.randomize_va_space=0) is NOT RECOMMENDED and significantly reduces system SECURITY. Understand the risks before proceeding.
```
sudo nano /etc/default/grub
```

- Modify the GRUB_CMDLINE_LINUX_DEFAULT line to:
```
GRUB_CMDLINE_LINUX_DEFAULT="intel_pstate=passive intel_idle.max_cstate=0 idle=poll cpufreq.default_governor=ondemand nosmt=force pcie_aspm=off mitigations=off kernel.randomize_va_space=0 ipv6.disable=1 zswap.enabled=0 nvidia-drm.modeset=1"
```
- Parameters :

> ```intel_pstate=passive```          _Use the older intel_pstate driver in passive mode._

> ```intel_idle.max_cstate=0```       _Disable all CPU C-states (low-power states)._

> ```idle=poll```                     _Use polling for idle instead of interrupts (can increase power consumption but reduce latency)._

> ```nosmt=force```                   _Disable Symmetric Multi-Threading (Hyperthreading may hurt the performance, do benchmarks)._

> ```pcie_aspm=off```                 _Disable PCIe Active State Power Management (can improve latency but increase power consumption)._

> ```mitigations=off```               _Disable kernel mitigations for CPU vulnerabilities (increases risk but might improve performance)._

> ```kernel.randomize_va_space=0```   _Disable address space layout randomization (security risk, might slightly improve performance)._

> ```ipv6.disable=1```                _Disable IPv6 networking._

> ```zswap.enabled=0```              _Disable kernel feature that provides a compressed RAM cache for swap pages (better performance if enough RAM)._

> ``` nvidia-drm.modeset=1```         _enabling DRM (Direct Rendering Manager) kernel mode setting is required to allow for Xorg#Rootless Xorg._

> ```cpufreq.default_governor=ondemand```    _Set governor CPU to "ondemand"_  



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

**WARNING**: ```allowed_users=anybody```: This setting allows any user on the system to run X server applications (graphical programs) as root. This means if any user account is compromised, the attacker gains full root access to your graphical environment and, consequently, your entire system.
```needs_root_rights=yes```: This explicitly grants root privileges to X server applications. Combined with allowed_users=anybody, a malicious program run by any user would have full root access.


    


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


```
sudo nano /etc/sudoers
```

- Add the following line:

```
%cuda ALL=(ALL) NOPASSWD: /usr/bin/nvidia-smi
```
**WARNING**: The modification was overly permissive. 

### 4.4 Nvidia


#### 4.4.1 Nvidia Xorg conf


- Configuring the 10-nvidia-drm-outputclass :
```
sudo nano /usr/share/X11/xorg.conf.d/nvidia-drm-outputclass.conf
```
- Important: Verify that the ModulePath entries exist for your system.  Incorrect paths can prevent X from starting.
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

# "Coolbits" "28"  Enable Overclocking and FanControl
```




#### 4.4.2 Modprobe conf
```
sudo nano /etc/modprobe.d/nvidia-options.conf
```

- Add the following lines:

```
options nvidia NVreg_UsePageAttributeTable=1
# Can improve performance, especially for graphics-intensive applications, by optimizing memory access patterns.
options nvidia NVreg_EnableMSI=1
# Reduces CPU overhead associated with handling interrupts from the graphics card, potentially improving system responsiveness.
options nvidia NVreg_EnableStreamMemOPs=1
# Can improve performance in applications that involve significant data transfer between the CPU and GPU.
options nvidia NVreg_RegistryDwords="PerfLevelSrc=0x2222"
# Performance level
options nvidia NVreg_RegistryDwords="OverrideMaxPerf=0x1"
# Max performance (caution: heat/power)
options nvidia-drm modeset=1
# Enable kernel mode setting (modeset)
options nvidia-current NVreg_RestrictProfilingToAdminUsers=0
# Allow non-administrative users to use NVIDIA profiling tools.  This can be useful for debugging and performance analysis but might pose a security risk in some environments
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

**WARNING**: Overclocking can damage your hardware.  
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

**EXTREME CAUTION**: Increase offset values incrementally and RUN BENCHMARK AFTER EACH CHANGES.  Start with small increases (e.g., +10 MHz for graphics clock, +100 MHz for memory clock).


- Set maximum performance:
```
sudo nvidia-settings -a '[gpu:0]/GpuPowerMizerMode=1'
```

- Set fan control:
```
sudo nvidia-settings -a '[gpu:0]/GPUFanControlState=1'
sudo nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=XXX'   # XXX% fan speed
```

- Set power limit:
```
sudo nvidia-smi --power-limit=XXX   # Replace XXX with your GPU's maximum power limit, found using sudo nvidia-smi -q
```

- Graphics overclocking (replace XXX with the desired offset, starting with small values like 10):
```
sudo nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffsetAllPerformanceLevels=XXX'   # Where XXX is the value added to the maximum Memory clocks (exp:120 = +120MHZ)
```

- Memory overclocking (replace XXXX with the desired offset, starting with small values like 100):
```
sudo nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=XXXX'  # Where XXXX is the value added to the maximum Memory clocks (exp:1000 = +1000MHz) 
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

sudo nvidia-smi --power-limit=XXX    # Where XXX is value for power-limit

##Set maximum performance mode

sudo nvidia-settings -a '[gpu:0]/GpuPowerMizerMode=1'

##Set fan control

sudo nvidia-settings -a '[gpu:0]/GPUFanControlState=1'
sudo nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=XXX' # Where XXX is % of Fan Speed


##Graphics overclocking

sudo nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffsetAllPerformanceLevels=XXX'    # Where XXX is the value added to the maximum graphics clocks

##Memory overclocking

sudo nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=XXXX'    # Where XXXX is the value added to the maximum Memory clocks

```

- Make script executable:
```
sudo chmod +x /home/$USER/GpuTweaks.sh
```

- Add to autostart (Applications > Settings > Session and Startup > Application Autostart)

*   Click "+ Add".
*   Set Name: GpuTweaks
*   Set Command: Clic on File icon and find "/home/your_user_name/GpuTweaks.sh"
  

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

barrier=0:  **WARNING** Write barriers ensure data integrity by
 forcing writes to be completed in a specific order. Disabling them can 
significantly improve performance, especially on SSDs, but if your system crashes or loses power during a write operation,
 data corruption can occur.  Use with caution!  If you're unsure, leave barrier=1 (the default).


- Add options
```
UUID=YOUR_ROOT_PARTITION_UUID / ext4 noatime,barrier=0,errors=remount-ro 0 1
```


- Reload systemctl
 
```
sudo systemctl daemon-reload
```


### 5.2 Disable Journaling on ext4

**EXTREME CAUTION**: Disabling journaling significantly increases the risk of data loss in case of a power failure or system crash.  This is NOT RECOMMENDED for most users.  The following steps involve creating a backup of your system, which is essential before disabling journaling.


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

**WARNING**: This can lead to data loss!  Ensure you have a complete backup before proceeding.
```
lsblk    # Identify your root partition (e.g., /dev/nvme0n1p2)

sudo tune2fs -O ^has_journal /dev/nvme0nXXX    # Replace /dev/nvme0nXXX with your actual root ( / ) partition (you can find it with : " lsblk ")
```


### 5.3 Autologin Lightdm

**WARNING**: Enabling autologin is a security risk.  Anyone with physical access to your computer will be automatically logged in.
```
sudo nano /etc/lightdm/lightdm.conf
```

- Find or add the following lines within the appropriate sections (don't forget to change the username):

```
[LightDM]
logind-check-graphical=true

[Seat:*]
autologin-user=your_user_name # Where "your_user_name" is your user name.
autologin-user-timeout=0
```


### 5.4 PulseAudio Configuration
```
sudo nano /etc/pulse/daemon.conf
```

- Uncomment (remove the leading ;) and modify the following lines:

```
flat-volumes = no
avoid-resampling = true
```


#### 5.4.1 Disable idle and tsched
```
sudo nano /etc/pulse/system.pa
```

- Add and Comment this lines:
```
load-module module-udev-detect tsched=0
# disables the timer-based scheduling feature of the module. (This can improve performance, but might introduce latency)
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

Betterfox github - https://github.com/yokoffing/Betterfox/tree/esr128

nvidia-patch github - https://github.com/keylase/nvidia-patch.git

Unigine Heaven benchmark - https://benchmark.unigine.com/heaven

Rescuezilla - https://github.com/rescuezilla/rescuezilla/releases/download/2.5.1/rescuezilla-2.5.1-64bit.noble.iso




## Troubleshootings



### Enable Acceleration GPU Firefox


- Open Firefox Profile Folder: Go to about:profiles in Firefox, find your active profile, and click "Open Folder."
  
- Create user.js: In the profile folder, create a new text file named user.js.

- Add the following lines :
```
// Enable WebRender and related features for improved graphics performance.
user_pref("gfx.webrender.compositor.force-enabled", true);
user_pref("gfx.x11-egl.force-enabled", true);
user_pref("gfx.canvas.accelerated", true);
user_pref("media.gpu-process-decoder", true);
user_pref("gfx.layers.acceleration.force-enabled", true);
user_pref("gfx.skia.force_enabled", true);
user_pref("gfx.webrender.all", true);
user_pref("gfx.webrender.force-enabled", true);
user_pref("media.hardware-video-decoding.force-enabled", true);
// Media:
user_pref("media.ffvpx.enabled", false); // Disable FFvpx codec
user_pref("media.av1.enabled", false);
```


### Firefox Performance


```
// Disable animations and some timing APIs to make the browser feel faster.
user_pref("dom.animations-api.animations.enabled", true); //Keep animations enabled, but other timing disabled.
user_pref("image.animation_mode", "none"); // Stop images from moving (like GIFs) to save CPU.
user_pref("dom.enable_event_timing", false); // Turns off a feature that tracks how long website actions take (less work for your computer).
user_pref("dom.enable_resource_timing", false); // Stops tracking how long it takes to load website parts (saves resources).
user_pref("dom.enable_user_timing", false); // Stops tracking custom website performance (less background work).

// Network optimizations: Adjust settings to load websites faster.
user_pref("network.buffer.cache.size", 262144); // Makes the browser hold more website data in memory for faster loading.
user_pref("network.http.fast-fallback-to-IPv4", true); // Helps websites load faster if they have older internet connections.
user_pref("network.http.max-persistent-connections-per-proxy", 15); // Lets the browser keep more connections open for faster loading.
user_pref("network.http.max-persistent-connections-per-server", 15); //Allows more connections to one server for faster loading.
user_pref("network.http.max-connections", 900); // Increases how many connections the browser can make at once.
user_pref("network.http.max-urgent-start-excessive-connections-per-host", 5); // Allows a few more connections for very important website parts.
user_pref("network.http.pacing.requests.enabled", false); // Turns off a feature that slows down website loading (for faster speeds).
user_pref("network.dnsCacheExpiration", 36000); // Makes the browser remember website addresses for 10 hours (faster loading).
user_pref("network.ssl_tokens_cache_capacity", 10240); // Makes the browser remember secure website information longer.
user_pref("network.dns.disablePrefetchFromHTTPS", true); // Stops the browser from guessing website addresses from secure sites (less work).
user_pref("network.predictor.enabled", false); // Turns off a feature that tries to guess which websites you'll visit (saves resources).
user_pref("browser.cache.disk.enable", true); // Allows the browser to store website data on your computer (faster loading).
user_pref("browser.cache.memory.capacity", 131072); // Sets how much memory the browser uses to store website data.
user_pref("media.memory_cache_max_size", 262144); // Sets how much memory the browser uses to store videos and audio.
user_pref("browser.cache.jsbc_compression_level", 3); // Compresses website code more efficiently (saves space).
user_pref("media.cache_readahead_limit", 7200); // Makes the browser load videos and audio further ahead.
user_pref("media.cache_resume_threshold", 3600); // Lets you resume videos and audio from further back.
user_pref("content.notify.interval", 20000); // Makes the browser check for updates on websites less often (saves resources).
user_pref("network.dns.disablePrefetch", true); // Stops the browser from guessing website addresses (less work).
user_pref("network.predictor.enable-prefetch", false); // Turns off another website guessing feature (saves resources).
user_pref("network.prefetch-next", false); // Stops the browser from loading the next page before you click (saves resources).
user_pref("browser.urlbar.speculativeConnect.enabled", false); // Stops the browser from connecting to websites before you type them (saves resources).
user_pref("browser.sessionstore.interval", 60000); // Saves your tabs and windows less often (saves resources).
user_pref("browser.sessionstore.resume_from_crash", false); // Stops the browser from trying to restore your session after a crash (saves resources).
user_pref("browser.newtab.preload", false); // Stops the browser from loading the new tab page early (saves resources).
user_pref("browser.newtabpage.preload", false); //Stops the browser from loading the new tab page early (saves resources).
user_pref("browser.startup.page", 0); //Sets the browser to start with a blank page.
user_pref("browser.newtabpage.enabled", false); //Disables the new tab page.
user_pref("fission.autostart", false); // Turns off a feature that separates websites (less security, but faster).
user_pref("dom.ipc.processCount", 1); // Limits how many website processes the browser uses (saves resources).
user_pref("dom.ipc.processCount.webIsolated", 1); // Limits how many isolated website processes the browser uses.
user_pref("dom.ipc.processCount.webCOOP+COEP", 0); // Disables processes for special websites (saves resources).
user_pref("dom.ipc.processPrelaunch.fission.number", 0); // Stops the browser from starting website processes early (saves resources).
user_pref("dom.ipc.processCount.inference", 0); // Disables processes that try to guess what you'll do (saves resources).
user_pref("widget.dmabuf.force-enabled", true); // Makes graphics faster (if your computer supports it).
```

####  Firefox Privacy and Security 
```
user_pref("privacy.globalprivacycontrol.enabled", true); // Tells websites you don't want to be tracked.
user_pref("security.ssl.treat_unsafe_negotiation_as_broken", true); // Makes the browser more strict about secure connections.
user_pref("browser.xul.error_pages.expert_bad_cert", true); // Shows you more details about website security problems.
user_pref("security.tls.enable_0rtt_data", false); // Turns off a feature that can be risky (more secure).
user_pref("browser.privatebrowsing.forceMediaMemoryCache", true); // Makes the browser store videos and audio in memory when you're in private mode.
user_pref("privacy.history.custom", true); // Lets you choose what the browser remembers.
user_pref("dom.security.https_first", true); // Always tries to load websites securely.
user_pref("dom.security.https_first_schemeless", true); // Always tries to load websites securely, even without https://.
user_pref("signon.formlessCapture.enabled", false); // Stops the browser from saving login information from some websites.
user_pref("signon.privateBrowsingCapture.enabled", false); // Stops the browser from saving login information in private mode.
user_pref("security.mixed_content.block_display_content", true); // Stops websites from showing unsafe parts (like pictures).
user_pref("extensions.postDownloadThirdPartyPrompt", false); // Stops extra messages after you download add-ons.
user_pref("network.http.referer.XOriginTrimmingPolicy", 2); // Tells websites less about where you came from.
user_pref("privacy.userContext.ui.enabled", true); // Lets you use containers (like separate profiles) for privacy.
user_pref("media.peerconnection.ice.proxy_only_if_behind_proxy", true); // Makes video calls more private.
user_pref("media.peerconnection.ice.default_address_only", true); // Makes video calls more private.
user_pref("browser.safebrowsing.downloads.remote.enabled", false); // Stops the browser from checking downloaded files with Google.
user_pref("permissions.default.desktop-notification", 2); // Blocks website notifications.
user_pref("permissions.default.geo", 2); // Blocks websites from knowing your location.
user_pref("permissions.manager.defaultsUrl", ""); // Clears default website permissions.
user_pref("webchannel.allowObject.urlWhitelist", ""); // Stops websites from using a feature that can be risky.
user_pref("datareporting.policy.dataSubmissionEnabled", false); // Stops the browser from sending data to Mozilla.
user_pref("datareporting.healthreport.uploadEnabled", false); // Stops the browser from sending health reports to Mozilla.
user_pref("toolkit.telemetry.unified", false); // Stops the browser from sending data to Mozilla.
user_pref("toolkit.telemetry.enabled", false); // Stops the browser from sending data to Mozilla.
user_pref("toolkit.telemetry.server", "data:,
```






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


### Disable Idle in Intel sound card :


```
sudo nano /etc/sysfs.conf
```

- Add the following lines :
```
module/snd_hda_intel/parameters/power_save = 0
module/snd_hda_intel/parameters/power_save_controller = N
```

### Static Driver Loading


Use with Caution, This will prevent pulse audio from needing to probe the hardware


```
sudo aplay -l
```
Note "card" and "device" numbers (if you see card 0 and device 0, that corresponds to hw:0,0)
```
sudo nano /etc/pulse/default.pa
```

- Replace the following lines :
```
load-module module-alsa-sink device=hw:0,0   # Replace "hw:0,0" with your numbers card
load-module module-alsa-source device=hw:0,0   # Replace "hw:0,0" with your numbers card
```


### Default Sink and Source


- List Sources (Input Devices):
```
pactl list sources short
```


- This will list available audio input sources (microphones, line-in, etc.).  Example output:
```
0   alsa_input.pci-0000_00_1b.0.analog-stereo
1   alsa_input.usb-Generic_USB_Audio-00.analog-stereo
```

- List Sinks (Input Devices):
```
pactl list sinks short
```


- This will list available audio output sources (headphones, line-out, etc.).  Example output:
```
0   alsa_output.pci-0000_00_1b.0.analog-stereo
```


- Set the Default Sink and Source : 
```
sudo nano ~/.config/pulse/default.pa  # (User-Specific):  This is generally the recommended approach for user-specific settings.
```

- Replace the following lines :
```
set-default-sink <sink_name_or_index>
set-default-source <source_name_or_index>
```


Replace <sink_name_or_index> with the name or index you got from pactl list sinks short. For example : 
```
set-default-sink alsa_input.pci-0000_00_1b.0.analog-stereo
set-default-source alsa_output.pci-0000_00_1b.0.analog-stereo
```





