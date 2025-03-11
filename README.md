










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



### 1.5 Configure apt sources

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


### 1.6 Update Apt

- Update the package lists:
```
sudo apt update
```


### 1.7 Set Timezone

- List available timezones:
```
sudo timedatectl list-timezones
```

- Set your timezone (replace `Your/Timezone` with your actual timezone, e.g., `Europe/Paris`):
```
sudo timedatectl set-timezone Your/Timezone
```




### 1.8 Add rights

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

- Note: It's generally recommended to use /etc/default/cpufrequtils for configuration if it exists.
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
devices/system/cpu/cpufreq/ondemand/up_threshold = 70
# Percentage CPU utilization before scaling up frequency.
devices/system/cpu/cpufreq/ondemand/sampling_down_factor = 6
# How aggressively the CPU scales down frequency. Higher values mean less aggressive scaling down.
devices/system/cpu/cpufreq/ondemand/sampling_rate = 20000000
# How often the CPU usage is checked. Higher values mean more frequent checks (more responsive).
devices/system/cpu/cpu0/cpufreq/cpuinfo_min_freq = XXXXXXX
# same speed than "MIN_SPEED="XXXXXXX"" in /etc/sysfs.conf
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

Reload
```
sudo sysctl -p

sudo systemctl daemon-reload
```


### 3.8 Grub Configuration

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


### 4.1 Xorg Configuration
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


    


### 4.2 Install Nvidia Driver and CUDA
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

### 4.43 Nvidia


3#### 4.4.1 Nvidia Xorg conf


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
    Option     "Coolbits" "28"              # "Coolbits" "28"  Enable Overclocking and FanControl
EndSection
```




#### 4.3.2 Modprobe conf
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



#### 4.43.3 Update Initramfs
```
sudo update-initramfs -u
```
- Reboot



### 4.4 Nvidia Patch (removes restriction on maximum number of simultaneous NVENC video encoding sessions)
```
git clone https://github.com/keylase/nvidia-patch.git
cd nvidia-patch
sudo bash ./patch.sh
```


### 4.5 Overclocking

**WARNING**: Overclocking can damage your hardware.  
Proceed with extreme caution, and increase values incrementally, testing
thoroughly after each change.  Monitor temperatures closely.


#### 4.5.1 Benchmark

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


#### 4.5.2 Overclock

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

#### 4.5.3 Make changes permanents

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

### 5.2 Install and Configure Firefox ESR

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


- Add Ublock Origin and set it : https://codeberg.org/celenity/Phoenix/wiki/Content-Blocking


#### 5.2.1 Enable Acceleration GPU Firefox

##### 1. nv-codec-headers
```
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
cd nv-codec-headers
make && sudo make install
```

##### 2. Then install all dependencies needed (extended version - you probably already have some of these packages)
```
sudo apt install build-essential git gstreamer1.0-plugins-bad libgstreamer-plugins-bad1.0-dev libffmpeg-nvenc-dev libva-dev libegl-dev meson
```

##### 4. Now you can clone the repo, build the driver and install it
```
git clone https://github.com/elFarto/nvidia-vaapi-driver
cd nvidia-vaapi-driver/
meson setup build
meson install -C build
```

##### 5. Set all the needed environment variables

```
sudo nano ~/.profile
```


- Add the following lines
```
# Enable elFarto's nvidia-vaapi-driver on Firefox
export MOZ_DISABLE_RDD_SANDBOX=1
export LIBVA_DRIVER_NAME=nvidia
export NVD_BACKEND=direct
```

##### 6 Firefox configuration

Open Firefox Profile Folder: Go to about:profiles in Firefox, find your active profile, and click "Open Folder."
  
- Create user.js: In the profile folder, create a new text file named user.js.

- Add the following lines :
```
// --- GRAPHICS & PERFORMANCE ---

