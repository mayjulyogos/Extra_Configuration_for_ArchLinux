# Extra configurations for Arch Linux

<br>

## YAY and Pacman package managers

### How to setup yay packages
```bash
sudo pacman -S --needed git base-devel
git clone [https://aur.archlinux.org/yay.git](https://aur.archlinux.org/yay.git)
cd yay
makepkg -si
cd .. && rm -rf yay
```

### Boost Compiling Speed for Qdirstat installation and further installation with YAY
```bash
sudo nano /etc/makepkg.conf
# Press Crtl + K or Ctrl + f to search for MAKEFLAGS find this line #MAKEFLAGS="-j2"
# Change it to this line
MAKEFLAGS="-j$(nproc)"
# Save Ctrl + o and then Ctrl + x
```

### How to manage pacman and yay packages
```bash
# Show system package statistics and status
sudo pacman -Qi
# or
yay -Ps

# Install packages
sudo pacman -S <package>
# or
yay -S <package>

# Clean up unnecessary dependencies
sudo pacman -Sc
# or
yay -Yc

# Update packages
sudo pacman -Syu
# or
yay -Syu

# Remove packages
sudo pacman -Rs <package>  # Official repos
# or
yay -Rs <package>          # Official repos or AUR

# Remove orphan packages (dependencies not longer needed by any app)
pacman -Qtdq | xargs -r sudo pacman -Rns
# or use if
if [ -n "$(pacman -Qtdq)" ]; then sudo pacman -Rns $(pacman -Qtdq); else echo "No orphan packages found."; fi
```

### Pacman or YAY Packages Uninstallation
```bash
# Remove package
sudo pacman -Rns package_name

# Remove orphaned dependencies
sudo pacman -Rns $(pacman -Qtdq)

# Clean package cache (optional)
sudo pacman -Sc

# Find leftover user files
find ~/.config ~/.local ~/.cache ~/.local/share ~/.local/state \
    -iname "*package_name*"

# Remove any leftover directories you find
rm -rf ~/.config/package_name
rm -rf ~/.cache/package_name
rm -rf ~/.local/share/package_name
rm -rf ~/.local/state/package_name

# Verify removal
pacman -Q package_name
command -v package_name
```

<br>

---

<br>

## Install flatpak
```bash
sudo pacman -S flatpak
flatpak remote-add --if-not-exists flathub \
https://dl.flathub.org/repo/flathub.flatpakrepo

# restart 
```

<br>

---

<br>

## Add Reflector
* Reflector is a python script/tool built specifically for Arch Linux that manages and optimizes your system's package download servers
* Fetches the Master List: It contacts the official Arch Linux server to get a real-time list of every active mirror worldwide
* Filters Out Bad Servers: It automatically removes mirrors that are broken, offline, out-of-date, or insecure (non-HTTPS)
* Tests Speed & Latency: It runs a live connection speed test against the working mirrors
* Overwrites your Mirrorlist: It takes the fastest, healthiest servers it found and overwrites your main mirror configuration file (/etc/pacman.d/mirrorlist), placing the fastest server at the very top
```bash
sudo pacman -S reflector rsync curl

# Replace or edit the file to match these optimized settings for your region:
# Save and exit (Ctrl+O, Enter, then Ctrl+X)
sudo nano /etc/xdg/reflector/reflector.conf
--save /etc/pacman.d/mirrorlist
--protocol https
--country 'Malaysia,Singapore'
--latest 20
--sort rate
--connection-timeout 15

# Check the mirrorlist file to ensure it shows a new timestamp and a list of local/regional servers:
cat /etc/pacman.d/mirrorlist
# Test your download speed by forcing Pacman to refresh its databases using the new mirrors:
sudo pacman -Syyu
c
# Enable weekly automation
sudo systemctl enable --now reflector.timer
```

<br>

---

<br>

## Zsh
```bash
# Installation
sudo pacman -S zsh zsh-completions

# Set Zsh as your default shell
sudo chsh -s /usr/bin/zsh arch

# Log out of your Arch Linux user session completely or restart your computer to apply changes

# Install oh-my-zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Install standard Nerd Fonts package for using advanced theme
sudo pacman -S ttf-nerd-fonts-symbols-common
```

<br>

---

<br>


## Firewalld & GNOME Integration
```bash
su

# Install firewalld and the graphical configuration tool
pacman -S firewalld network-manager-applet

# Enable and start the service
systemctl enable --now firewalld

# Install AppArmor and the default profiles
pacman -S apparmor

# Enable the systemd service
systemctl enable apparmor.service

nano /etc/default/grub
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet rhgb rd.driver.blacklist=nouveau,nova_core modprobe.blacklist=nouveau,nova_core nvidia-drm.modeset=1 nvidia-drm.fbdev=1 acpi_backlight=native apparmor=1 lsm=landlock,lockdown,yama,apparmor,bpf"
# Ctrl + O, Enter, then Ctrl + X

grub-mkconfig -o /boot/grub/grub.cfg

pacman -S firewall-config

# Automatically inject nvidia modules into mkinitcpio.conf for early loading (KMS)
sed -i 's/^MODULES=(/MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm /' /etc/mkinitcpio.conf

# Regenerate the initramfs
mkinitcpio -P

# Regenerate grub one more time to apply the command line arguments
grub-mkconfig -o /boot/grub/grub.cfg
```

<br>

---

<br>

##  Configure Audio and Bluetooth
```bash
su

pacman -Syu
pacman -S \
pulseaudio \
pipewire \
wireplumber \
pipewire-alsa \
pipewire-pulse \
pipewire-jack \
alsa-utils \
sof-firmware \
alsa-firmware

pacman -S bluez bluez-utils

systemctl enable --now bluetooth.service

reboot
```

<br>

---

<br>

## GPU Drivers Installation
* You may install the GPU Driver by following the GPU you use so you don't need to install intel GPU if you are using AMD GPU
```bash
sudo pacman -S nvidia-utils nvidia-settings nvidia-open-dkms linux-headers libva-nvidia-driver libva-utils # nvidia drivers

sudo pacman -S linux-firmware mesa vulkan-intel intel-media-driver libva-utils intel-gpu-tools # intel drivers

sudo pacman -S mesa lib32-mesa xf86-video-amdgpu vulkan-radeon lib32-vulkan-radeon # AMD GPU
```

