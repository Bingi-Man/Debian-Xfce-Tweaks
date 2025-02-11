




Debian  Tweaks Xfce4 - Nvidia Overclocking Install Guide




This guide will allow you to install a minimal Xfce desktop, adding additional 
packages as needed and max performance for Nvidia GPU.









1- BASE SYSTEM INSTALLATION AND CONFIGURATION



1.1 - Install Debian 12 Netinst :

        - Choose only "standard system utilities" during installation. This minimizes unnecessary packages.



1.2 - Install Xfce4:

su -

apt install xfce4

- REBOOT



1.3 - Configure apt sources:

su -

nano /etc/apt/sources.list

- Replace the content with:


deb http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main contrib non-free non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware

deb http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-updates main contrib non-free non-free-firmware
    
    
    
1.5 - Update Apt:

sudo apt update



1.6 - Set Timezone:

sudo timedatectl list-timezones

sudo timedatectl set-timezone Your/Timezone  # Replace Your/Timezone with your actual timezone (e.g., Europe/Paris)



1.7 - Add user to sudoers :

su -

apt install sudo

exit  # Exit the root shell

sudo usermod -aG sudo $USER # Add current user to sudo group. 



1.8 - Add rights

sudo usermod -aG video $USER # Add current user to video group
sudo usermod -aG audio $USER # Add user to audio group