// **WebRender (Graphics Rendering)**
// WebRender is a modern GPU-based rendering engine.  Enabling it can improve performance,
// especially on systems with dedicated graphics cards.  However, it may cause compatibility
// issues on some older hardware or with certain drivers.
user_pref("gfx.webrender.all", true); // Force enable WebRender on all pages.
user_pref("gfx.webrender.compositor.force-enabled", true); // Force enable the WebRender compositor.
// user_pref("gfx.webrender.force-enabled", true); // Force enable WebRender if gfx.webrender.all not work.
user_pref("gfx.x11-egl.force-enabled", true); // Force enable EGL on X11 (NO WAYLAND).  Improves graphics performance.
user_pref("gfx.canvas.accelerated", true); // Enable canvas acceleration.
user_pref("gfx.layers.acceleration.force-enabled", true); // Force enable layer acceleration.
user_pref("gfx.skia.force_enabled", true);  // Force enable the Skia graphics library.
user_pref("media.ffmpeg.vaapi.enabled", true);  // Enable VA-API

// DMA-BUF: Force enable DMA-BUF (Direct Memory Access Buffer) for graphics.
user_pref("widget.dmabuf.force-enabled", true);

// **Hardware Video Decoding**
// Using hardware acceleration for video decoding can significantly reduce CPU usage and improve
// battery life.  However, compatibility varies depending on your hardware and drivers.
user_pref("media.hardware-video-decoding.force-enabled", true); // Force enable hardware video decoding.
user_pref("media.gpu-process-decoder", true); // Enable GPU process decoding.

// Media: Disable RDD-FFmpeg, FFvpx and av1.
user_pref("media.rdd-ffmpeg.enabled", false);
user_pref("media.ffvpx.enabled", false);
user_pref("media.av1.enabled", false);

```

#### 5.2.1 Performance and Privacy Firefox for CPU Offloading

```
// --- NETWORK & CONNECTIVITY ---

// **Connection Settings**
user_pref("network.http.max-connections", 500); // Maximum number of HTTP connections.  Higher values can improve loading of pages with many resources, but can also be more aggressive.
user_pref("network.http.max-persistent-connections-per-proxy", 15); // Persistent connections per proxy.
user_pref("network.http.max-persistent-connections-per-server", 15); // More connections
user_pref("network.http.max-urgent-start-excessive-connections-per-host", 5); // Adjust excessive connections per host.
user_pref("network.auth.subresource-http-auth-allow", 1); // Allow HTTP authentication for subresources.

// **Prefetching & Speculative Connections (IMPORTANT: Privacy Implications)**
// Prefetching and speculative connections can speed up browsing by loading resources before
// they are explicitly requested.  However, this can also reveal information about your browsing
// habits to websites and trackers.  DISABLE these if you prioritize privacy.
user_pref("network.dns.disablePrefetch", true); // Disable DNS prefetching.  HIGHLY RECOMMENDED for privacy.
user_pref("network.predictor.enable-prefetch", false); // Disable prefetching by the network predictor.  HIGHLY RECOMMENDED for privacy.
user_pref("network.prefetch-next", false); // Disable link prefetching.  HIGHLY RECOMMENDED for privacy.
user_pref("browser.urlbar.speculativeConnect.enabled", false); // Disable speculative connections from the URL bar.  RECOMMENDED for privacy.
user_pref("network.predictor.enabled", false); // Disable network predictor. HIGHLY RECOMMENDED for privacy.
user_pref("network.dns.disablePrefetchFromHTTPS", true); // Disable DNS prefetching from HTTPS links.  RECOMMENDED for privacy.

// **Caching**
user_pref("browser.cache.disk.enable", true); // Enable disk caching.  improves CPU performance.
user_pref("browser.cache.memory.capacity", 512288);  // 512MB,  -1=default , 262144= 256MB.  Memory cache capacity.  Larger values can improve performance, but use more RAM.
user_pref("media.memory_cache_max_size", 512288); // 512MB.  Media memory cache size.
user_pref("browser.privatebrowsing.forceMediaMemoryCache", true); // Force media caching in memory during private browsing.
user_pref("network.buffer.cache.size", 262144); // Increased buffer cache size.
user_pref("media.cache_readahead_limit", 7200); // seconds.  Media cache readahead limit.
user_pref("media.cache_resume_threshold", 3600); // seconds.  Media cache resume threshold.
user_pref("browser.cache.jsbc_compression_level", 3); // JavaScript bytecode cache compression level.

