# Extra Configurations for Arch Linux

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

# Enable weekly automation
sudo systemctl enable --now reflector.timer
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

## Chinese Input (Optional)
* This following chinese input tool is specified for GNOME
* If using hyprland you may use fcitx5
```bash
sudo pacman -S ibus-libpinyin wqy-microhei wqy-zenhei

gnome-session-quit --logout
```

<br>

---

<br>

## Install yay and CoolerControl
* Using CoolerControl you may also create your own cooling fan curve which your fan spin following by the temperature of your hardware.
```bash
sudo pacman -S qt6-wayland
yay -S coolercontrol-bin
sudo systemctl enable --now coolercontrold

# Open it in your web browser
# http://localhost:11987
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
* Installation
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

# Make sure Steam is completely closed before continuing
pkill -x steam 2>/dev/null || true
# Move the Steam installation
mkdir -p ~/.local/share

# If an existing Steam installation exists, temporarily move it away
if [ -d ~/.local/share/Steam ]; then
    mv ~/.local/share/Steam ~/.local/share/Steam.bak
fi

# Create the dedicated Btrfs subvolume
sudo btrfs subvolume create ~/.local/share/Steam

# Restore existing Steam data if present
if [ -d ~/.local/share/Steam.bak ]; then
    cp -a ~/.local/share/Steam.bak/. ~/.local/share/Steam/
    rm -rf ~/.local/share/Steam.bak
fi

# 4. Verify that Steam is a Btrfs subvolume
sudo btrfs subvolume list /

# 5. Launch Steam
steam
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



# config file
# --- Font ---
font-size = 15
font-variation = wght=600

# --- Appearance & Window ---
foreground = #e2e8f0
background = #0f172a
selection-foreground = #0f172a
selection-background = #94a3b8

# Liquid Glass Visuals
background-opacity = 0.75
background-blur-radius = 20

window-padding-x = 12
window-padding-y = 10
window-decoration = false
maximize = true

# --- Cursor ---
cursor-style = block
cursor-style-blink = true

# --- Mouse & Scroll ---
scrollback-limit = 50000
mouse-hide-while-typing = true

# --- Keybindings ---
keybind = ctrl+d=new_split:right
keybind = ctrl+shift+d=new_split:down
keybind = ctrl+t=new_tab
keybind = ctrl+w=close_surface
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