<br>

---

<br>

## CUDA
```bash
sudo pacman -S cuda
```

<br>

---

<br>

## Fctix 5
- For Chinese Input
```bash
sudo pacman -S fcitx5 fcitx5-configtool fcitx5-chinese-addons fcitx5-gtk fcitx5-qt
```
- Add environment variables to `hyprland.lua`
- Fcitx5 prefers using `Wayland's native input method protocol` instead of forcing applications through these `older input modules`. Having them set can occasionally cause `input glitches` or `warning popups`.
- So it is recommended to commend out these following environments variable
```bash
-------------------------------
---- ENVIRONMENT VARIABLES ----
-------------------------------

hl.env("XCURSOR_SIZE", "24")
hl.env("HYPRCURSOR_SIZE", "24")

-- Fcitx5 Framework Input Rules
-- hl.env("IMMODULE", "fcitx")
-- hl.env("XMODIFIERS", "@im=fcitx")
-- hl.env("GTK_IM_MODULE", "fcitx")
-- hl.env("QT_IM_MODULE", "fcitx")
-- hl.env("SDL_IM_MODULE", "fcitx")
-- hl.env("GLFW_IM_MODULE", "ibus") -- Fallback for specific GLFW windows
```
- Add Autostart Fctix5 Deamon
```bash
-------------------
---- AUTOSTART ----
-------------------

-- See https://wiki.hypr.land/configuring/core/autostart/

-- Autostart necessary processes (like notifications daemons, status bars, etc.)
-- Or execute your favorite apps at launch like this:
--
hl.on("hyprland.start", function () 
   hl.exec_cmd("nm-applet")
   hl.exec_cmd("/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1")
   hl.exec_cmd("hyprpaper")
   hl.exec_cmd("hyprpm reload")
   hl.exec_cmd("elephant")
   hl.exec_cmd("fcitx5 -d")
end)
```

### Steps to add Chinese input
- Launch Fcitx 5 Configuration
- Search for `Chinese Pinyin` using the Search bar under `Available Input Methods:`
- Add the Pinyin input method by pressing the `left arrow` button
- Now the `Pinyin` will shows up at the left side panel bellow your `English` Input
- You may press on the settings button to change how many candidates will shows each time
- You may also navigate to `Addons`
- Press the `Configure` button next of `Classic User Interface`
- You may change your `Font Type and Font Size`
- You may also change the theme to `dark or light`

<br>

---

<br>

## Install yay and CoolerControl
* Using CoolerControl you may also create your own cooling fan curve which your fan spin following by the temperature of your hardware.
```bash
sudo pacman -S qt6-wayland
sudo pacman -S liquidctl
sudo pacman -S coolercontrol
sudo systemctl enable --now coolercontrold

# Open it in your web browser
# http://localhost:11987
```

<br>

---

<br>

## Auto-cpufreq
```bash
git clone https://github.com/AdnanHodzic/auto-cpufreq.git
cd auto-cpufreq
sudo ./auto-cpufreq-installer

systemctl enable --now auto-cpufreq 
sudo systemctl mask power-profiles-daemon.service
```

<br>

### Enable HWP Dynamic Boost
- Use a Startup Script (rc.local or systemd service)
#### 1. Create a new service file
```bash
sudo nano /etc/systemd/system/enable-hwp-dynamic-boost.service
```

#### 2. Paste the following configuration into the file
```bash
[Unit]
Description=Enable Intel HWP Dynamic Boost on Boot
After=multi-user.target

[Service]
Type=oneshot
ExecStart=/bin/sh -c 'echo 1 > /sys/devices/system/cpu/intel_pstate/hwp_dynamic_boost'
RemainAfterExit=true

[Install]
WantedBy=multi-user.target
```

#### 3. Save and exit (`Ctrl+O`, Enter, then `Ctrl+X`)

#### 4. Enable and start the service so it runs automatically on every boot:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now enable-hwp-dynamic-boost.service
```
#### 5. Verification
- It should now output `1`
```bash
cat /sys/devices/system/cpu/intel_pstate/hwp_dynamic_boost

sudo auto-cpufreq --stats
```

<br>

---

<br>

## YT-DLP
```bash
# Install tools and dependency
sudo pacman -S yt-dlp ffmpeg deno

# Verification
yt-dlp --version
ffmpeg -version
deno --version

# Example command lines
# Video and audio
yt-dlp -f "bestvideo+bestaudio/best" --merge-output-format mkv --audio-format flac -x "https://www.youtube.com/watch?v=R8MZbl-GRdY"

# Audio only
yt-dlp -f "bestaudio" -x --audio-format flac "https://www.youtube.com/watch?v=R8MZbl-GRdY"

# Old TVs Compatibility
yt-dlp --no-playlist -f "bv*[height<=1080][fps<=30]+ba/b[height<=1080][fps<=30]" --recode-video mp4 --postprocessor-args "ffmpeg:-c:v libx264 -preset fast -crf 22 -c:a aac -b:a 192k" "https://www.youtube.com/watch?v=pszihOrS21g"
```

<br>

---

<br>

## Davinci Resolve
```bash
# 1. Set up your AUR development folder and clone the repository
mkdir -p ~/AUR
cd ~/AUR
git clone https://aur.archlinux.org/davinci-resolve.git
cd davinci-resolve

# 2. Move your downloaded zip (regardless of version) into the build folder
mv ~/Documents/DaVinci_Resolve_*_Linux.zip .

# 3. Compile and install the package
# (makepkg will automatically look for the zip in this directory)
makepkg -si
```

<br>

---

<br>

## Virtual Machine Platform
```bash
# 1. Install all the necessary virtualization packages
sudo pacman -S --needed qemu-desktop virt-manager libvirt dnsmasq iptables-nft ebtables dmidecode

# 2. Enable and start the virtualization background daemon
sudo systemctl enable --now virtqemud.socket virtnetworkd.socket virtnodedevd.socket virtsecretd.socket virtstoraged.socket

