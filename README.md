



---
title: Max Performance Debian XFCE NVIDIA
shortTitle: Debian XFCE NVIDIA Tuning
intro: 'This tutorial guides you through optimizing a Debian 12 system with XFCE and NVIDIA drivers for maximum performance. It covers system installation, essential utilities, performance tuning, NVIDIA driver setup, and advanced configurations.'
product: "{{ optional product callout }}"
type: tutorial
topics:
  - linux
  - debian
  - xfce
  - nvidia
  - performance
versions:
  - bookworm
---



## Introduction


This tutorial is intended for users with some familiarity with the Linux command line who want to maximize the performance of their Debian 12 (Bookworm) system running the XFCE desktop environment and using NVIDIA graphics cards.  It assumes you have a basic understanding of concepts like package management, system services, and file system configuration.

You will accomplish a complete system setup, from a minimal Debian installation to a highly optimized environment. This includes installing and configuring XFCE, essential utilities, NVIDIA drivers, and applying various system tweaks to reduce overhead and improve responsiveness.  The goal is to achieve the highest possible performance for tasks like gaming, video editing, and GPU-accelerated computing.  There is no specific "project" to complete, but the end result is a fully configured, performance-tuned system.

**Important Considerations and Warnings:**

*   **Security:** Several steps in this tutorial involve disabling security features (AppArmor, mitigations, `allowed_users=anybody` in Xwrapper.config, overly permissive `sudoers` entries).  *Strongly* consider the security implications before proceeding.  If you are not comfortable with these risks, do *not* disable these features.  Alternatives are often available (e.g., configuring AppArmor instead of disabling it).
*   **Data Loss:** Disabling journaling on your ext4 filesystem is *extremely risky* and can lead to data loss or corruption in the event of a power outage or system crash.  Do *not* do this unless you have a robust backup strategy and understand the risks.
*   **Overclocking:** Overclocking your GPU can lead to instability, overheating, and even hardware damage.  Proceed with extreme caution and follow the instructions carefully, monitoring temperatures and stability at each step.
*   **System Stability:** Many of the tweaks in this tutorial push the system towards its limits.  Thorough testing is crucial after each major change to ensure stability.

## Step 1: Base System Installation and Configuration

{% comment %}
In one sentence, describe what the user will do in this step
Steps should break down the tasks the user will complete in sequential order
Avoid replicating conceptual information that is covered elsewhere, provide inline links instead. Only include conceptual information unique to this use case.
{% endcomment %}

In this step, you will install a minimal Debian 12 system, install XFCE, configure package sources, and set up basic system settings.

### 1.1 Install Debian 12 Netinst