// **DNS Cache**
user_pref("network.dnsCacheExpiration", 36000); // 10 hours.  DNS cache expiration time.  Longer values can reduce DNS lookups, but may cause issues if websites change IP addresses frequently.

// **SSL Token Cache**
user_pref("network.ssl_tokens_cache_capacity", 10240); // Increase SSL token cache capacity.

// **HTTP Pacing**
user_pref("network.http.pacing.requests.enabled", false); // Disable HTTP request pacing.

// **IPv4 Fallback**
user_pref("network.http.fast-fallback-to-IPv4", true); // Enable fast fallback to IPv4 if IPv6 is slow.

// **Captive Portal Detection (IMPORTANT: Privacy Implications)**
// Captive portal detection checks if you are connected to a network that requires login
// (e.g., public Wi-Fi).  This involves sending requests to a specific URL.  Disable this if
// you are concerned about privacy.
user_pref("captivedetect.canonicalURL", ""); // Disable captive portal detection.  RECOMMENDED for privacy.
user_pref("network.captive-portal-service.enabled", false); // Disable captive portal service.  RECOMMENDED for privacy.

// **Network Connectivity Service**
user_pref("network.connectivity-service.enabled", false); // Disable connectivity service.

// --- PRIVACY & SECURITY ---

// **Telemetry & Data Reporting (IMPORTANT: Privacy)**
// Telemetry and data reporting send usage data to Mozilla.  This data is used to improve
// Firefox, but it can also be a privacy concern.  DISABLE all of these settings if you
// prioritize privacy.
user_pref("datareporting.policy.dataSubmissionEnabled", false); // Disable data submission.  HIGHLY RECOMMENDED for privacy.
user_pref("datareporting.healthreport.uploadEnabled", false); // Disable health report uploading.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.unified", false); // Disable unified telemetry.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.enabled", false); // Disable telemetry.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.server", "data:,"); // Set telemetry server to an invalid address.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.archive.enabled", false); // Disable telemetry archiving.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.newProfilePing.enabled", false); // Disable new profile ping.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.shutdownPingSender.enabled", false); // Disable shutdown ping sender.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.updatePing.enabled", false); // Disable update ping.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.bhrPing.enabled", false); // Disable background hang reporting ping.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.telemetry.firstShutdownPing.enabled", false); // Disable first shutdown ping.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.coverage.opt-out", true); // Opt-out of coverage telemetry.  HIGHLY RECOMMENDED for privacy.
user_pref("toolkit.coverage.endpoint.base", ""); // Clear coverage endpoint.  HIGHLY RECOMMENDED for privacy.
user_pref("browser.newtabpage.activity-stream.feeds.telemetry", false); // Disable telemetry on the new tab page.  RECOMMENDED for privacy.
user_pref("browser.newtabpage.activity-stream.telemetry", false); // Disable telemetry on the new tab page.  RECOMMENDED for privacy.

// **Studies & Experiments (IMPORTANT: Privacy)**
// Studies and experiments allow Mozilla to test new features and collect data.  Disable these
// if you do not want to participate.
user_pref("app.shield.optoutstudies.enabled", false); // Disable Shield studies.  RECOMMENDED for privacy.
user_pref("app.normandy.enabled", false); // Disable Normandy.  RECOMMENDED for privacy.
user_pref("app.normandy.api_url", ""); // Clear Normandy API URL.  RECOMMENDED for privacy.

// **Crash Reporting (IMPORTANT: Privacy)**
// Crash reporting sends information about crashes to Mozilla.  This can help improve Firefox,
// but it can also be a privacy concern.
user_pref("breakpad.reportURL", ""); // Clear Breakpad report URL.  RECOMMENDED for privacy.
user_pref("browser.tabs.crashReporting.sendReport", false); // Disable sending crash reports.  RECOMMENDED for privacy.
user_pref("browser.crashReports.unsubmittedCheck.autoSubmit2", false); // Disable automatic submission of unsubmitted crash reports.  RECOMMENDED for privacy.
user_pref("toolkit.crashreporter.enabled", false);// Disable crash reporter. RECOMMENDED for privacy.