# 3. Add your user to the libvirt group for passwordless VM management
sudo usermod -aG libvirt $USER

# 4. Restart firewalld so it detects and loads the new libvirt network zones
sudo systemctl restart firewalld

# 5. Start the default virtual network bridge for VM internet access
sudo virsh net-autostart default

# 6. Set the default virtual network to boot automatically on system startup
sudo virsh net-start default || true
```

### Excluding Virtual Machine for Arch Linux System snapshot
```bash
# 1. Stop libvirtd to release any locked image files
sudo systemctl stop libvirtd

# 2. Back up existing images and create a new subvolume
sudo mv /var/lib/libvirt/images /var/lib/libvirt/images.bak
sudo btrfs subvolume create /var/lib/libvirt/images

# 3. Move your files back and clean up
sudo mv /var/lib/libvirt/images.bak/* /var/lib/libvirt/images/
sudo rm -rf /var/lib/libvirt/images.bak

# 4. Restart libvirtd
sudo systemctl start libvirtd
```

<br>

---

<br>

## Steam installation and unistallation
* First time Installation
```bash
# 1. Enable multilib repository
sudo nano /etc/pacman.conf

# Ensure these lines are uncommented:
[multilib]
Include = /etc/pacman.d/mirrorlist

# 2. Update system and install Steam
sudo pacman -Syu
sudo pacman -S steam lib32-nvidia-utils lib32-libglvnd

# 3. Create a dedicated Btrfs subvolume for Steam
# Create Steam as a nested Btrfs subvolume.
# Its contents are not included when @home is snapshotted.

# Create the parent directory structure
mkdir -p ~/.local/share
# Create the dedicated Btrfs subvolume directly where Steam expects its files
sudo btrfs subvolume create ~/.local/share/Steam
# Fix ownership permission
sudo chown -R $USER:$USER ~/.local/share/Steam
# Verify subvolume creation
sudo btrfs subvolume list /

# launch steam
steam
```

<br>

- Exclude Existing Steam Installation from Btrfs Snapshots
```bash
# 1. Make sure Steam is completely closed
pkill -x steam 2>/dev/null || true

# 2. Prepare directories
mkdir -p ~/.local/share

# 3. If an existing standard directory exists, move it to backup
if [ -d ~/.local/share/Steam ] && [ ! -d ~/.local/share/Steam.bak ]; then
    mv ~/.local/share/Steam ~/.local/share/Steam.bak
fi

# 4. Create the dedicated Btrfs subvolume
# (If it already exists, this will safely do nothing or error out safely)
if ! btrfs subvolume show ~/.local/share/Steam &>/dev/null; then
    sudo btrfs subvolume create ~/.local/share/Steam
fi

# 5. CRITICAL FIX: Fix ownership BEFORE copying files
sudo chown -R $USER:$USER ~/.local/share/Steam

# 6. Restore existing Steam data from backup
if [ -d ~/.local/share/Steam.bak ]; then
    echo "Restoring Steam files from backup..."
    cp -a ~/.local/share/Steam.bak/. ~/.local/share/Steam/
    # Optional: remove backup once you verify everything works
    # rm -rf ~/.local/share/Steam.bak
fi

# 7. Verify and launch
echo "Subvolume status:"
btrfs subvolume show ~/.local/share/Steam
```

<br>

- GE-Proton Installation for games compatibility
```bash
cd ~/.local/share/Steam/compatibilitytools.d/

# Download GE-Proton
wget https://github.com/GloriousEggroll/proton-ge-custom/releases/download/GE-Proton11-11/GE-Proton11-11-x86_64.tar.gz

# Extract the tar.gz file
tar -xf GE-Proton11-11-x86_64.tar.gz

# Remove the extracted tar.gz file
rm GE-Proton11-11-x86_64.tar.gz

# FULLY SHUTDOWN Your Steam app
# RESTART Your Steam App 
# Now you should see GE-Proton... available under the compaitibility tool table
```

<br>

* Uninstallation
```bash
# Make sure Steam is completely closed
pkill -x steam 2>/dev/null || true

# 1. Remove Steam package
sudo pacman -Rns steam

# 2. Delete the dedicated Steam Btrfs subvolume
sudo btrfs subvolume delete ~/.local/share/Steam

# 3. Remove Steam configuration and cache
rm -rf ~/.steam
rm -rf ~/.config/steam

# 4. Verify removal
sudo btrfs subvolume list /
pacman -Q steam
# Expected:
# error: package 'steam' was not found
```

<br>

---

<br>

## Ghostty
```bash
sudo pacman -Syu ghostty

# 1. Create the ghostty configuration folder
mkdir -p ~/.config/ghostty

# 2. Open and create the config file as your normal user
nano ~/.config/ghostty/config



# --- Font ---
font-family = JetBrains Mono
font-size = 15
font-variation = wght=600

# --- Appearance & Window ---
foreground = #ffffff
background = #000000
selection-foreground = #000000
selection-background = #ffffff

# Liquid Glass Visuals
# background-opacity = 0.75
# background-blur-radius = 50

window-padding-x = 10
window-padding-y = 10
window-decoration = false
maximize = true

# --- Cursor ---
cursor-style = block
cursor-style-blink = true

# --- Mouse & Scroll ---
scrollback-limit = 100000000
mouse-hide-while-typing = true

# --- Keybindings ---
keybind = ctrl+d=new_split:right
keybind = ctrl+shift+d=new_split:down
keybind = ctrl+t=new_tab
keybind = ctrl+w=close_surface

# --- Enable xterm-256color ---
term = xterm-256color
```

<br>

---

<br>

# Btrfs structure
```text
Btrfs
│
├── @
│   └── /
│       ├── boot
│       │   ├── vmlinuz-linux
│       │   ├── initramfs-linux.img
│       │   └── efi/
│       ├── etc
│       ├── usr
│       ├── var
│       └── ...
│
├── @home
│   └── /home
│       └── arch
│           ├── Documents
│           ├── Downloads
│           ├── .config
│           └── .local
│               └── share
│                   └── Steam
│
├── @cache
├── @log
│
└── @snapshots
```

<br>

----

<br>

## Ecluding any files
```bash
# 1. Stop any service using the folder
sudo systemctl stop <service-name>

# 2. Rename the original folder to a temporary backup
sudo mv /path/to/folder /path/to/folder.bak

# 3. Create the new nested Btrfs subvolume
sudo btrfs subvolume create /path/to/folder

# 4. Copy the existing files back (preserving permissions)
sudo cp -a /path/to/folder.bak/. /path/to/folder/

# 5. Remove the backup and restart the service
sudo rm -rf /path/to/folder.bak
sudo systemctl start <service-name>
```

<br>

---

<br>

## How to resync time
```bash
sudo timedatectl set-ntp false && sudo timedatectl set-ntp true
```

<br>

---

<br>

## exFat support
* exFat is a modern file system created by Microsoft in 2006 to handle large files on flash drives and memory cards.
* No 4GB Limit: Unlike FAT32, it stores single files much larger than 4GB
* Cross-Platform: It works natively on both Windows and macOS computers
* Great for Flash Media: It is optimized for USB thumb drives and SD cards
* No Journaling: It lacks a recovery log, making it prone to data corruption if unplugged improperly
* No Security: It does not support file-level permissions or built-in encryption
```bash
sudo pacman -S exfatprogs
```
<br>

---

<br>

## Formatting USB
- For me I prefer exFAT which is supported on android, windows, mac and linux
- You may also change the partition format of the usb as you want
- Remember to format your partition as `MBR` instead of `GPT` for compatibility
```bash
# Identify the USB Drive
lsblk

# Unmount Any Active Partitions
sudo umount /dev/sda1

# Wipe old file signatures
sudo wipefs -a /dev/sda

# Create a new partition and layout
sudo fdisk /dev/sda
# g then Enter — Creates a clean GPT partition table.
# n then Enter — Creates a new partition (press Enter through the default prompts to use the full drive capacity).
# w then Enter — Writes the changes to the disk and exits.

# Format the USB Partition
sudo mkfs.exfat -n "MyUSB" /dev/sda1
```


<br>

---

<br>

## Easy Effects
```bash
# Install Easy Effects and required dependencies
sudo pacman -S --needed easyeffects lsp-plugins-lv2 calf zam-plugins-lv2 mda.lv2 libbs2b rnnoise
```

<br>

---

<br>

## Podman — Arch Linux GPU Container

### 1. Install Podman and NVIDIA Container Toolkit on the host site
```bash
sudo pacman -Syu podman nvidia-container-toolkit
```

<br>

### 2. Configure NVIDIA GPU Access
- Generate the NVIDIA Container Device Interface (CDI) configuration:
```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
```

- Verify the available GPU devices:
```bash
nvidia-ctk cdi list
```

- Expected output includes:
```text
nvidia.com/gpu=0
nvidia.com/gpu=all
```

- Ensure the host NVIDIA driver works before continuing:
```bash
nvidia-smi
```

<br>

### 3. Create Persistent Workspace Storage
- Create a Podman-managed volume for files that should persist independently of the container:
```bash
podman volume create arch-workspace
```

- Inspect the volume:
```bash
podman volume inspect arch-workspace
```

<br>

### 4. Download the Arch Linux Image
```bash
podman pull docker.io/library/archlinux:latest
```

<br>

### 5. Create and Start the Container
```bash
podman run -it \
  --name arch \
  --hostname arch \
  --pull=never \
  --device nvidia.com/gpu=all \
  --security-opt=no-new-privileges \
  --pids-limit=2048 \
  --shm-size=1g \
  --env LANG=C.UTF-8 \
  --publish 8080:8080 \
  --volume arch:/arch:rw \
  --workdir /arch \
  docker.io/library/archlinux:latest \
  /bin/bash
```

### Custom Container without default distro
- Install this following package on your `Host system`
```bash
sudo pacman -S arch-install-scripts
```
- Creat You custom `rootfs`
```bash
mkdir /tmp/custom-rootfs
sudo pacstrap -C /etc/pacman.conf /tmp/custom-rootfs base base-devel
```
- Compress your `rootfs` and bring it into `Podman` as a `local image`
```bash
# Create the tarball
sudo tar -C /tmp/custom-rootfs -c . -f /tmp/custom-rootfs.tar

# Import it into Podman
podman import /tmp/custom-rootfs.tar localhost/my-custom-rootfs:latest
```
- Run this following command line to create your podman contianer
```bash
podman run -it \
--name arch-custom \
--hostname arch \
--pull=never \
--device nvidia.com/gpu=all \
--security-opt=no-new-privileges \
--pids-limit=2048 \
--shm-size=1g \
--env LANG=C.UTF-8 \
--publish 8080:8080 \
--volume arch:/arch:rw \
--workdir /arch \
localhost/my-custom-arch:latest \
/bin/bash
```

<br>

#### Container configuration

| Option                                  | Purpose                                                            |
| --------------------------------------- | ------------------------------------------------------------------ |
| `--name arch`                           | Sets the container name                                            |
| `--hostname arch`                       | Sets the container hostname                                        |
| `--pull=never`                          | Uses the previously downloaded image                               |
| `--device nvidia.com/gpu=all`           | Exposes NVIDIA GPU devices through CDI                             |
| `--security-opt=no-new-privileges`      | Prevents privilege escalation through setuid/setgid executables    |
| `--pids-limit=2048`                     | Limits the number of processes                                     |
| `--shm-size=1g`                         | Sets shared memory to 1 GiB                                        |
| `--env LANG=C.UTF-8`                    | Sets the locale environment variable                               |
| `--volume arch:/arch:rw`                | Mounts persistent workspace storage                                |
| `--workdir /arch`                       | Starts the shell in `/arch`                                        |
| `--publish 8080:8080`                   | Exclude specific port which is accessible for both host and client |
| `/bin/bash`                             | Starts an interactive Bash shell                                   |

<br>

### 6. Initial Container Setup
- Run these commands inside the container:
```bash
pacman -Syu

pacman -S --needed \
  base-devel \
  git \
  nano \
  vim \
  less \
  python \
  python-pip \
  procps-ng \
  iproute2 \
  iputils \
  file \
  which \
  pciutils \
  cmake \
  ninja \
  aria2
```

- Verify the container and GPU:
```bash
cat /etc/os-release
whoami
nvidia-smi
```

- Note: The container shares the host kernel and NVIDIA kernel driver. Do not install another NVIDIA kernel driver inside the container.

- cuda installation
```bash
# Install CUDA
pacman -S cuda

# You may face file conflict because the files listed (`nvidia.icd` and the OpenCL library files) already exist within your container's filesystem.
# To bypass this conflict and force Pacman to overwrite the existing files, run the installation command with the --overwrite flag:
pacman -S cuda --overwrite '*'

# 1. Source the CUDA profile script
source /etc/profile.d/cuda.sh

# Alternatively, add the path directly if the script isn't found:
export PATH=/opt/cuda/bin:$PATH

# Verification
nvcc --version
```

<br>

### 7. Container Management
- Run these commands from the **host terminal**, not from inside the container.

#### Check container status
```bash
podman ps -a
```

- `podman ps` lists running containers, while `podman ps -a` also lists stopped containers.

#### Start and attach to an existing container
```bash
podman start -ai arch
```

#### Open another shell in a running container
```bash
podman exec -it arch /bin/bash
```

#### Stop the container
```bash
podman stop arch
```

#### Inspect the container
```bash
podman inspect arch
```

#### Inspect workspace storage
```bash
podman volume inspect arch
```

#### View container logs
```bash
podman logs arch
```

<br>

### 8. Persistence and Isolation
- **Container filesystem:** Installed packages and configuration persist when the container stops. Removing the container deletes its writable filesystem.
- **Workspace volume:** Files under `/workspace` persist independently of the container.
- **Host filesystem:** Your home directory is not mounted by this configuration.
- **Kernel:** The container shares the host Linux kernel; it does not boot an independent kernel.
- **GPU:** The NVIDIA GPU is shared with the host and other processes that can access it.

### To remove the container while preserving the workspace:
```bash
podman stop arch
podman rm arch
podman volume rm arch # remove workspace volume
```

- Do not remove the `arch-workspace` volume if you want to keep your workspace files.

<br>

---

<br>

## How to exclude podman from being snapshot
```bash
# 1. Stop the podman service and socket to release any locked files
sudo systemctl stop podman.service podman.socket

# 2. Move the existing containers directory to a temporary backup
sudo mv /home/arch/.local/share/containers /home/arch/.local/share/containers.bak

# 3. Create a new Btrfs subvolume where the directory used to be
sudo btrfs subvolume create /home/arch/.local/share/containers

# 4. Copy your existing Podman data back into the new subvolume
sudo cp -a /home/arch/.local/share/containers.bak/. /home/arch/.local/share/containers/

# 5. Clean up the backup folder
sudo rm -rf /home/arch/.local/share/containers.bak

# 6. Restart the podman service
sudo systemctl start podman.service

# Verification
# 1. Check the subvolume status
sudo btrfs subvolume show /home/arch/.local/share/containers

# 2. List all Btrfs subvolumes
# Confirm that /home/arch/.local/share/containers appears in your system's subvolume list:
sudo btrfs subvolume list /

# 3. Verify podman storage and status
podman system info
```

<br>

---

<br>

## How to prevent your laptop from being hibernated automatically
### 1. Edit the logind.conf File
- Open the systemd logind configuration file using a text editor with root privileges (such as `nano`):
```bash
sudo nano /etc/systemd/logind.conf
```

<br>

### 2. Modify Lid Switch Settings
- Look for the following lines in the file. You will likely need to remove the `#` symbol at the beginning of each line (to uncomment them) and change their values to `ignore`:
- `HandleLidSwitch`: Controls behavior when closing the lid on battery power.
- `HandleLidSwitchExternalPower`: Controls behavior when closing the lid while plugged in.
- `HandleLidSwitchDocked`: Controls behavior when connected to a docking station or external monitor.
- Save the file and exit the editor (in `nano`, press `Ctrl + O`, then Enter to save, and `Ctrl + X` to exit).
```text
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

<br>

### 3. Apply the Changes
- Restart the `systemd-logind` service to apply your new configuration immediately:
```bash
sudo systemctl restart systemd-logind
```

<br>

---

<br>

## Format USB
### 1. Check your drive and partitions
- Plug in your USB drive and run `lsblk` to confirm its partition name (in your case, `/dev/sda1`):
```bash
lsblk
```

<br>

### 2. Close the LUKS container
- If the partition was previously encrypted, ensure its container is closed so the device isn't busy:
- (Replace the long LUKS name with whatever matches your specific `lsblk` output if it differs).
```bash
sudo cryptsetup close luks-5bc4be54-9010-411d-9f3c-544f5ce14cde
```

<br>

### 3. Format the partition to exFAT with force
#### 1.Create an MBR Partition Table: Whole Disk Partitioning.

- Use `parted` to wipe the disk and create a new MBR (referred to as `msdos` in parted) partition table on the whole drive:
```bash
sudo parted /dev/sda --script mklabel msdos
```

- _Verification:_ Run `sudo parted /dev/sda print` and check that the "Partition Table" field reads `msdos`.

#### 2.Create a Primary Partition: Partition Creation.
- Create a primary partition that spans the entire drive and set it up:
```bash
sudo parted /dev/sda --script mkpart primary exfat 0% 100%
```

- _Verification:_ Run `lsblk /dev/sda` and confirm that `/dev/sda1` now exists.

##### 3.Format the Partition with exFAT: Filesystem Formatting.
- Now, run your original command to apply the exFAT filesystem to the new MBR partition:
```bash
sudo mkfs.exfat -F -n "Sandisk" /dev/sda1
```

- _Verification:_ The output will confirm the volume was successfully created and labeled "Sandisk".

<br>

---

<br>

## Font Installation
- To install fonts on Arch Linux, place the font files in the correct directory or install their package, then refresh the system font cache with `fc-cache -fv`.
### Option 1: Font Installation using pacman packages
```bash
# Search for available fonts
pacman -Ss ttf-

# Install a font package
sudo pacman -S ttf-noto ttf-liberation

# For popular programming fonts like JetBrains Mono with Nerd Fonts icons
sudo pacman -S ttf-jetbrains-mono-nerd

# Refresh the font cache to apply changes
fc-cache -f -v

# Verify the Installation
# You can check if your system successfully recognizes the newly installed font by listing available fonts matching a name
fc-list : family | grep "JetBrainsMono"
```

<br>

### Option 2: Install fonts manually (Custom or Downloaded fonts)
```bash
# For user only
# Create a local font directory if it doesn't exist, and place your font files there
mkdir -p ~/.local/share/fonts
cp /path/to/font.ttf ~/.local/share/fonts/

# System-wide (For all users)
# Place the font files inside /usr/share/fonts/ (you can create a custom subfolder)
sudo mkdir -p /usr/share/fonts/custom-fonts
sudo cp /path/to/font.ttf /usr/share/fonts/custom-fonts/

# Refresh the font cache to apply changes
fc-cache -f -v

# Verify the Installation
# You can check if your system successfully recognizes the newly installed font by listing available fonts matching a name
fc-list : family | grep "JetBrainsMono"
```

<br>

---

<br>

## Llama.cpp

<br>

### Install Dependencies
```bash
pacman -Syu

pacman -S --needed \
  base-devel \
  git \
  nano \
  vim \
  less \
  python \
  python-pip \
  procps-ng \
  iproute2 \
  iputils \
  file \
  which \
  pciutils \
  cmake \
  ninja \
  aria2
```

<br>

### Install CUDA (CUDA TOOL-KIT)
```bash
pacman -S cuda

# You may face file conflict because the files listed (`nvidia.icd` and the OpenCL library files) already exist within your container's filesystem.
# To bypass this conflict and force Pacman to overwrite the existing files, run the installation command with the --overwrite flag:
pacman -S cuda --overwrite '*'

# 1. Source the CUDA profile script
source /etc/profile.d/cuda.sh

# Alternatively, add the path directly if the script isn't found:
export PATH=/opt/cuda/bin:$PATH

# Verification
nvcc --version
```

<br>

### Git clone llama.cpp repository
```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
```

<br>

### Build llama.cpp using CUDA
```bash
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release --parallel $(nproc)
```

<br>

### Download models
- Models will be downloaded at `/root/.cache/huggingface/hub/blobs/.....`
- Models files name will be a bunch of letters and numbers such as `10cc391b403021dd11c614679d2fd92f611c3681d29e29651b717316965d61e1`
- Use `sudo mv 10cc391b403021dd11c614679d2fd92f611c3681d29e29651b717316965d61e1 ./new_file_name` to rename it
- Move the file to `llama.cpp/models` by using `sudo mv new_file_name /path_to/llama.cpp/models` for easier file management
- Rename the file and move the file once by `sudo mv 10cc391b403021dd11c614679d2fd92f611c3681d29e29651b717316965d61e1 /path_to/llama.cpp/models/new_file_name`
```bash
# Start a local Open-AI compatible server with a web UI:
# This command is used to download model files and run the model with network connection
./build/bin/llama-server -hf unsloth/Qwen3.5-4B-GGUF:Q8_0 --host 0.0.0.0 --port 8080

# Run inference directly in the terminal
./build/bin/llama-cli -hf unsloth/Qwen3.5-4B-GGUF:Q8_0

# You may also download the model files using aria2
aria2c -x 16 'model_files_address'

# After downloading the model files using aria2 you may need to rename the file
sudo mv 16ea896f904533fb76d6a73357f95089b2c1b586d3bffa4810cb910423e86c6f ./final_file_name
```

<br>

### Run Models
```bash
./build/bin/llama-server \
  -m ./models/gemma-3-4b-it-Q8_0.gguf \
  --mmproj ./models/mmproj-F32.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  -c 20480
  
./build/bin/llama-server \
  -m ./models/gemma-3-4b-it-Q4_0.gguf \
  --mmproj ./models/mmproj-F16.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  -c 16384
  
./build/bin/llama-server \
  -m ./models/Qwen3.5-4B-Q6_K.gguf \
  --mmproj ./models/mmproj-BF16.gguf \
  --host 0.0.0.0 \
  --port 8080 \
  -c 40960
```

| Prameters  | Purpose                                          |
| ---------- | ------------------------------------------------ |
| `-m`       | Clarify your model location                      |
| `--mmproj` | Clarify your model location for visual support   |
| `-host`    | Clarify the host port your model going to run on |
| `-port`    | Clarify the accessible port for web UI           |
| `-c`       | Sets the context windows                         |

<br>

---

<br>

## SSH (Secured)
### Installation
```bash
sudo pacman -S openssh
```
- Enable `SSH` service
```bash
sudo systemctl enable sshd
```

<br>

### SSH Keys
- Don't use a `password` to `login` your `server`
- Use `SSH Keys`
```bash
# Run these two following command on your own laptop
ssh-keygen -t ed25519

# Set your passphrase to 1
ssh-copy-id your_server_username@your_server_ipv4_address
```
- Now when you login your `server` will not ask for a `password`

<br>

### Turn off password
- Open your `sshd_config`
```bash
sudo nano /etc/ssh/sshd_config
```
- Set `PubkeyAuthentication` to `yes`
- Set `PasswordAuthentication` to `no`
```bash
PubkeyAuthentication yes
# To disable tunneled clear text passwords...
PasswordAuthentication no
```
- Restart `SSH`
```bash
sudo systemctl restart ssh
```

<br>

### Check what SSH is actually running
```bash
sudo sshd -T | grep -i passwordauth
```
- If you found it is still set to `yes`, fix the file inside that folder than restart `SSH` again
```bash
sudo systemctl restart ssh
```

<br>

### Check for every listening program
```bash
sudo ss -tulpn
```
- To disable the listening port which you don't need
```bash
sudo systemctl disable --now name_of_the_service
```

<br>

### Updates
- Make sure to upgrade your `server system`
```bash
# For Ubuntu Servers
sudo apt update && sudo apt upgrade && sudo apt install unattended-upgrade

# Reboot to update your server
systemctl reboot
```

<br>

### Fail2ban
- Installation on your `server`
```bash
sudo apt install fail2ban
```
- To watch it work
```bash
sudo fail2ban-client status sshd
```

<br>

### Weekly security check
- To see the bans
```bash
sudo fail2ban-client status sshd | tail -3
```
- Check who actually logged in
```bash
last -5
```

<br>

---

<br>

## Tailscale
Installation
```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

Status & Verification
- `tailscale status` — Displays the connection status, tailnet IP addresses, and active peers on your network.
- `tailscale ip` — Prints the Tailscale IPv4 and IPv6 addresses of your current device.
- `tailscale ping <peer-ip-or-name>` — Tests the connection to another device on your network and shows whether it is connecting via a direct peer-to-peer route or relayed (DERP).

Managing the Connection
- `tailscale up` — Starts Tailscale and connects the device to your tailnet.
- `tailscale down` — Disconnects your device from the tailnet (stops traffic routing but keeps the daemon running).
- `tailscale logout` — Disconnects the machine and clears all authentication keys, logging you out completely.

Feature Configuration (Flags for `tailscale up`)
- `tailscale up --advertise-exit-node` — Configures the current device to act as an exit node (allowing other devices to route all their internet traffic through it).
- `tailscale up --exit-node=<exit-node-ip-or-name>` — Forces your current device to route all its internet traffic through the specified exit node.
- `tailscale up --exit-node-allow-lan-access` — Used alongside the exit node command to ensure you can still talk to devices on your local physical Wi-Fi/LAN.
- `tailscale up --advertise-routes=192.168.1.0/24` — Configures the machine as a subnet router to expose an entire physical local network to your tailnet.

File Sharing & SSH
- `tailscale file send <peer-name> <path-to-file>` — Sends a file securely to another device on your tailnet using Taildrop.
- `tailscale file get <target-directory>` — Retrieves files that have been sent to your device via Taildrop.
- `tailscale ssh <user>@<peer-name>` — Establishes a secure SSH connection to another device without needing to manage SSH keys, provided Tailscale SSH is enabled.

Diagnostics & Advanced
- `tailscale netcheck` — Diagnoses your current physical network connection, showing your public IP, NAT mapping type, and latency to Tailscale relay servers.
- `tailscale cert <tailnet-domain-name>` — Fetches or renews free Let's Encrypt TLS certificates for your machine’s Tailscale domain.
- `tailscale version` — Displays the version of the Tailscale client currently installed.

Elimintate SSH Keys Entirely with Tailscale SSH
- `sudo tailscale up --ssh` Start Tailscale on the server with the SSH flag enabled
- `tailscale ssh user@server-name` On your local device, simply run this command line to connect your server with the SSH Key

<br>

---

<br>

## Xprinter xp-420b
### Step 1: Install CUPS and system dependencies
```bash
sudo pacman -S --needed cups cups-filters ghostscript
```

### Step 2: Install and Set Permissions for Binary Filters
```bash
# Navigate to your extracted files directory
cd ~/Documents/XP-420B_Driver/

# Copy driver binaries to the CUPS filter directory
sudo cp rastertosnailtspl-x64 /usr/lib/cups/filter/rastertosnailtspl
sudo cp rastertosnailxpl-x64 /usr/lib/cups/filter/rastertosnailxpl
sudo cp rastertosnailzpl-x64 /usr/lib/cups/filter/rastertosnailzpl
sudo cp rastertosnailep2-x64 /usr/lib/cups/filter/rastertosnailep2

# Set standard permissions and root ownership
sudo chown root:root /usr/lib/cups/filter/rastertosnail*
sudo chmod 755 /usr/lib/cups/filter/rastertosnail*

# Create the symlink expected by the XP-420B PPD configuration
sudo ln -sf /usr/lib/cups/filter/rastertosnailtspl /usr/lib/cups/filter/rastertosnailtspl-xprinter
```

### Step 3: Install the Printer PPD File
```bash
sudo mkdir -p /usr/share/cups/model/xprinter
sudo cp ~/Documents/XP-420B_Driver/XP-420B.ppd /usr/share/cups/model/xprinter/
sudo chmod 644 /usr/share/cups/model/xprinter/XP-420B.ppd
```

### Step 4: Start the CUPS Service
```bash
sudo systemctl enable --now cups.service
sudo systemctl restart cups.service
```

### Step 5: Register the USB Printer Queue
```bash
# 1. Verify USB device connectivity (should return: direct usb://Xprinter/XP-420B...)
lpinfo -v | grep usb

# 2. Register printer queue
sudo lpadmin -p "Xprinter_XP-420B" \
  -v "usb://Xprinter/XP-420B?serial=00000000000001" \
  -E \
  -P /usr/share/cups/model/xprinter/XP-420B.ppd
```

### Verify and Send a Test Print Job
```bash
# Check queue status (should state "printer Xprinter_XP-420B is idle")
lpstat -p Xprinter_XP-420B -l

# Print a test page
lpr -P Xprinter_XP-420B /usr/share/cups/data/testprint
```

<br>

---

<br>

## Green Tunnel
### Installation
- Install `Node.js` and `Green Tunnel` on Arch
```bash
sudo pacman -S nodejs npm
npm install -g green-tunnel
```
### Create a Native Systemd User Service
```bash
# Create a Native Systemd User Service
mkdir -p ~/.config/systemd/user/

# Create and open the service file:
nano ~/.config/systemd/user/greentunnel.service
```

### Paste the following configuration:
```bash
[Unit]
Description=Green Tunnel Proxy Service
After=network.target

[Service]
Type=simple
ExecStart=/home/arch/.npm-global/bin/gt -p 8080
Restart=always
RestartSec=3

[Install]
WantedBy=default.target
```

### Save the file, then enable and start the service for your user:
```bash
systemctl --user daemon-reload
systemctl --user enable --now greentunnel.service
```

### Verification
```bash
systemctl --user status greentunnel.service
```

### Configure Hyprland Environment Variables
- Edit your `~/.config/hypr/hyprland.lua` file 
- Add these following lines under the `---- Environment ----` section
```bash
-- Green Tunnel Proxy Configuration
hl.env("http_proxy", "http://127.0.0.1:8080/")
hl.env("https_proxy", "http://127.0.0.1:8080/")
hl.env("HTTP_PROXY", "http://127.0.0.1:8080/")
hl.env("HTTPS_PROXY", "http://127.0.0.1:8080/")
hl.env("no_proxy", "localhost,127.0.0.1,.local")
hl.env("NO_PROXY", "localhost,127.0.0.1,.local")
```

### Verification
```bash
ss -tulpn | grep 8080
```

<br>

---

<br>

## How to Mount Any External Drive Permanently on Arch Linux
When mounting an external drive automatically on boot using /etc/fstab, the configuration depends entirely on your drive's filesystem. Using the wrong options can lead to mount failures, read-only errors, or permission issues. Here is a comprehensive, universal guide tailored for your GitHub repository. It explains how to identify any drive's filesystem and partition type, and provides clear, adaptable templates for the most common filesystems (exFAT, NTFS, ext4, and vfat).

<br>

### 1. Verify Drive Details: Check UUID and filesystem.
- Connect your external drive.
- Run `lsblk` to find your partition name (e.g., `/dev/sda1`, `/dev/sdb1`).
- Run the following command with your specific partition to inspect its filesystem and UUID:
- _(Replace `/dev/sdX1` with your actual partition name)._
```bash
lsblk -f /dev/sdX1
```
#### Look at the output columns:
- **`FSTYPE`**: Tells you the filesystem (e.g., `exfat`, `ntfs`, `ext4`, `vfat`). **You must match this in your fstab entry.**
- **`UUID`**: The unique identifier of the partition. **Always use `UUID` instead of `device names`** (like `/dev/sda1`), because device letters can change randomly after a reboot or when other drives are plugged in.

<br>

### 2. Install Required Filesystem Tools
- Arch Linux does not enable support for all filesystems by default. Depending on what your drive uses, install the corresponding package:
```bash
# For exfat drives:
sudo pacman -S exfatprogs

# For ntfs drives:
sudo pacman -S ntfs-3g

# For ext4 / vfat / fat32: Built-in to the Linux kernel / standard utils (dosfstools for vfat).
```

<br>

### 3. Create the Mount Point Directory: Target folder.
- Create the folder where the external drive will be attached to your file system tree:
```bash
sudo mkdir -p /mnt/storage
```

<br>

### 4. Configure `/etc/fstab` Based on Your Filesystem
- Open the system's static filesystem table with a text editor:
- Scroll to the bottom of the file and add **one** of the lines below, depending strictly on the `FSTYPE` you found in **Step 1**. Make sure to replace `YOUR-UUID-HERE` with your actual drive `UUID`.
```bash
sudo nano /etc/fstab
```

#### Option A: For `exFAT` Filesystems
- _Why:_ `exFAT` does not natively support Linux POSIX file permissions. The `uid=1000,gid=1000` options force your local user account to own and have write access to the mounted files.
```text
UUID=YOUR-UUID-HERE /mnt/storage exfat defaults,uid=1000,gid=1000,rw,nofail 0 0
```
#### Option B: For `NTFS` Filesystems
- _Why:_ Uses the `ntfs-3g` driver for stable read/write access, mapping ownership to your local user ID (`1000`).
```bash
UUID=YOUR-UUID-HERE /mnt/storage ntfs-3g defaults,uid=1000,gid=1000,rw,nofail 0 0
```
#### Option C: For `ext4` Filesystems (Linux-native)
- _Why:_ `ext4` natively supports Linux permissions. You do not need `uid`/`gid` here; instead, you can change ownership _after_ mounting using `sudo chown -R $USER:$USER /mnt/storage`.
```bash
UUID=YOUR-UUID-HERE /mnt/storage ext4 defaults,noatime,nofail 0 2
```
#### Option D: For `vfat` / `FAT32` Filesystems
```bash
UUID=YOUR-UUID-HERE /mnt/storage vfat defaults,uid=1000,gid=1000,rw,nofail 0 0
```

<br>

### 5. Reload Systemd and Test: Apply changes immediately.
- Reload the systemd daemon to recognize the updated fstab file, then test the mount:
```bash
sudo systemctl daemon-reload
sudo mount -a
```

<br>

### Verification
- To confirm everything is mounted correctly and you have full read/write permissions as a regular user:
#### 1. Check the mount status:
- _(You should see `/mnt/storage` listed under `MOUNTPOINTS`)._  
```bash
lsblk -f /dev/sda1
```  
#### 2. Test creating a file without `sudo`:
```bash
touch /mnt/storage/test.txt
rm /mnt/storage/test.txt
```

<br>

---

<br>

## Distrobox
### Installation
```bash
sudo pacman -S distrobox podman
```

### Create container
- Create specific folder as storage for the containers (optional)
```bash
mkdir container-folder
```
- Create the container
```bash
distrobox create \
  --image fedora:latest \
  --name container_name \
  --nvidia \
  --home /path/to/your/container-folder
```

### Launch and start using your distrobox container
```bash
distrobox enter container_name
```

### Exclude the container folder from your system snapshots
```bash
distrobox stop container_name
sudo mv /path/to/container-folder /path/to/container-folder.bak
sudo btrfs subvolume create /path/to/container-folder
sudo cp -a /path/to/container-folder.bak/. /path/to/container-folder

# Verification
# Check if /path/to/container-folder is under the list
sudo btrfs subvolume list /
  
sudo rm -rf /path/to/container-folder.bak
```

<br>

---

<br>

## LocalSend
### Installation
```bash
sudo flatpak install -y flathub org.localsend.localsend_app
```
### Firewalld Configs
- You need to allow `port 53317` using firewalld to allow localsend works
```bash
sudo firewall-cmd --add-port=53317/tcp --permanent
sudo firewall-cmd --add-port=53317/udp --permanent
sudo firewall-cmd --reload
```

<br>

---

<br>

## Gwenview
- The `image viewer` created by `KDE`
```bash
sudo pacman -S gwenview
```

<br>

---

<br>

## Libre Office
- The best alternative to `Microsoft Office`
```bash
sudo flatpak install -y flathub org.libreoffice.LibreOffice
```