{% comment %}
A step may require the user to perform several tasks - break those tasks down into chunks, allowing the user to scan quickly to find their place if they navigated away from this screen to perform the task.
An example might be creating a personal access token for the action to use and then storing it in secrets
For UI based tasks, include the button or options the users should click
If the task adds code, include the code in context (don't just show `needs: setup` show the entire `setup` and `dependent` jobs)
{% endcomment %}

Download the Debian 12 Netinst image and create a bootable USB drive or CD.  Boot from the installation media and follow the on-screen instructions.  During the software selection step, *choose only "standard system utilities."*

### 1.2 Install Xfce4

After the base system installation, log in as root and install XFCE:

```bash
su -
apt install xfce4
reboot

title: Max Performance Debian XFCE NVIDIA

shortTitle: Debian XFCE NVIDIA Tuning

intro: 'This tutorial guides you through optimizing a Debian 12 system with XFCE and NVIDIA drivers for maximum performance. It covers system installation, essential utilities, performance tuning, NVIDIA driver setup, and advanced configurations.'

product: "{{ optional product callout }}"

type: tutorial

topics:

  - linux

  - debian

  - xfce

  - nvidia

  - performance

versions:

  - bookworm

---









This tutorial is intended for users with some familiarity with the Linux command line who want to maximize the performance of their Debian 12 (Bookworm) system running the XFCE desktop environment and using NVIDIA graphics cards.  It assumes you have a basic understanding of concepts like package management, system services, and file system configuration.



You will accomplish a complete system setup, from a minimal Debian installation to a highly optimized environment. This includes installing and configuring XFCE, essential utilities, NVIDIA drivers, and applying various system tweaks to reduce overhead and improve responsiveness.  The goal is to achieve the highest possible performance for tasks like gaming, video editing, and GPU-accelerated computing.  There is no specific "project" to complete, but the end result is a fully configured, performance-tuned system.



**Important Considerations and Warnings:**



*   **Security:** Several steps in this tutorial involve disabling security features (AppArmor, mitigations, `allowed_users=anybody` in Xwrapper.config, overly permissive `sudoers` entries).  *Strongly* consider the security implications before proceeding.  If you are not comfortable with these risks, do *not* disable these features.  Alternatives are often available (e.g., configuring AppArmor instead of disabling it).

*   **Data Loss:** Disabling journaling on your ext4 filesystem is *extremely risky* and can lead to data loss or corruption in the event of a power outage or system crash.  Do *not* do this unless you have a robust backup strategy and understand the risks.

*   **Overclocking:** Overclocking your GPU can lead to instability, overheating, and even hardware damage.  Proceed with extreme caution and follow the instructions carefully, monitoring temperatures and stability at each step.

*   **System Stability:** Many of the tweaks in this tutorial push the system towards its limits.  Thorough testing is crucial after each major change to ensure stability.



## Step 1: Base System Installation and Configuration



{% comment %}

In one sentence, describe what the user will do in this step

Steps should break down the tasks the user will complete in sequential order

Avoid replicating conceptual information that is covered elsewhere, provide inline links instead. Only include conceptual information unique to this use case.

{% endcomment %}



In this step, you will install a minimal Debian 12 system, install XFCE, configure package sources, and set up basic system settings.



### 1.1 Install Debian 12 Netinst



{% comment %}

A step may require the user to perform several tasks - break those tasks down into chunks, allowing the user to scan quickly to find their place if they navigated away from this screen to perform the task.

An example might be creating a personal access token for the action to use and then storing it in secrets

For UI based tasks, include the button or options the users should click

If the task adds code, include the code in context (don't just show `needs: setup` show the entire `setup` and `dependent` jobs)

{% endcomment %}



Download the Debian 12 Netinst image and create a bootable USB drive or CD.  Boot from the installation media and follow the on-screen instructions.  During the software selection step, *choose only "standard system utilities."*



### 1.2 Install Xfce4



After the base system installation, log in as root and install XFCE:



```bash

su -

apt install xfce4

reboot


1.3 Configure apt sources


Modify the apt sources list to include contrib, non-free, and non-free-firmware repositories:


Bash


su -

nano /etc/apt/sources.list


Replace the entire content of the file with the following:


deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

deb http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware



1.4 (Missing in original, but necessary) Update package lists


Bash


sudo apt update


1.5 (Renumbered) Update Apt:


Bash


sudo apt update


1.6 (Renumbered) Set Timezone:


First, list available timezones:


Bash


sudo timedatectl list-timezones


Then, set your timezone (replace Your/Timezone with the correct value, e.g., Europe/Paris):


Bash


sudo timedatectl set-timezone Your/Timezone


1.7 (Renumbered) Add user to sudoers


Install sudo and add your user to the sudo group:


Bash


su -

apt install sudo

exit

sudo usermod -aG sudo $USER


1.8 (Renumbered) Add rights


Add your user to the video and audio groups:


Bash


sudo usermod -aG video $USER

sudo usermod -aG audio $USER


1.9 (Renumbered) Allow members of group sudo to execute any command


WARNING: The original tutorial suggests using NOPASSWD: ALL.  This is generally not recommended for security reasons.  It's better to require a password for sudo.  The original instructions are included below, but be aware of the risk.


Bash


sudo nano /etc/sudoers


Original (Insecure): Replace %sudo   ALL=(ALL:ALL) ALL with:


%sudo   ALL=(ALL:ALL) NOPASSWD: ALL



Recommended (More Secure):  Leave the line as it is, or if it's not present, add:


%sudo   ALL=(ALL:ALL) ALL



Bash


reboot


Step 2: Essential Utilities and Desktop Configuration


Install essential utilities and configure your desktop environment.


2.1 Install Essential Utilities:


Bash


sudo apt install acpid dbus-x11 accountsservice apt-transport-https ca-certificates curl software-properties-common -y

sudo apt install mousepad xfce4-terminal nodejs npm lshw net-tools gmtp -y


2.2 Set Default Applications:


Open the XFCE Settings Manager, navigate to "Default Applications," 
and under "Utilities," set the "Terminal Emulator" to "Xfce Terminal."


2.3 Disable Apparmor:


WARNING: Disabling AppArmor significantly reduces 
system security.  Understand the implications before proceeding.  
Consider configuring AppArmor profiles instead of disabling it entirely.


Bash


sudo systemctl stop apparmor

sudo systemctl disable apparmor

sudo aa-teardown


2.4 Dark Mode (Xfce):


Open the XFCE Settings Manager, go to "Appearance," and choose "Adwaita Dark."


2.4.1 Set Terminal


Open the XFCE Settings Manager, go to "Default Applications," then "Utilities," and set "Terminal Emulator" to "Xfce Terminal."


2.5 Install and Configure Firefox ESR:


Bash


sudo apt install firefox-esr


2.5.1 Firefox Dark Mode:


Open Firefox, go to Settings > General > Language and Appearance > Colors > Manage Colors...


Set Text: White
Set Background: dark grey
Set Unvisited Links: White
Set Visited Links: light gray
Choose "Always"


2.5.2 Tweaks Firefox ESR


Refer to the Betterfox project on GitHub for potential Firefox ESR tweaks: https://github.com/yokoffing/Betterfox/tree/esr128


Step 3: Performance Tuning


Apply various system tweaks to improve performance.


3.1 Disable Suspend:


Bash


sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target


3.2 Create and Configure xset_noblank Script:


Create a script to disable screen blanking:


Bash


sudo nano /home/$USER/xset_noblank.sh


Add the following content:


Bash


#!/bin/bash

xset s noblank

xset s noexpose

xset dpms 0 0 0

xset s off

xset -dpms


3.3 Make script executable:


Bash


sudo chmod +x /home/$USER/xset_noblank.sh


3.4 Add to autostart:


Open the XFCE Settings Manager, go to "Session and Startup," then "Application Autostart."  Click "+ Add."


Set Name: xset_noblank
Set Command: /home/$USER/xset_noblank.sh
Uncheck "screen-locker"


Bash


reboot


3.5 Install cpufrequtils and sysfsutils:


Bash


sudo apt install -y cpufrequtils sysfsutils


3.5.1 Enable Services


Bash


sudo systemctl enable cpufrequtils

sudo systemctl enable sysfsutils


3.6 Disable Core Dumps:


Bash


sudo nano /etc/sysctl.d/50-coredump.conf


Add the following line:


kernel.core_pattern=|/bin/false



3.7 Enable the service:


Bash


sudo sysctl -p /etc/sysctl.d/50-coredump.conf


3.8 Configure CPU Governor and Frequencies:


Note: The original tutorial suggests editing /etc/init.d/cpufrequtils.  It's generally better to use /etc/default/cpufrequtils if it exists.


Bash


sudo nano /etc/default/cpufrequtils  # Or /etc/init.d/cpufrequtils if the former doesn't exist


If using /etc/default/cpufrequtils:


ENABLE="true"
GOVERNOR="ondemand"
MAX_SPEED="4400000"  # Replace with your CPU's max speed (use cpufreq-info)
MIN_SPEED="3500000"  # Replace with a reasonable minimum (e.g., half of max)




If using /etc/init.d/cpufrequtils (less recommended):  Replace the existing lines with the same content as above.


3.9 Sysfs Tuning:


Bash


sudo nano /etc/sysfs.conf


Add the following lines:


devices/system/cpu/cpufreq/ondemand/up_threshold = 99
devices/system/cpu/cpufreq/ondemand/sampling_down_factor = 6
devices/system/cpu/cpufreq/ondemand/sampling_rate = 20000000
vm.swappiness = 10
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
vm.vfs_cache_pressure = 50
module/snd_hda_intel/parameters/power_save = 0
module/snd_hda_intel/parameters/power_save_controller = N



3.10 Grub Configuration:


WARNING: Disabling security mitigations (mitigations=off kernel.randomize_va_space=0) is highly discouraged and significantly reduces system security.  Do not do this unless you have a very specific reason and understand the risks.


Bash


sudo nano /etc/default/grub


Replace the GRUB_CMDLINE_LINUX_DEFAULT line with:


GRUB_CMDLINE_LINUX_DEFAULT="processor.ignore_ppc=1 intel_pstate=passive intel_idle.max_cstate=0 idle=poll nosmt=force pcie_aspm=off mitigations=off kernel.randomize_va_space=0 ipv6.disable=1"



Recommended (More Secure): Remove mitigations=off kernel.randomize_va_space=0 from the line above.


3.11 Reload Grub


Bash


sudo grub-mkconfig -o /boot/grub/grub.cfg

reboot


3.12 Check Temperatures and CPU Frequencies:


Bash


sudo sensors

sudo cpufreq-info


Step 4: Python, NVIDIA, and Tweaks


Install Python, NVIDIA drivers, and configure related settings.


4.1 Install Python with pyenv:


Bash


sudo apt update

sudo apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl git libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

curl -fsSL https://pyenv.run | bash


4.1.2 bashrc and profile path


Bash


echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc

echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc

echo 'eval "$(pyenv init - bash)"' >> ~/.bashrc

exec "$SHELL"


4.1.3 Install Python


Bash


pyenv install 3.12.5

pyenv global 3.12.5


4.2 Xorg Configuration:


Bash


sudo usermod -a -G input $USER

sudo nano /etc/X11/Xwrapper.config


Add the following lines:


allowed_users=anybody
needs_root_rights=yes



WARNING: Setting allowed_users=anybody is a major security risk.  Anyone on the system (or potentially remotely, depending on your configuration) can access your X server.  Do not do this unless you absolutely understand the implications.  allowed_users=console is a much safer alternative.


4.3 Install Nvidia Driver and CUDA:


Bash


sudo apt install -y linux-headers-amd64 nvidia-driver firmware-misc-nonfree

reboot  # Reboot after driver installation

sudo apt install -y nvidia-cuda-dev nvidia-cuda-toolkit

reboot  # Reboot after CUDA installation


4.3.1 CUDA Rights and Group Configuration:


Bash


sudo groupadd cuda

sudo usermod -aG cuda $USER


4.3.2 Allow members of group cuda to execute nvidia-smi:


WARNING: The original tutorial uses a very permissive sudoers entry.  This is generally not recommended.


Bash


sudo nano /etc/sudoers


Add the following line:


%cuda ALL=(ALL) NOPASSWD: /usr/bin/nvidia-smi



Recommended (More Secure):  Do not use NOPASSWD.  Require a password for sudo.  A better approach might be to use capabilities instead of sudo for this specific task.


4.4 Nvidia:


4.4.1 Nvidia Xorg conf


Create or edit /etc/X11/xorg.conf.d/20-nvidia.conf (create the directory and file if they don't exist):


Bash


sudo mkdir -p /etc/X11/xorg.conf.d

sudo nano /etc/X11/xorg.conf.d/20-nvidia.conf


Add the following lines:


Section "OutputClass"
    Identifier     "nvidia"
    MatchDriver    "nvidia-drm"
    Driver         "nvidia"
    Option "AllowEmptyInitialConfiguration"
    Option "DPMS" "false"
    ModulePath "/usr/lib/nvidia/current" # Check if exist in your OS and replace it if necessary
    ModulePath "/usr/lib/xorg/modules"
    Option     "Coolbits" "28"
EndSection



Important: Verify the ModulePath entries.  The correct paths may vary depending on your specific NVIDIA driver version and installation.  Use find /usr -name "libnvidia-*.so" to locate the correct paths.


4.4.2 Modprobe conf


Create or edit /etc/modprobe.d/nvidia-options.conf:


Bash


sudo nano /etc/modprobe.d/nvidia-options.conf


Add the following lines:


options nvidia NVreg_UsePageAttributeTable=1
options nvidia NVreg_EnableMSI=1
options nvidia NVreg_EnableStreamMemOPs=1
options nvidia NVreg_RegistryDwords="PerfLevelSrc=0x2222"
options nvidia NVreg_RegistryDwords="OverrideMaxPerf=0x1"
options nvidia-drm modeset=1
options nvidia-drm fbdev=1



4.4.3 Profiling to user:


Bash


sudo nano /etc/modprobe.d/nvidia-options.conf


Uncomment (remove the #) the following line if it exists, or add it if it doesn't:


options nvidia-current NVreg_RestrictProfilingToAdminUsers=0



4.4.4 Update Initramfs:


Bash


sudo update-initramfs -u

reboot


4.5 Nvidia Patch (removes restriction on maximum number of simultaneous NVENC video encoding sessions)


Bash


git clone https://github.com/keylase/nvidia-patch.git

cd nvidia-patch

sudo bash ./patch.sh


4.6 Overclocking


WARNING: Overclocking can damage your hardware.  Proceed with extreme caution, monitor temperatures, and test stability thoroughly.


4.6.1 Benchmark


Download the Unigine Heaven benchmark: https://benchmark.unigine.com/heaven


Bash


cd Downloads

sudo chmod +x Unigine_Heaven-4.0.run

sudo ./Unigine_Heaven-4.0.run

cd Unigine_Heaven-4.0

./heaven


Run the benchmark and note the initial performance, temperatures, and clock speeds.


4.6.2 Overclock


Important: Increase clock offsets incrementally and run the benchmark after each change to ensure stability.  Start with small increases (e.g., +10 MHz for graphics clock, +100 MHz for memory clock).




Set max performance:


Bash




sudo nvidia-settings -a '[gpu:0]/GPUFanControlState=1'





sudo nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=XXXX'




4.6.3 Make changes permanents:


Create a script to apply overclocking settings:


Bash


sudo nano /home/$USER/GpuTweaks.sh


Add the following lines (replace XXX, XXXX, and power limit with your tested and stable values):


Bash


#!/bin/bash

## Set power limit to maximum

sudo nvidia-smi --power-limit=XXX #  Where XXX is value for power-limit



## Set maximum performance mode

sudo nvidia-settings -a '[gpu:0]/GpuPowerMizerMode=1'



## Set fan control

sudo nvidia-settings -a '[gpu:0]/GPUFanControlState=1'

sudo nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=100'



## Graphics overclocking

sudo nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffsetAllPerformanceLevels=XXX' #  Where XXX is the value added to the maximum graphics clocks



## Memory overclocking

sudo nvidia-settings -a '[gpu:0]/GPUMemoryTransferRateOffsetAllPerformanceLevels=XXXX' #  Where XXXX is the value added to the maximum Memory clocks


Make script executable :


Bash


sudo chmod +x /home/$USER/GpuTweaks.sh


Add to autostart :


Go to Applications > Settings > Session and Startup > Application Autostart
Click "+ Add"
Set Name: GpuTweaks
Set Command: /home/$USER/GpuTweaks.sh


Bash


reboot


4.7 Install FFmpeg:


Bash


sudo apt install libffmpeg-nvenc-dev

sudo apt install ffmpeg


Step 5: Filesystem, Boot, and Service Configurations


Further system tweaks related to the filesystem, boot process, and system services.


5.1 Set noatime and Tweak fstab for Ext4:


WARNING: The barrier=0 option is generally not recommended for modern filesystems and hardware, as it can increase the risk of data corruption in case of power loss.


Bash


sudo nano /etc/fstab


Add noatime to the options for your root filesystem (replace YOUR_ROOT_PARTITION_UUID with the actual UUID of your root partition, which you can find using lsblk -f):


UUID=YOUR_ROOT_PARTITION_UUID / ext4 noatime,errors=remount-ro 0 1



Recommended (More Secure):  Remove barrier=0 from the options.  Leave the /boot/efi line unchanged if you have an EFI system.


Bash


sudo systemctl daemon-reload


5.2 Disable Journaling on ext4:


EXTREME WARNING: Disabling journaling on your ext4 filesystem is extremely risky and can lead to data loss or corruption.  Do not
 do this unless you have a robust backup strategy and understand the 
risks.  The original tutorial's instructions are included below, but 
this step is strongly discouraged.


The original tutorial includes instructions for creating a disk image and backing up the system.  These instructions are not
 related to disabling journaling and seem misplaced.  I've omitted them 
here.  If you need to back up your system, use a reliable backup tool 
like rsync, timeshift, or a dedicated disk imaging utility.


To disable journaling (again, not recommended):


Bash


lsblk  # Identify your root partition (e.g., /dev/nvme0n1p2)

sudo tune2fs -O ^has_journal /dev/nvme0n1p2  # Replace with your actual root partition


5.3 Autologin Lightdm:


WARNING: Enabling autologin is a security risk.  Anyone with physical access to your computer will be able to log in automatically.


Bash


sudo nano /etc/lightdm/lightdm.conf


Add or modify the following lines (replace $USER with your username):


[LightDM]
logind-check-graphical=true

[Seat:*]
autologin-user=$USER
autologin-user-timeout=0



5.4 PulseAudio Configuration:


Bash


sudo nano /etc/pulse/daemon.conf


Uncomment (remove the ;) and modify the following lines:


flat-volumes = no
avoid-resampling = yes
default-sample-rate = 48000



5.5 Disable idle


Bash


sudo nano /etc/pulse/system.pa


Comment this line :


load-module module-suspend-on-idle



Add this line :


load-module module-udev-detect tsched=0



5.6 Disable Services


Note: Disabling services can improve performance but
 may also remove functionality you need.  Carefully consider the purpose
 of each service before disabling it.


Bash


sudo systemctl disable cron.service

sudo systemctl disable avahi-daemon.service

sudo systemctl disable e2scrub_reap.service

sudo systemctl disable logrotate.service

sudo systemctl disable logrotate.timer

sudo systemctl disable systemd-networkd.service  # Only if using a static network configuration

sudo systemctl disable systemd-timesyncd.service

sudo systemctl disable upower.service  # Only if you don't need battery monitoring/power management

sudo systemctl disable apt-daily-upgrade.service

sudo systemctl disable apt-daily.timer

sudo systemctl disable packagekit.service

sudo systemctl disable nftables.service  # Only if not using nftables for firewalling


Further reading




Include a bulleted list of tutorials or articles the user can reference to extend the concepts taught in this tutorial



Debian Wiki - CpuFrequencyScaling
Arch Linux Wiki - Improving performance
Arch Linux Wiki - Ext4
Arch Linux Wiki - LightDM
Arch Linux Wiki - PulseAudio/Troubleshooting
NVIDIA Developer Forums
Betterfox Project (GitHub)
pyenv Project (GitHub)
nvidia-patch Project (GitHub)


