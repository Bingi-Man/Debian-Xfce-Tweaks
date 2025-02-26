










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


Only select "STANDARD SYSTEM UTILITIES". This ensures a minimal installation with fewer unnecessary packages.


### 1.2 Add user to sudoers

- Install `sudo`:
```
su -
apt install sudo
exit
```


### 1.3 Allow members of group sudo to execute any command

**WARNING:** This step significantly reduces system security.  Understand the implications before proceeding.  It allows any user in the `sudo` group to execute *any* command without a password.
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

**WARNING:** Disabling AppArmor reduces system security.  Understand the implications before proceeding.  Consider configuring AppArmor profiles instead of disabling it entirely.
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

**WARNING**: Disabling security mitigations (mitigations=off kernel.randomize_va_space=0) is not recommended and significantly reduces system security.  Understand the risks before proceeding.
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

EXTREME CAUTION: Increase offset values incrementally and RUN BENCHMARK AFTER EACH CHANGES.  Start with small increases (e.g., +10 MHz for graphics clock, +100 MHz for memory clock).


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
default-sample-rate = 48000
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

### Enable Acceleration GPU 

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


### Tweaks for offloading cpu in Firefox

```
// Disable animations and some timing APIs to reduce potential fingerprinting surface and improve performance.
user_pref("dom.animations-api.animations.enabled", true);
user_pref("image.animation_mode", "none"); // Don't animate images
user_pref("dom.enable_event_timing", false);
user_pref("dom.enable_resource_timing", false);
user_pref("dom.enable_user_timing", false);

// Network optimizations: adjust cache size, and tune connection limits.
user_pref("network.buffer.cache.size", 262144); // Increased buffer cache size
user_pref("network.http.fast-fallback-to-IPv4", true);

// Session and Tab Management:  Disable undoing closed tabs, disable auto-updates, and disable crash reporter.
user_pref("browser.sessionstore.max_tabs_undo", 0);
user_pref("extensions.update.autoUpdateDefault", false);
user_pref("extensions.update.enabled", false);
user_pref("toolkit.crashreporter.enabled", false);

// Accessibility: Force accessibility services to be disabled.
user_pref("accessibility.force_disabled", 1);

// URL Bar and Search: Disable search suggestions and history suggestions in the URL bar.
user_pref("browser.urlbar.suggest.searches", false);
user_pref("browser.urlbar.suggest.history", false);

// Media: Disable Encrypted Media Extensions (EME) for DRM, but enable OpenH264.  Clear GMP manager URL.
user_pref("media.eme.enabled", false);
user_pref("media.gmp-gmpopenh264.enabled", true);
user_pref("media.gmp-manager.url", "");

// Reader Mode: Disable automatic reader mode parsing.
user_pref("reader.parse-on-load.enabled", false);

// Default Browser Check: Disable default browser check.
user_pref("browser.shell.checkDefaultBrowser", false);

// Disable Activity Stream in the library.
user_pref("browser.library.activity-stream.enabled", false);

// Enable Global Privacy Control.
user_pref("privacy.globalprivacycontrol.enabled", true);

// Image Decoding: Adjust the amount of memory used for decoding images at a time.
user_pref("image.mem.decode_bytes_at_a_time", 32768);

// Increase the number of persistent connections per proxy.
user_pref("network.http.max-persistent-connections-per-proxy", 15);

// Enable Masonry layout in CSS Grid.
user_pref("layout.css.grid-template-masonry-value.enabled", true);

// Enable web task scheduling and the Sanitizer API.
user_pref("dom.enable_web_task_scheduling", true);
user_pref("dom.security.sanitizer.enabled", true);

// Security: Treat unsafe negotiation as broken, enable strict transport security, and disable 0-RTT data.
user_pref("security.ssl.treat_unsafe_negotiation_as_broken", true);
user_pref("browser.xul.error_pages.expert_bad_cert", true); // Show advanced information on certificate errors
user_pref("security.tls.enable_0rtt_data", false);

// Caching: Force media caching in memory during private browsing.
user_pref("browser.privatebrowsing.forceMediaMemoryCache", true);

// Session Restore: Increase the interval between session saves.
user_pref("browser.sessionstore.interval", 60000); // Save session every 60 seconds (instead of the default 15)

// Privacy: Enable custom history settings.
user_pref("privacy.history.custom", true);

// UI Tweaks: Separate private search defaults, refresh engine alias, disable URL bar group labels.
user_pref("browser.search.separatePrivateDefault.ui.enabled", true);
user_pref("browser.urlbar.update2.engineAliasRefresh", true);
user_pref("browser.urlbar.groupLabels.enabled", false);

// HTTPS-First Mode: Always try to connect using HTTPS.
user_pref("dom.security.https_first", true);
user_pref("dom.security.https_first_schemeless", true);

// Disable formless login capture and private browsing capture.
user_pref("signon.formlessCapture.enabled", false);
user_pref("signon.privateBrowsingCapture.enabled", false);

// Network Authentication: Allow HTTP authentication for subresources.
user_pref("network.auth.subresource-http-auth-allow", 1);

// Disable truncating user pastes in the editor.
user_pref("editor.truncate_user_pastes", false);

// Block mixed content (non-HTTPS resources on HTTPS pages).
user_pref("security.mixed_content.block_display_content", true);

// Disable scripting in PDF.js.
user_pref("pdfjs.enableScripting", false);

// Disable third-party prompts after extension downloads.
user_pref("extensions.postDownloadThirdPartyPrompt", false);

// Referrer Policy:  Reduce referrer information sent to different origins.
user_pref("network.http.referer.XOriginTrimmingPolicy", 2);

// Enable user context (containers) UI.
user_pref("privacy.userContext.ui.enabled", true);

// WebRTC: Only use ICE proxy if behind a proxy, and only use default addresses.
user_pref("media.peerconnection.ice.proxy_only_if_behind_proxy", true);
user_pref("media.peerconnection.ice.default_address_only", true);

// Disable remote Safe Browsing lookups for downloads.
user_pref("browser.safebrowsing.downloads.remote.enabled", false);

// Permissions: Block desktop notifications and geolocation by default.  Clear default permissions URL.
user_pref("permissions.default.desktop-notification", 2); // 2 = Block
user_pref("permissions.default.geo", 2); // 2 = Block
user_pref("permissions.manager.defaultsUrl", "");

// Disable whitelisting for web channels.
user_pref("webchannel.allowObject.urlWhitelist", "");

// Telemetry and Data Reporting: Disable all telemetry and data submission.
user_pref("datareporting.policy.dataSubmissionEnabled", false);
user_pref("datareporting.healthreport.uploadEnabled", false);
user_pref("toolkit.telemetry.unified", false);
user_pref("toolkit.telemetry.enabled", false);
user_pref("toolkit.telemetry.server", "data:,");
user_pref("toolkit.telemetry.archive.enabled", false);
user_pref("toolkit.telemetry.newProfilePing.enabled", false);
user_pref("toolkit.telemetry.shutdownPingSender.enabled", false);
user_pref("toolkit.telemetry.updatePing.enabled", false);
user_pref("toolkit.telemetry.bhrPing.enabled", false);
user_pref("toolkit.telemetry.firstShutdownPing.enabled", false);
user_pref("toolkit.coverage.opt-out", true);
user_pref("toolkit.coverage.endpoint.base", "");
user_pref("browser.newtabpage.activity-stream.feeds.telemetry", false);
user_pref("browser.newtabpage.activity-stream.telemetry", false);

// Studies and Experiments: Disable Shield studies, Normandy, and related features.
user_pref("app.shield.optoutstudies.enabled", false);
user_pref("app.normandy.enabled", false);
user_pref("app.normandy.api_url", "");

// Crash Reporting: Disable Breakpad crash reporting and automatic submission.
user_pref("breakpad.reportURL", "");
user_pref("browser.tabs.crashReporting.sendReport", false);
user_pref("browser.crashReports.unsubmittedCheck.autoSubmit2", false);

// Captive Portal Detection: Disable captive portal detection.
user_pref("captivedetect.canonicalURL", "");
user_pref("network.captive-portal-service.enabled", false);

// Network Connectivity Service: Disable connectivity service.
user_pref("network.connectivity-service.enabled", false);
//Disable Private Attribution Token
user_pref("dom.private-attribution.submission.enabled", false);

// Disable VPN promotion URL.
user_pref("browser.privatebrowsing.vpnpromourl", "");

// Add-ons and Recommendations: Disable add-on recommendations and discovery.
user_pref("extensions.getAddons.showPane", false); // Hide the "Get Add-ons" panel
user_pref("extensions.htmlaboutaddons.recommendations.enabled", false);
user_pref("browser.discovery.enabled", false);
user_pref("browser.newtabpage.activity-stream.asrouter.userprefs.cfr.addons", false);
user_pref("browser.newtabpage.activity-stream.asrouter.userprefs.cfr.features", false);

// UI: Disable "More from Mozilla" in preferences, tab manager, about:config warning, and about:welcome.
user_pref("browser.preferences.moreFromMozilla", false);
user_pref("browser.tabs.tabmanager.enabled", false);
user_pref("browser.aboutConfig.showWarning", false);
user_pref("browser.aboutwelcome.enabled", false);

// Cookie Banner Handling: Enable cookie banner handling service (but don't automatically reject).
user_pref("cookiebanners.service.mode", 1);
user_pref("cookiebanners.service.mode.privateBrowsing", 1);

// Full Screen API: Disable full-screen transitions and warnings.
user_pref("full-screen-api.transition-duration.enter", "0 0");
user_pref("full-screen-api.transition-duration.leave", "0 0");
user_pref("full-screen-api.warning.delay", -1);
user_pref("full-screen-api.warning.timeout", 0);

// URL Bar: Enable calculator and unit conversion suggestions, but disable trending searches.
user_pref("browser.urlbar.suggest.calculator", true);
user_pref("browser.urlbar.unitConversion.enabled", true);
user_pref("browser.urlbar.trending.featureGate", false);

// New Tab Page: Disable top sites, top stories, Pocket, and snippets.
user_pref("browser.newtabpage.activity-stream.feeds.topsites", false);
user_pref("browser.newtabpage.activity-stream.feeds.section.topstories", false);
user_pref("extensions.pocket.enabled", false); // Disable Pocket integration
user_pref("browser.newtabpage.activity-stream.feeds.snippets", false);
user_pref("browser.newtabpage.activity-stream.asrouter.providers.snippets", "");

// Downloads: Always ask before handling new types, don't add to recent documents, and open PDFs inline.
user_pref("browser.download.always_ask_before_handling_new_types", true);
user_pref("browser.download.manager.addToRecentDocs", false);
user_pref("browser.download.open_pdf_attachments_inline", true);

// Bookmarks: Don't close the bookmarks menu when opening a bookmark in a new tab.
user_pref("browser.bookmarks.openInTabClosesMenu", false);

// Context Menu: Show "View Image Info" in the context menu.
user_pref("browser.menu.showViewImageInfo", true);

// Find Bar: Highlight all matches in the find bar.
user_pref("findbar.highlightAll", true);

// Word Selection: Don't eat spaces when selecting words.
user_pref("layout.word_select.eat_space_to_next_word", false);

// Search Suggestions: Disable search suggestions in general and specifically QuickSuggest.
user_pref("browser.search.suggest.enabled", false);
user_pref("browser.urlbar.quicksuggest.enabled", false);
user_pref("browser.urlbar.suggest.quicksuggest.sponsored", false);
user_pref("browser.urlbar.suggest.quicksuggest.nonsponsored", false);

// Security: Enable CRLite for certificate revocation checking and remote settings for CRLite filters.
user_pref("security.pki.crlite_mode", 2);
user_pref("security.remote_settings.crlite_filters.enabled", true);

// Add-ons Discovery: Disable add-ons discovery.
user_pref("extensions.htmlaboutaddons.discover.enabled", false);

// Privacy: Enable user context (containers) and new tab segregation.
user_pref("privacy.userContext.enabled", true);
user_pref("privacy.usercontext.about_newtab_segregation.enabled", true);

// Query Stripping: Enable query stripping to remove tracking parameters from URLs.
user_pref("privacy.query_stripping.enabled", true);
user_pref("privacy.query_stripping.enabled.pbmode", true);

// Screenshots: Disable uploading screenshots.
user_pref("extensions.screenshots.upload-disabled", true);

// Page Thumbnails: Disable capturing page thumbnails.
user_pref("browser.pagethumbnails.capturing_disabled", true);

// WebRTC: Disable WebRTC entirely, including peer connection, video, identity, and navigator.
user_pref("media.peerconnection.enabled", false);
user_pref("media.peerconnection.video.enabled", false);
user_pref("media.peerconnection.identity.enabled", false);
user_pref("media.navigator.enabled", false);
user_pref("media.navigator.video.enabled", false);

// New Tab Page: Disable search, sponsored content, snippets, and highlights on the new tab page.
user_pref("browser.newtabpage.activity-stream.showSearch", false);
user_pref("browser.newtabpage.activity-stream.showSponsored", false);
user_pref("browser.newtabpage.activity-stream.showSponsoredTopSites", false);
user_pref("browser.newtabpage.activity-stream.section.highlights.includeBookmarks", false);
user_pref("browser.newtabpage.activity-stream.section.highlights.includeDownloads", false);
user_pref("browser.newtabpage.activity-stream.section.highlights.includeVisited", false);
user_pref("browser.newtabpage.activity-stream.section.highlights.includePocket", false);

// Onboarding: Disable onboarding.
user_pref("browser.onboarding.enabled", false);

// New Tab Preload: Disable preloading of the new tab page.
user_pref("browser.newtab.preload", false);

// Startup: Set the homepage to a blank page and disable the startup page.
user_pref("browser.startup.homepage", "about:blank");
user_pref("browser.newtabpage.enabled", false);
user_pref("browser.startup.page", 0);

// WebSpeech: Disable WebSpeech synthesis.
user_pref("media.webspeech.synth.enabled", false);

// Media: Enable RDD-FFmpeg and disable FFvpx.
user_pref("media.rdd-ffmpeg.enabled", true);
user_pref("media.ffvpx.enabled", false);

// Content Notify Interval: Increase the interval for content notifications.
user_pref("content.notify.interval", 20000); // 20 seconds

// Prefetching: Disable DNS prefetching, link prefetching, and speculative connections.
user_pref("network.dns.disablePrefetch", true);
user_pref("network.predictor.enable-prefetch", false);
user_pref("network.prefetch-next", false);
user_pref("browser.urlbar.speculativeConnect.enabled", false);

// Session Restore: Disable resuming from crashes.
user_pref("browser.sessionstore.resume_from_crash", false);

// JavaScript Bytecode Cache: Adjust the compression level.
user_pref("browser.cache.jsbc_compression_level", 3);

// Media Cache: Adjust readahead and resume thresholds.
user_pref("media.cache_readahead_limit", 7200); // seconds
user_pref("media.cache_resume_threshold", 3600); // seconds

// Network Connections: Increase the maximum number of connections and adjust excessive connections per host.
user_pref("network.http.max-connections", 900);
user_pref("network.http.max-urgent-start-excessive-connections-per-host", 5);

// HTTP Pacing: Disable HTTP request pacing.
user_pref("network.http.pacing.requests.enabled", false);

// DNS Cache: Increase DNS cache expiration time.
user_pref("network.dnsCacheExpiration", 36000); // 10 hours

// SSL Token Cache: Increase SSL token cache capacity.
user_pref("network.ssl_tokens_cache_capacity", 10240);

// Disable DNS prefetching from HTTPS.
user_pref("network.dns.disablePrefetchFromHTTPS", true);

// Network Predictor: Disable network predictor.
user_pref("network.predictor.enabled", false);

// Content Blocking: Set content blocking category to strict.
user_pref("browser.contentblocking.category", "strict");

// Cookies: Require SameSite=None cookies to be secure.
user_pref("network.cookie.sameSite.noneRequiresSecure", true);

// URL Bar: Trim HTTPS from the URL bar.
user_pref("browser.urlbar.trimHttps", true);

// Form Fill: Disable form filling.
user_pref("browser.formfill.enable", false);

// Insecure Connection Text: Show warnings for insecure connections.
user_pref("security.insecure_connection_text.enabled", true);
user_pref("security.insecure_connection_text.pbmode.enabled", true);

// Punycode: Show Punycode for internationalized domain names.
user_pref("network.IDN_show_punycode", true);

// Graphics Caching: Adjust cache sizes for accelerated canvases and Skia fonts.
user_pref("gfx.canvas.accelerated.cache-items", 4096);
user_pref("gfx.canvas.accelerated.cache-size", 512);
user_pref("gfx.content.skia-font-cache-size", 20);

// Custom Stylesheets: Enable support for user stylesheets.
user_pref("toolkit.legacyUserProfileCustomizations.stylesheets", true);

// Compact Mode: Show compact mode option in the UI.
user_pref("browser.compactmode.show", true);

// Focus Ring: Customize the focus ring style and width.
user_pref("browser.display.focus_ring_on_anything", true);
user_pref("browser.display.focus_ring_style", 0); // Solid
user_pref("browser.display.focus_ring_width", 0); // No width

// Color Scheme: Prefer dark color scheme.
user_pref("layout.css.prefers-color-scheme.content-override", 2);

// Clipboard Events: Disable clipboard events.
user_pref("dom.event.clipboardevents.enabled", false);

// URL Bar: Do not trim URLs completely.
user_pref("browser.urlbar.trimURLs", false);

// Keyword Search: Enable keyword search.
user_pref("keyword.enabled", true);

// Downloads: Start downloads in the temporary directory.
user_pref("browser.download.start_downloads_in_tmp_dir", true);

// Helper Apps: Delete temporary files on exit.
user_pref("browser.helperApps.deleteTempFileOnExit", true);

// UITour: Disable UITour.
user_pref("browser.uitour.enabled", false);

// DMA-BUF: Force enable DMA-BUF (Direct Memory Access Buffer) for graphics.
user_pref("widget.dmabuf.force-enabled", true);

// OCSP: Disable OCSP (Online Certificate Status Protocol) stapling.
user_pref("security.OCSP.enabled", 0);

// Caching: Enable disk caching, set memory cache capacity, and adjust media memory cache size.
user_pref("browser.cache.disk.enable", true);
user_pref("browser.cache.memory.capacity", 131072);  // 128MB,  -1=default , 262144= 256MB
user_pref("media.memory_cache_max_size", 262144); // 256MB
user_pref("network.http.max-persistent-connections-per-server", 15); // More connections

// Geolocation: Disable geolocation.
user_pref("geo.enable", false);

// Search Region: Set the search region.
user_pref("browser.search.region", "US");

// Process Management and Isolation:
user_pref("fission.autostart", false); // Disable Fission (site isolation)
user_pref("dom.ipc.processCount", 1); // Limit general content processes
user_pref("dom.ipc.processCount.webIsolated", 1); // Limit isolated web content processes
user_pref("dom.ipc.processCount.webCOOP+COEP", 0); // Disable processes for COOP/COEP sites
user_pref("dom.ipc.processPrelaunch.fission.number", 0); // Disable Fission process pre-launching
user_pref("dom.ipc.processCount.inference", 0); // Disable inference processes

// Network and Connectivity:
user_pref("network.captive-portal-service.enabled", false); // Disable captive portal detection
user_pref("browser.search.geoip.url", ""); // Disable GeoIP URL
user_pref("browser.aboutHomeSnippets.updateUrl", ""); // Disable home snippets update URL

// Privacy and Tracking:
user_pref("privacy.trackingprotection.enabled", false); // Disable tracking protection
user_pref("dom.enable_performance_timing", false); // Disable performance timing API
user_pref("dom.enable_event_timing", false); // Disable event timing API
user_pref("dom.enable_resource_timing", false); // Disable resource timing API
user_pref("dom.enable_user_timing", false); // Disable user timing API

// Updates and Features:
user_pref("browser.search.update", false); // Disable search engine updates
user_pref("app.update.enabled", false); // Disable Firefox updates
user_pref("browser.casting.enabled", false); // Disable casting
user_pref("browser.newtab.preload", false); // Disable new tab preloading
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

- Disable Idle in Intel sound card :
```
sudo nano /etc/sysfs.conf
```
- Add the following lines :
```
module/snd_hda_intel/parameters/power_save = 0
module/snd_hda_intel/parameters/power_save_controller = N
```