// **Safe Browsing (IMPORTANT: Security)**
// Safe Browsing helps protect you from malicious websites and downloads.  It is generally
// recommended to keep this enabled.  However it may send user information (e.g. URL, file hashes, etc.) to third parties like Google. RECOMMENDED for privacy, but slightly reduces security.
// about the files you download to Google.
user_pref("browser.safebrowsing.downloads.remote.enabled", false); // Disable remote Safe Browsing lookups for downloads.  
user_pref("browser.safebrowsing.malware.enabled", false); // disable the Safe Browsing service
user_pref("browser.safebrowsing.phishing.enabled", false); // disable the Safe Browsing service
user_pref("browser.safebrowsing.downloads.enabled", false); // Disable Safe Browsing lookups for downloads

// **Permissions (IMPORTANT: Security & Privacy)**
// These settings control default permissions for websites.  Blocking notifications and geolocation
// by default is a good privacy practice.
user_pref("permissions.default.desktop-notification", 2); // 2 = Block.  Block desktop notifications by default.
user_pref("permissions.default.geo", 2); // 2 = Block.  Block geolocation by default.
user_pref("permissions.manager.defaultsUrl", ""); // Clear default permissions URL.

// **Web Channels (IMPORTANT: Security)**
user_pref("webchannel.allowObject.urlWhitelist", ""); // Disable whitelisting for web channels.  Improves security.

// **HTTPS-First Mode (IMPORTANT: Security)**
// HTTPS-First Mode attempts to connect to all websites using HTTPS, which is more secure than HTTP.
user_pref("dom.security.https_first", true); // Always try to connect using HTTPS.  HIGHLY RECOMMENDED for security.
user_pref("dom.security.https_first_schemeless", true); // Apply HTTPS-First to URLs without a scheme. HIGHLY RECOMMENDED for security.

// **Mixed Content Blocking (IMPORTANT: Security)**
// Mixed content occurs when an HTTPS page loads resources (e.g., images, scripts) over HTTP.
// This is a security risk.
user_pref("security.mixed_content.block_display_content", true); // Block mixed display content (e.g., images).  HIGHLY RECOMMENDED for security.

// **Referrer Policy (IMPORTANT: Privacy)**
// The referrer header tells websites where you came from.  Reducing this information improves privacy.
user_pref("network.http.referer.XOriginTrimmingPolicy", 2); // Reduce referrer information sent to different origins.

// **Global Privacy Control (GPC) (IMPORTANT: Privacy)**
// GPC is a signal that you can send to websites to indicate that you do not want your data
// to be sold or shared.
user_pref("privacy.globalprivacycontrol.enabled", true); // Enable Global Privacy Control.

// **Query Stripping (IMPORTANT: Privacy)**
// Query stripping removes tracking parameters from URLs.
user_pref("privacy.query_stripping.enabled", true); // Enable query stripping.
user_pref("privacy.query_stripping.enabled.pbmode", true); // Enable query stripping in private browsing mode.

// **Certificate Revocation (IMPORTANT: Security)**
user_pref("security.pki.crlite_mode", 2); // Enable CRLite for certificate revocation checking.
user_pref("security.remote_settings.crlite_filters.enabled", true); // Enable remote settings for CRLite filters.
user_pref("security.OCSP.enabled", 0);// Disable OCSP (Online Certificate Status Protocol) stapling.

// **Security Hardening**
user_pref("security.ssl.treat_unsafe_negotiation_as_broken", true); // Treat unsafe negotiation as broken.  Improves security.
user_pref("browser.xul.error_pages.expert_bad_cert", true); // Show advanced information on certificate errors.  Helpful for troubleshooting.
user_pref("security.tls.enable_0rtt_data", false); // Disable 0-RTT data.  Improves security, but may slightly increase latency.
user_pref("security.insecure_connection_text.enabled", true); // Show warnings for insecure connections (HTTP).
user_pref("security.insecure_connection_text.pbmode.enabled", true); // Show warnings for insecure connections in private browsing mode.
user_pref("network.IDN_show_punycode", true); // Show Punycode for internationalized domain names (IDNs).  Helps prevent phishing attacks.