1.9 - Allow members of group sudo to execute any command ( https://superuser.com/questions/1495807/can-someone-explain-what-is-user-all-all-nopasswdall-does-in-sudoers-file )

sudo nano /etc/sudoers

- Replace this lines :

%sudo   ALL=(ALL:ALL) NOPASSWD: ALL

- REBOOT




2 - ESSENTIAL UTILITIES AND DESKTOP CONFIGURATION



2.1 - Install Essential Utilities:

sudo apt install acpid dbus-x11 accountsservice apt-transport-https ca-certificates curl software-properties-common -y

sudo apt install mousepad xfce4-terminal nodejs npm lshw net-tools gmtp



2.2 - Set Default Applications:

    - Go to Settings > Default Applications > Utilities
    - Set Terminal Emulator to Xfce Terminal



2.3 - Disable Apparmor :

sudo systemctl stop apparmor

sudo systemctl disable apparmor

sudo aa-teardown

    - Caution: Disabling Apparmor reduces security. Understand the implications before proceeding. Consider alternatives like configuring Apparmor to allow necessary access instead of completely disabling it.



2.4 - Dark Mode (Xfce):

    - Go to Applications > Settings > Appearance
    - Choose Adwaita Dark


2.4.1 - Set Terminal

    - Go to Settings//Default Applications//Utilities
    - Terminal Emulator : Xfce Terminal



2.5 - Install and Configure Firefox ESR:

sudo apt install firefox-esr


2.5.1 - Firefox Dark Mode:

      - Go to Settings > General > Manage Colors...
      - Set Text: White
      - Set Background: dark grey
      - Set Unvisited Links: White
      - Set Visited Links: light gray
      - Choose "Always"
       
        
2.5.2 - Tweaks Firefox ESR

https://github.com/yokoffing/Betterfox/tree/esr128
   
   
   
   
3 - PERFORMANCE TUNING



3.1 - Disable Suspend :

sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target



3.2 - Create and Configure xset_noblank Script :

sudo nano /home/$USER/xset_noblank.sh

- Add the following lines:

#!/bin/bash
xset s noblank s noexpose
xset dpms 0 0 0
xset s off
xset -dpms



3.3 - Make script executable :

sudo chmod +x /home/$USER/xset_noblank.sh



3.4 - Add to autostart :
     
    - Go to Applications > Settings > Session and Startup > Application Autostart
    - Click "+ Add"
    - Set Name: xset_noblank
    - Set Command: /home/$USER/xset_noblank.sh
    - Uncheck "screen-locker"

REBOOT



3.5 - Install cpufrequtils and sysfsutils:

sudo apt install -y cpufrequtils sysfsutils


3.5.1 - Enable Services

sudo systemctl enable cpufrequtils
sudo systemctl enable sysfsutils



3.6 - Disable Core Dumps : (Prevents large core dump files from filling up disk space)

sudo nano /etc/sysctl.d/50-coredump.conf

- Add the following line:

kernel.core_pattern=|/bin/false



3.7 - Enable the service :

sudo sysctl -p /etc/sysctl.d/50-coredump.conf



3.8 - Configure CPU Governor and Frequencies : ( https://wiki.debian.org/CpuFrequencyScaling )

sudo nano /etc/init.d/cpufrequtils # it's better to use "/etc/default/cpufrequtils" if possible

- Replace the following lines:

ENABLE="true"
GOVERNOR="ondemand"
MAX_SPEED="4400000" (where max speed is show in terminal when type : cpufreq-info)
MIN_SPEED="3500000" (~ half of max speed)



3.9 - Sysfs Tuning :

sudo nano /etc/sysfs.conf

- Add the following lines:

devices/system/cpu/cpufreq/ondemand/up_threshold = 99
devices/system/cpu/cpufreq/ondemand/sampling_down_factor = 6
devices/system/cpu/cpufreq/ondemand/sampling_rate = 20000000
vm.swappiness = 10
vm.dirty_background_ratio = 5
vm.dirty_ratio = 10
vm.vfs_cache_pressure = 50
module/snd_hda_intel/parameters/power_save = 0
module/snd_hda_intel/parameters/power_save_controller = N



3.10 - Grub Configuration :

-  Disable energy savings and security

   - Warning ! Disable security ( mitigations=off kernel.randomize_va_space=0 ) is not recommanded ( https://wiki.archlinux.org/title/Improving_performance )
   
sudo nano /etc/default/grub

- Replace this line : 
	
GRUB_CMDLINE_LINUX_DEFAULT="processor.ignore_ppc=1 intel_pstate=passive intel_idle.max_cstate=0 idle=poll nosmt=force pcie_aspm=off mitigations=off kernel.randomize_va_space=0 ipv6.disable=1"



3.11 - Reload Grub

sudo grub-mkconfig -o /boot/grub/grub.cfg

REBOOT



3.12 - Check Temperatures and CPU Frequencies:

sudo sensors

sudo cpufreq-info




4 - PYTHON, NVIDIA, AND TWEAKS



4.1 - Install Python with pyenv : ( https://github.com/pyenv/pyenv )

sudo apt update

sudo apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl git libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

curl -fsSL https://pyenv.run | bash


4..1.2 - bashrc and profile path

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init - bash)"' >> ~/.bashrc

echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo '[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init - bash)"' >> ~/.bashrc

exec "$SHELL"


4.1.3 - Install Python

pyenv install 3.12.5

pyenv global 3.12.5



4.2 - Xorg Configuration :

sudo usermod -a -G input $USER
sudo nano /etc/X11/Xwrapper.config

- Add the following line:

allowed_users=anybody
needs_root_rights=yes

    - Caution: Setting allowed_users=anybody is a significant security risk. Avoid this unless absolutely necessary and understand the implications. Consider using allowed_users=console instead, which is generally safer.



4.3 - Install Nvidia Driver and CUDA :

sudo apt install -y linux-headers-amd64 nvidia-driver firmware-misc-nonfree

REBOOT  # Reboot after driver installation

sudo apt install -y nvidia-cuda-dev nvidia-cuda-toolkit

REBOOT  # Reboot after CUDA installation


4.3.1 - CUDA Rights and Group Configuration :

sudo groupadd cuda

sudo usermod -aG cuda $USER  # Add current user to cuda group


4.3.2 - Allow members of group cuda to execute nvidia-smi ( https://forums.developer.nvidia.com/t/allow-non-root-users-to-change-application-clocks/244709 )

sudo nano /etc/sudoers

Add the following line:
	
%cuda ALL=(ALL) NOPASSWD: /usr/bin/nvidia-smi

    - Caution : The provided sudoers modifications were overly permissive and potentially dangerous.



4.4 - Nvidia :


4.4.1 - Nvidia Xorg conf
 
- Add the following lines :

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


4.4.2 - Modprobe conf

options nvidia NVreg_UsePageAttributeTable=1
options nvidia NVreg_EnableMSI=1
options nvidia NVreg_EnableStreamMemOPs=1
options nvidia NVreg_RegistryDwords="PerfLevelSrc=0x2222"
options nvidia NVreg_RegistryDwords="OverrideMaxPerf=0x1"
options nvidia-drm modeset=1
options nvidia-drm fbdev=1


4.4.3 - Profiling to user :

sudo nano /etc/modprobe.d/nvidia-options.conf

# Uncomment (remove the #) this line :

options nvidia-current NVreg_RestrictProfilingToAdminUsers=0


4.4.4 - Update Initramfs:

sudo update-initramfs -u

REBOOT



4.5 - Nvidia Patch (removes restriction on maximum number of simultaneous NVENC video encoding sessions)

git clone https://github.com/keylase/nvidia-patch.git
cd /home/smartlink/nvidia-patch/
sudo bash ./patch.sh



4.6 - Overclocking


4.6.1 - Benchmark (to verify each times you increase offset values, first with graphics clocks (the most important by far) and second with Memory Transfer Rate) 

- Go to  https://benchmark.unigine.com/heaven  and download heaven free download linux

cd Downloads
sudo chmod +x Unigine_Heaven-4.0.run
sudo ./Unigine_Heaven-4.0.run
cd Unigine_Heaven-4.0
./heaven

- Run the benchmark, you can check Memory Transfer Rate, graphics clocks, the temperature 

cd Downloads/Unigine_Heaven-4.0/heaven
./heaven


4.6.2 - Overclock

- Close alls programs and proceed with extreme caution, EACH TIMES YOU INCREASE offset values (XXX), run BENCHMARK. The overclocking values MUST be determined carefully and incrementally, not by blindly adding large offsets.


-  Max perf

sudo nvidia-settings -a '[gpu:0]/GpuPowerMizerMode=1'


- Set fan control

sudo nvidia-settings -a '[gpu:0]/GPUFanControlState=1'
sudo nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=100' # mean 100% speed fan


- Set power limit to maximum

sudo nvidia-smi --power-limit=XXX #  Where XXX is value for power-limit  (exp:200) " sudo nvidia-smi -q " to find GPU Max Power Limit
 

- Graphics overclocking (WARNING !! INCREASE BY 10 BY 10 STARTING FROM 0 and run BENCHMARK each times you increase)

sudo nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffsetAllPerformanceLevels=XXX' #  Where XXX is the value added to the maximum Memory clocks (exp:120 = +120MHZ)


- Memory overclocking  (increase by 100 by 100 starting from 0 and run benchmark each times you increase)

sudo nvidia-settings -a '[gpu:0]/GPUOffsetAllPerformanceLevels=XXXX' #  Where XXXX is the value added to the maximum Memory clocks (exp:1000 = +1000MHz) 


4.6.3 - Make changes permanents :

sudo nano /home/smartlink/GpuTweaks.sh

- Add the following lines :

#!/bin/bash
#
# Set power limit to maximum
sudo nvidia-smi --power-limit=XXX #  Where XXX is value for power-limit
#
# Set maximum performance mode
sudo nvidia-settings -a '[gpu:0]/GpuPowerMizerMode=1'
#
# Set fan control
sudo nvidia-settings -a '[gpu:0]/GPUFanControlState=1'
sudo nvidia-settings -a '[fan:0]/GPUTargetFanSpeed=100'
#
# Graphics overclocking
sudo nvidia-settings -a '[gpu:0]/GPUGraphicsClockOffsetAllPerformanceLevels=XXX' #  Where XXX is the value added to the maximum graphics clocks
#
# Memory overclocking
sudo nvidia-settings -a '[gpu:0]/GPUOffsetAllPerformanceLevels=XXXX' #  Where XXXX is the value added to the maximum Memory clocks

REBOOT



4.7 - Install FFmpeg :

sudo apt install libffmpeg-nvenc-dev
sudo apt install ffmpeg




5- FILESYSTEM, BOOT, AND SERVICE CONFIGURATIONS



5.1 - Set noatime and Tweak fstab for Ext4 : ( https://wiki.archlinux.org/title/Ext4 )

sudo nano /etc/fstab

    - Add options to the appropriate mount point. The provided UUID should be replaced with a placeholder and instructions to find the correct UUID.
    Example:

UUID=YOUR_ROOT_PARTITION_UUID / ext4 noatime,barrier=0,errors=remount-ro 0 1

    - Use lsblk -f to find the UUID of your root partition. The barrier=0 option is generally not recommended anymore for modern filesystems and hardware. The /boot/efi line should be left as is if you have an EFI system.

sudo systemctl daemon-reload



5.2 - Disable Journaling on ext4 : : ( https://wiki.archlinux.org/title/Ext4 )

    - Caution: Disabling journaling can lead to data corruption in case of unexpected shutdowns or power failures. Be extremely careful when using dd with a device as the output. Double-check the device name (/dev/sdX) as writing to the wrong device can lead to data loss

https://github.com/rescuezilla/rescuezilla/releases/download/2.5.1/rescuezilla-2.5.1-64bit.noble.iso


5.2.1 - Find USB device name :

lsblk


5.2.2 - Unmount USB

sudo umount /dev/sdX # Where sdX is your USB


5.2.3 - Create the disk image with dd :

sudo dd if=~/Downloads /rescuezilla-2.5.1-64bit.noble.iso of=/dev/sdX bs=1M status=progress


5.2.4 - Boot on USB and backup

Insert another USB or have another Hard Drive
Backup all your Debian Disk (exp:nvme0n1) on new USB/HDD/SSD


5.2.5 - Disable journalling on Ext4 

    - Caution: This can lead to data loss ! Disabling journaling increases performance but risks data corruption if the system crashes

lsblk # Identify your Debian partition root / (exp:nvme0n1p2)
tune2fs -O ^has_journal /dev/nvme0n1p2 # Where "nvme0n1p2" is your ROOT PARTITION !



5.3 - Autologin Lightdm : ( https://wiki.archlinux.org/title/LightDM )

sudo nano /etc/lightdm/lightdm.conf

        - Security Considerations: Enabling autologin is a security risk. Anyone with physical access to your computer will be logged in automatically. Consider the implications before enabling this.
        
- Replace the following lines :

[LightDM]
logind-check-graphical=true
[Seat:*]
autologin-user=$USER
autologin-user-timeout=0



5.4 - PulseAudio Configuration : ( https://wiki.archlinux.org/title/PulseAudio/Troubleshooting )

sudo nano /etc/pulse/daemon.conf

- Uncomment (remove the ;) : 

flat-volumes = no # relative volumes can be enabled by disabling flat volumes
avoid-resampling = yes
default-sample-rate = 48000



5.5 - Disable idle

sudo nano /etc/pulse/system.pa

- Add and comment this lines :

load-module module-udev-detect tsched=0 # Disable system-timer based model
# load-module module-suspend-on-idle # Disable idle



5.6 - Disable Services

sudo systemctl disable cron.service #  Runs scheduled tasks. Disable if you don't have any crucial cron jobs.
sudo systemctl disable avahi-daemon.service #  Used for network service discovery (mDNS/DNS-SD). Safe to disable if you don't need it.
sudo systemctl disable e2scrub_reap.service # Related to ext2/3/4 filesystem checking. Can be disabled unless you have specific filesystem integrity concerns.
sudo systemctl disable logrotate.service # Rotates log files.
sudo systemctl disable logrotate.timer 
sudo systemctl disable systemd-networkd.service # If you're using a static network configuration, you can disable this.
sudo systemctl disable systemd-timesyncd.service # Network time synchronization.
sudo systemctl disable upower.service # Power management service. Safe to disable unless you need battery monitoring or power management features.
sudo systemctl disable apt-daily-upgrade.service # Automatic package updates. 
sudo systemctl disable apt-daily.timer #  Daily apt operations.
sudo systemctl disable packagekit.service #  Software management service.
sudo systemctl disable nftables.service # If you're not using nftables for firewalling, you can disable it.



