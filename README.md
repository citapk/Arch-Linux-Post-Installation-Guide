# Çıt's Arch Linux Post-Installation Guide - Thinkpad T490
Hello! I wrote this guide for **[Thinkpad T490](https://psref.lenovo.com/syspool/Sys/PDF/ThinkPad/ThinkPad_T490/ThinkPad_T490_Spec.PDF) and [XFCE](https://xfce.org/) specifically**, however you can still follow **almost** all the steps if you don't have the same computer as I have. So, let's begin!

## Install AUR Helper
Normally, everyone prefers [yay](https://github.com/Jguer/yay) but I prefer [paru](https://github.com/Morganamilo/paru) since it is written in [rust](https://www.rust-lang.org/).
### Install Paru AUR Helper
`sudo pacman -S --needed base-devel git && git clone https://aur.archlinux.org/paru.git && cd paru && makepkg -si`
## Install CachyOS Repositories and BORE Kernel (Optional)
[CachyOS](https://cachyos.org/) is a **performance focused Arch-based** Linux distribution. If you would like an **out of the box experience** (it can be considered out of the box compared to Arch) and need the best performance possible, I would simply install CachyOS. However, I just want to use their [greatly optimized repositories and kernel](https://github.com/CachyOS/linux-cachyos#cachyos-repositories) while configuring every other thing about my system on my own. **Be careful, do not use CachyOS kernel if you need good battery life!** However, you can still install the kernel and use it while performing tasks with high load such as gaming and switch to default **Linux** or **Linux-LTS** kernel when you need more battery life.
### Install
`wget https://mirror.cachyos.org/cachyos-repo.tar.xz && tar xvf cachyos-repo.tar.xz && cd cachyos-repo && sudo ./cachyos-repo.sh && sudo pacman -Sy && sudo pacman -S linux-cachyos-bore`
## GRUB Configuration
**Location of GRUB**: `sudo nano /etc/default/grub`
```
GRUB_CMDLINE_LINUX_DEFAULT="mitigations=off"
GRUB_DEFAULT=saved
GRUB_SAVEDEFAULT=true
GRUB_DISABLE_SUBMENU=y
```
### Information About GRUB Configuration
- **You have to execute this command in terminal to apply changes**: `sudo grub-mkconfig -o /boot/grub/grub.cfg`
- **CPU mitigations reduce performance by 30% on these CPUs:**
  - **AMD**: `Zen 1, Zen 1+, Zen 2`
  - **Intel**: `6th, 7th and 8th Generation`
  - This is why you might want to disable mitigations via `mitigations=off` kernel parameter especially if you're gaming. However, disabled mitigations might **introduce security risks**. If you care about **security** more than **performance**, do **not** disable mitigations.
- If you will be using **only one kernel** that you installed while installing Arch, you don't have to change the stats of `GRUB_DEFAULT, GRUB_SAVEDEFAULT` and `GRUB_DISABLE_SUBMENU` because we are changing their stats to be able to switch between kernels easily.
## Intel iGPU Configuration For [X11/Xorg](https://www.x.org/wiki/) (If you're using [Wayland](https://wayland.freedesktop.org/), you can skip this step)
- Some projects such as [Debian](https://www.phoronix.com/news/Ubuntu-Debian-Abandon-Intel-DDX), [Fedora](https://www.phoronix.com/news/Fedora-Xorg-Intel-DDX-Switch), [KDE](https://community.kde.org/Plasma/5.9_Errata#Intel_GPUs) and [Mozilla](https://bugzilla.mozilla.org/show_bug.cgi?id=1710400) suggest **modesetting** driver instead of using [xf86-video-intel](https://archlinux.org/packages/?name=xf86-video-intel) with **intel** driver since intel driver is known to have problems such as freezing the system and granting poor performance. However, even though it is better, using **modesetting** driver can also cause issues such as **screen tearing** but we'll be using [Picom](https://github.com/yshui/picom) with vsync on, so screen tearing won't be an issue as long as Picom is running.
- **Location of Intel Configuration File**: `sudo nano /etc/X11/xorg.conf.d/20-intel.conf`
```
Section "Device"
Identifier "Intel Graphics"
Driver "modesetting"
Option "Backlight" "intel_backlight"
Option "AccelMethod" "glamor"
Option "TearFree" "false"
Option "RenderAccel" "true"
Option "SwapbuffersWait" "false"
Option "DRI" "2"
Option "FramebufferCompression" "false"
Option "TripleBuffer" "false"
Option "Shadow" "false"
Option "LinearFramebuffer" "true"
Option "RelaxedFencing" "false"
Option "BufferCache" "true"
EndSection
```
## Install Necessary Packages
`sudo pacman -S unrar unzip intel-ucode ufw tlp fwupd fastfetch alsa-tools easyeffects vlc noto-fonts-cjk noto-fonts-emoji && sudo systemctl enable --now tlp.service && sudo systemctl enable --now ufw.service`
- You **don't** have to install these packages:
  - **intel-ucode**: if you don't have an Intel CPU
  - **ufw**: if you don't want to enable firewall
  - **tlp**: if you don't want to have better battery life. Use **power-profiles-daemon** if you only want to have high performance
  - **fwupd**: if your computer is not supported by fwupd for firmware updates
## Install Necessary Packages - XFCE
`sudo pacman -S pavucontrol xarchiver mousepad capitaine-cursors picom && paru -S xfce4-panel-profiles xfce4-docklike-plugin flat-remix-gtk`
## Install Other Packages - Gaming and Utilities (Optional, these are what I use personally)
`sudo pacman -S cachyos-gaming-meta gamemode lib32-gamemode protonup-qt vesktop heroic-games-launcher prismlauncher joplin-desktop spectacle obs-studio onlyoffice shotcut okular && paru -S nomacs ptyxis spotify zoom`
### For Gamemode
- Download the .ini file from [the link](https://github.com/FeralInteractive/gamemode/blob/master/example/gamemode.ini) and move it to the correct directory via this command:
  - `cd Downloads && sudo mv gamemode.ini /etc/`
## TLP Configuration - Power Management For Laptops
- You can use [TLP-UI](https://aur.archlinux.org/packages/tlpui) to configure [TLP](https://linrunner.de/tlp/index.html) via a graphical interface but I noticed it doesn't uncheck some options which prevents them from functioning, that's why it is safer to apply them manually. Also, again, **do not use TLP if you don't need battery life, because we are using TLP for the best battery life on battery mode and to switch between performance/power saving modes for our laptop, using [power-profiles-daemon](https://github.com/Rongronggg9/power-profiles-daemon) is easier and needs no configuration at all!!!**
- **Location**: `sudo nano /etc/tlp.conf` (**make sure unchecking each option**)
```
TLP_ENABLE=1
TLP_DEFAULT_MODE=BAT
CPU_DRIVER_OPMODE_ON_AC=active
CPU_DRIVER_OPMODE_ON_BAT=active
CPU_SCALING_GOVERNOR_ON_AC=performance
CPU_SCALING_GOVERNOR_ON_BAT=powersave
CPU_ENERGY_PERF_POLICY_ON_AC=performance
CPU_ENERGY_PERF_POLICY_ON_BAT=power
CPU_MIN_PERF_ON_AC=0
CPU_MAX_PERF_ON_AC=100
CPU_MIN_PERF_ON_BAT=0
CPU_MAX_PERF_ON_BAT=50
CPU_BOOST_ON_AC=1 (DON'T USE THIS OPTION IF YOUR CPU DOES NOT SUPPORT BOOST)
CPU_BOOST_ON_BAT=0 (DON'T USE THIS OPTION IF YOUR CPU DOES NOT SUPPORT BOOST)
CPU_HWP_DYN_BOOST_ON_AC=1 (DON'T USE THIS OPTION IF YOUR CPU DOES NOT SUPPORT DYNAMIC BOOST)
CPU_HWP_DYN_BOOST_ON_BAT=0 (DON'T USE THIS OPTION IF YOUR CPU DOES NOT SUPPORT DYNAMIC BOOST)
NMI_WATCHDOG=0
PLATFORM_PROFILE_ON_AC=performance
PLATFORM_PROFILE_ON_BAT=low-power
AHCI_RUNTIME_PM_ON_AC=on
AHCI_RUNTIME_PM_ON_BAT=auto
WIFI_PWR_ON_AC=off
WIFI_PWR_ON_BAT=on
WOL_DISABLE=Y
SOUND_POWER_SAVE_ON_AC=0
SOUND_POWER_SAVE_ON_BAT=1
PCIE_ASPM_ON_AC=performance
PCIE_ASPM_ON_BAT=powersupersave
RUNTIME_PM_ON_AC=on
RUNTIME_PM_ON_BAT=auto
START_CHARGE_THRESH_BAT0=85 (USE THIS OPTION ONLY IF YOUR LAPTOP SUPPORTS BATTERY THRESHOLDS AND IF YOU WANT TO USE THIS FEATURE*)
STOP_CHARGE_THRESH_BAT0=90 (USE THIS OPTION ONLY IF YOUR LAPTOP SUPPORTS BATTERY THRESHOLDS AND IF YOU WANT TO USE THIS FEATURE)
NATACPI_ENABLE=1 (Battery care driver for all supported laptops)
TPACPI_ENABLE=1 (Battery care driver for Thinkpads only)
TPSMAPI_ENABLE=1 (Battery care driver for Thinkpads only)
```
### Note About Battery Care Drivers
You can simply enable `NATACPI, TPACPI` and `TPSMAPI` all at once, TLP will detect which one is compatible for your laptop and enable one of them automatically.
## ZRAM Size Increase (Optional) 
- For 16 GB RAM, Arch Linux dedicates 4 GB ZRAM which is not enough for me. That's why I increase it to 8 GB. You can skip this step if you don't know what you're doing.
- **Location**: `sudo nano /etc/systemd/zram-generator.conf`
```
[zram0]
zram-size = 8192
```
- `sudo systemctl restart systemd-zram-setup@zram0.service`
## Configure Fish && Fastfetch (Optional)
- If you would like your terminal to predict what you are going to type with colorful letters, you might want to use [Fish](https://fishshell.com/) for your terminal.
  - `sudo pacman -S fish`
  - `chsh -s /usr/bin/fish` (**you have to reboot after this command for next commands to work**)
```
  function fish_greeting
  fastfetch
  end
```
  - `funcsave fish_greeting`
## Picom Configuration - XFCE
- After installing [Picom](https://github.com/yshui/picom), you have to **disable XFCE's default compositor** via following these steps:
  - Settings Manager - Window Manager Tweaks - Compositor - Enable Display Compositing (uncheck the box)
- After these steps, execute these commands in terminal:
  - **Location**: `sudo nano ~/.config/picom.conf`
```
backend = "glx";
glx-no-stencil = true;
fading = true;
vsync = true;
```
- And reboot your system to apply changes. You can set keybinds to disable and enable Picom since you might want to disable compositor while gaming to gain better performance. You can set keybinds using these steps:
  - **To Disable Picom**: `Settings Manager` - `Keyboard` - `Application Shortcuts` - `Add` - `Command`: **killall picom** - `Command Shortcut`: **Your preferred key combination**
  - **To Enable Picom**: `Settings Manager` - `Keyboard` - `Application Shortcuts` - `Add` - `Command`: **picom** - `Command Shortcut`: **Your preferred key combination**
## SDDM For XFCE
- [SDDM](https://github.com/sddm/sddm) looks better than [LightDM](https://github.com/canonical/lightdm), so why wouldn't we use it instead? :) I'm going to be using [Catppuccin SDDM](https://github.com/catppuccin/sddm/releases).
- **Needed Packages**: `sudo pacman -S sddm qt6-svg qt6-declarative`
- **Disable LightDM and enable SDDM**: `sudo systemctl disable lightdm.service && sudo systemctl enable sddm.service`
### Add Catppuccin Theme
**Extract .zip file on your downloads directory execute this command in terminal**: `cd Downloads && sudo cp -r catppuccin-frappe /usr/share/sddm/themes/` (**You can change catppuccin-frappe if you installed something else**)
### Let's Edit SDDM
- **Location**: `sudo nano /etc/sddm.conf.d/sddm.conf`
```
[Theme]
Current=catppuccin-frappe (for example if you installed catppuccin-mocha, type catppuccin-mocha instead)
```
- And reboot your computer to apply changes.
## XFCE Ricing
- After applying your preferred (or the ones I installed) desktop and icon themes, you might want to make rices. However, if you would like an already made rice, here is [mine](https://drive.google.com/file/d/1ef-N87La8UkVuTtwFG6oRucCx9G8ujA3/view?usp=sharing)
- If you preferred using my panel preset, you can apply it following these steps:
  - `Settings Manager` - `Panel Profiles` - `Import` - `Preset File` - `Çıt` - `Apply`
## Dolby Atmos Setup (EasyEffects) - [Reference](https://www.reddit.com/r/thinkpad/comments/q5pt38/x1_extreme_gen_4_dolby_atmos_setup_for_linux/)
- After launching [EasyEffects](https://github.com/wwmm/easyeffects) and setting it to launch at boot from app settings, download [this .zip file](https://www.mediafire.com/file/qt9znutry7fgzk7/dolby.zip/file) for [Dolby Atmos](https://www.dolby.com/technologies/dolby-atmos/) presets.
- After downloading and extracting the .zip file, move the folder to a location you will remember the path of.
- Next, in EasyEffects, open `Effects` page and click on `Add Effect`. Then, add a `Convolver` and click on convolver.
- Click on `Impulses` and then `Import Impulse`, now add all of the .irs files you downloaded.
- Last, on `Impulses` page, it will be enough to load one of the presets (you can switch between presets).