// **Cookies**
user_pref("network.cookie.sameSite.noneRequiresSecure", true); // Require SameSite=None cookies to be secure.  Improves security.

// **Content Blocking**
user_pref("browser.contentblocking.category", "strict"); // Set content blocking category to strict.  Blocks more trackers, but may break some websites.

// **WebRTC (Web Real-Time Communication) (IMPORTANT: Privacy)**
// WebRTC allows real-time communication (e.g., video chat) in the browser.  However, it can
// leak your IP address, even when using a VPN.  Disable it if you prioritize privacy and
// do not need WebRTC functionality.
user_pref("media.peerconnection.enabled", false); // Disable WebRTC entirely.  HIGHLY RECOMMENDED for privacy if you don't use WebRTC.
user_pref("media.peerconnection.video.enabled", false); // Disable WebRTC video.
user_pref("media.peerconnection.identity.enabled", false); // Disable WebRTC identity.
user_pref("media.navigator.enabled", false); // Disable media navigator.
user_pref("media.navigator.video.enabled", false); // Disable video in media navigator
user_pref("media.peerconnection.ice.proxy_only_if_behind_proxy", true); // Only use ICE proxy if behind a proxy.
user_pref("media.peerconnection.ice.default_address_only", true); // Only use default addresses for ICE.

// **Geolocation (IMPORTANT: Privacy)**
user_pref("geo.enable", false); // Disable geolocation.  HIGHLY RECOMMENDED for privacy.
user_pref("browser.search.geoip.url", "");// Disable Geo-specific search results.

// **PDF.js (Built-in PDF Viewer)**
user_pref("pdfjs.enableScripting", false); // Disable scripting in PDF.js.  Improves security.

// **Sanitizer API**
user_pref("dom.security.sanitizer.enabled", true); // Enable the Sanitizer API.  Improves security.

// **Web Speech (IMPORTANT: Privacy)**
user_pref("media.webspeech.synth.enabled", false); // Disable WebSpeech synthesis.

// **Private Attribution Token**
user_pref("dom.private-attribution.submission.enabled", false); // Disable Private Attribution Token

// --- UI (USER INTERFACE) & FUNCTIONALITY ---

// **URL Bar & Search**
user_pref("browser.urlbar.suggest.searches", false); // Disable search suggestions in the URL bar.  Improves privacy.
user_pref("browser.urlbar.suggest.history", false); // Disable history suggestions in the URL bar.  Improves privacy.
user_pref("browser.search.suggest.enabled", false); // Disable search suggestions in general.  Improves privacy.
user_pref("browser.urlbar.quicksuggest.enabled", false); // Disable QuickSuggest.  Improves privacy.
user_pref("browser.urlbar.suggest.quicksuggest.sponsored", false); // Disable sponsored QuickSuggest suggestions.
user_pref("browser.urlbar.suggest.quicksuggest.nonsponsored", false); // Disable non-sponsored QuickSuggest suggestions.
user_pref("browser.urlbar.trimHttps", true); // Trim HTTPS from the URL bar display.  Cosmetic change.
user_pref("browser.urlbar.trimURLs", false); // Do not trim URLs completely.
user_pref("browser.urlbar.groupLabels.enabled", false); // Disable URL bar group labels.
user_pref("browser.urlbar.update2.engineAliasRefresh", true); // Refresh engine alias.
user_pref("browser.urlbar.suggest.calculator", true); // Enable calculator suggestions in the URL bar.
user_pref("browser.urlbar.unitConversion.enabled", true); // Enable unit conversion suggestions in the URL bar.
user_pref("browser.urlbar.trending.featureGate", false); // Disable trending searches in the URL bar.

// **Search**
user_pref("browser.search.separatePrivateDefault.ui.enabled", true); // Separate private search defaults.
user_pref("browser.search.region", "US"); // Set the search region.  Change to your region.
user_pref("browser.search.update", false); // Disable search updates.
user_pref("keyword.enabled", true); // Enable keyword search (using bookmarks and the URL bar).

// **New Tab Page**
user_pref("browser.newtabpage.activity-stream.feeds.topsites", false); // Disable top sites on the new tab page.
user_pref("browser.newtabpage.activity-stream.feeds.section.topstories", false); // Disable top stories on the new tab page.
user_pref("extensions.pocket.enabled", false); // Disable Pocket integration.  RECOMMENDED for privacy.
user_pref("browser.newtabpage.activity-stream.feeds.snippets", false); // Disable snippets on the new tab page.
user_pref("browser.newtabpage.activity-stream.asrouter.providers.snippets", ""); // Clear snippets provider.
user_pref("browser.newtabpage.activity-stream.showSearch", false); // Disable search on the new tab page.
user_pref("browser.newtabpage.activity-stream.showSponsored", false); // Disable sponsored content on the new tab page.
user_pref("browser.newtabpage.activity-stream.showSponsoredTopSites", false); // Disable sponsored top sites on the new tab page.
user_pref("browser.newtabpage.activity-stream.section.highlights.includeBookmarks", false); // Don't include bookmarks in highlights.
user_pref("browser.newtabpage.activity-stream.section.highlights.includeDownloads", false); // Don't include downloads in highlights.
user_pref("browser.newtabpage.activity-stream.section.highlights.includeVisited", false); // Don't include visited pages in highlights.
user_pref("browser.newtabpage.activity-stream.section.highlights.includePocket", false); // Don't include Pocket in highlights.
user_pref("browser.newtabpage.activity-stream.asrouter.userprefs.cfr.addons", false); // Disable add-on recommendations.
user_pref("browser.newtabpage.activity-stream.asrouter.userprefs.cfr.features", false); // Disable feature recommendations.
user_pref("browser.newtab.preload", false); // Disable preloading of the new tab page.

// **Downloads**
user_pref("browser.download.always_ask_before_handling_new_types", true); // Always ask before handling new file types.  Improves security.
user_pref("browser.download.manager.addToRecentDocs", false); // Don't add downloaded files to the recent documents list.
user_pref("browser.download.open_pdf_attachments_inline", true); // Open PDF attachments inline in the browser.
user_pref("browser.download.start_downloads_in_tmp_dir", true); // Start downloads in the temporary directory.

// **Bookmarks**
user_pref("browser.bookmarks.openInTabClosesMenu", false); // Don't close the bookmarks menu when opening a bookmark in a new tab.

// **Context Menu**
user_pref("browser.menu.showViewImageInfo", true); // Show "View Image Info" in the context menu.

// **Find Bar**
user_pref("findbar.highlightAll", true); // Highlight all matches in the find bar.

// **Word Selection**
user_pref("layout.word_select.eat_space_to_next_word", false); // Don't eat spaces when selecting words.

// **Form Filling**
user_pref("browser.formfill.enable", false); // Disable form filling.  Improves privacy.
user_pref("signon.formlessCapture.enabled", false); // Disable formless login capture.
user_pref("signon.privateBrowsingCapture.enabled", false); // Disable private browsing capture.

// **Reader Mode**
user_pref("reader.parse-on-load.enabled", false); // Disable automatic reader mode parsing.

// **Full Screen API**
user_pref("full-screen-api.transition-duration.enter", "0 0"); // Disable full-screen transitions.
user_pref("full-screen-api.transition-duration.leave", "0 0"); // Disable full-screen transitions.
user_pref("full-screen-api.warning.delay", -1); // Disable full-screen warning delay.
user_pref("full-screen-api.warning.timeout", 0); // Disable full-screen warning timeout.

// **Cookie Banner Handling**
user_pref("cookiebanners.service.mode", 1); // Enable cookie banner handling service (but don't automatically reject).  1 = Warn, 2 = Reject if possible, otherwise warn.
user_pref("cookiebanners.service.mode.privateBrowsing", 1); // Enable cookie banner handling in private browsing mode.

// **Clipboard Events**
user_pref("dom.event.clipboardevents.enabled", false); // Disable clipboard events (websites can't see what you copy/paste).  Improves privacy.

// **Editor**
user_pref("editor.truncate_user_pastes", false); // Disable truncating user pastes in the editor.

// **Accessibility**
user_pref("accessibility.force_disabled", 1); // Force accessibility services to be disabled.

// **Session & Tab Management**
user_pref("browser.sessionstore.max_tabs_undo", 0); // Disable undoing closed tabs.  Improves privacy.
user_pref("browser.sessionstore.interval", 60000); // Save session every 60 seconds (instead of the default 15).
user_pref("browser.sessionstore.resume_from_crash", false); // Disable resuming from crashes.

// **User Context (Containers)**
user_pref("privacy.userContext.enabled", true); // Enable user context (containers).  Allows you to separate browsing contexts (e.g., work, personal).
user_pref("privacy.userContext.ui.enabled", true); // Enable the user context UI.
user_pref("privacy.usercontext.about_newtab_segregation.enabled", true); // Enable new tab segregation for containers.

// **Web Task Scheduling**
user_pref("dom.enable_web_task_scheduling", true); // Enable web task scheduling.

// **Animations**
user_pref("dom.animations-api.animations.enabled", true); // Enable animations API.
user_pref("image.animation_mode", "none"); // Don't animate images.  Improves performance and reduces distractions.

// **Timing APIs (IMPORTANT: Privacy - Fingerprinting)**
// These APIs can be used for fingerprinting (identifying your browser uniquely).  Disabling
// them improves privacy, but may break some websites that rely on them.
user_pref("dom.enable_event_timing", false); // Disable event timing.
user_pref("dom.enable_resource_timing", false); // Disable resource timing.
user_pref("dom.enable_user_timing", false); // Disable user timing.
user_pref("dom.enable_performance_timing", false); // Disable performance timing

// **Default Browser Check**
user_pref("browser.shell.checkDefaultBrowser", false); // Disable default browser check.

// **Activity Stream (Library)**
user_pref("browser.library.activity-stream.enabled", false); // Disable Activity Stream in the library.

// **Add-ons & Recommendations**
user_pref("extensions.getAddons.showPane", false); // Hide the "Get Add-ons" panel.
user_pref("extensions.htmlaboutaddons.recommendations.enabled", false); // Disable add-on recommendations.
user_pref("browser.discovery.enabled", false); // Disable add-on and theme discovery.
user_pref("extensions.postDownloadThirdPartyPrompt", false); // Disable third-party prompts after extension downloads.
user_pref("extensions.update.autoUpdateDefault", false); // Disable automatic updates for extensions.  IMPORTANT: Keep extensions updated manually for security.
user_pref("extensions.update.enabled", false); // Disable extension update checks.  IMPORTANT: Keep extensions updated manually for security.
user_pref("extensions.htmlaboutaddons.discover.enabled", false); // Disable add-ons discovery.

// **UI Tweaks**
user_pref("browser.preferences.moreFromMozilla", false); // Disable "More from Mozilla" in preferences.
user_pref("browser.tabs.tabmanager.enabled", false); // Disable tab manager.
user_pref("browser.aboutConfig.showWarning", false); // Disable about:config warning.  BE CAREFUL in about:config!
user_pref("browser.aboutwelcome.enabled", false); // Disable about:welcome.
user_pref("toolkit.legacyUserProfileCustomizations.stylesheets", true); // Enable support for user stylesheets (userChrome.css, userContent.css).
user_pref("browser.compactmode.show", true); // Show compact mode option in the UI.
user_pref("browser.display.focus_ring_on_anything", true); // Show focus ring on any focused element.
user_pref("browser.display.focus_ring_style", 0); // Solid focus ring style.
user_pref("browser.display.focus_ring_width", 0); // No width for the focus ring.
user_pref("browser.uitour.enabled", false); // Disable UITour.
user_pref("layout.css.prefers-color-scheme.content-override", 2); // Prefer dark color scheme for web content.  0=system, 1=light, 2=dark, 3=follow browser theme

// **Onboarding**
user_pref("browser.onboarding.enabled", false); // Disable onboarding.

// **Startup**
user_pref("browser.startup.homepage", "about:blank"); // Set the homepage to a blank page.
user_pref("browser.newtabpage.enabled", false); // Disable the default new tab page.  Use about:blank or a custom new tab extension.
user_pref("browser.startup.page", 0); // 0 = blank page, 1 = homepage, 3 = last session.

// **VPN Promotion**
user_pref("browser.privatebrowsing.vpnpromourl", "");// Disable VPN promotion URL.

// **Screenshots**
user_pref("extensions.screenshots.upload-disabled", true); // Disable uploading screenshots.

// **Page Thumbnails**
user_pref("browser.pagethumbnails.capturing_disabled", true); // Disable capturing page thumbnails.

// **Helper Apps**
user_pref("browser.helperApps.deleteTempFileOnExit", true); // Delete temporary files on exit.

// **Content Notify Interval**
user_pref("content.notify.interval", 20000); // 20 seconds.  Increase the interval for content notifications.

// **Fission (Site Isolation) (IMPORTANT: Security)**
// Fission is a security feature that isolates websites into separate processes.  This can
// prevent one website from accessing data from another.
user_pref("fission.autostart", false); 	// Disable Fission.  WARNING ! May improve compatibility, but reduces security.

// **Process Count (IMPORTANT: Security & Performance)**
// These settings control the number of processes used by Firefox.  More processes can improve
// security and stability, but also use more memory.
user_pref("dom.ipc.processCount.inference", 0); // Disable process count inference.
user_pref("dom.ipc.processCount.webIsolated", 1); // Number of processes for isolated web content.
user_pref("dom.ipc.processCount", 1); // Number of processes.  Lower values use less memory, but reduce security and may impact performance.
user_pref("dom.ipc.processCount.webCOOP+COEP", 0); // Disable processes for COOP+COEP.
user_pref("dom.ipc.processPrelaunch.fission.number", 0); // Disable Fission process prelaunch.

// **Privacy: History**
user_pref("privacy.history.custom", true);

// --- Uncategorized ---
// (Settings that need further review and categorization)
user_pref("browser.aboutHomeSnippets.updateUrl", "");
user_pref("privacy.trackingprotection.enabled", false);
user_pref("media.gmp-manager.url", "");
user_pref("media.gmp-gmpopenh264.enabled", true);
```


### 5.3 Disable Journaling on ext4

**EXTREME CAUTION**: Disabling journaling significantly increases the risk of data loss in case of a power failure or system crash.  This is NOT RECOMMENDED for most users.  The following steps involve creating a backup of your system, which is essential before disabling journaling.


#### 5.3.1 Backup your system

- Download Rescuezilla ( https://github.com/rescuezilla/rescuezilla/releases/download/2.5.1/rescuezilla-2.5.1-64bit.noble.iso ).

Create a bootable USB drive with Rescuezilla :


- Find USB device name :
```
lsblk # Identify your Debian disk (e.g., /dev/sda)
```

- Unmount USB
```
sudo umount /dev/sdX # Where sdX is your USB
```

- Create the disk image with dd :
```
sudo dd if=~/Downloads/rescuezilla-2.5.1-64bit.noble.iso of=/dev/sdX bs=1M status=progress   # Where sdX is your USB device name
```
- Boot from it, and back up your entire Debian disk to another storage device 
(USB drive, external hard drive, etc.).  Do not proceed without a complete backup.


#### 5.3.2 Disable journalling on Ext4

**WARNING**: This can lead to data loss!  Ensure you have a complete backup before proceeding.
```
lsblk    # Identify your root partition (e.g., /dev/nvme0n1p2)

sudo tune2fs -O ^has_journal /dev/nvme0nXXX    # Replace /dev/nvme0nXXX with your actual root ( / ) partition (you can find it with : " lsblk ")
```


### 5.4 Autologin Lightdm

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


### 5.5 PulseAudio Configuration
```
sudo nano /etc/pulse/daemon.conf
```

- Uncomment (remove the leading ;) and modify the following lines:

```
flat-volumes = no
avoid-resampling = true
```


#### 5.5.1 Disable idle and tsched
```
sudo nano /etc/pulse/system.pa
```

- Add and Comment this lines:
```
load-module module-udev-detect tsched=0
# disables the timer-based scheduling feature of the module. (This can improve performance, but might introduce latency)
#load-module module-suspend-on-idle
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

VA-API - https://github.com/elFarto/nvidia-vaapi-driver


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





