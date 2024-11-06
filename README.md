# Encrypted Arch Linux Installation with Hyprland

This guide covers the installation of Arch Linux with LUKS encryption and the Hyprland window manager. Carefully follow the steps, and make sure you are working with the correct devices and partitions to prevent data loss.

----------

### 1. Boot into the Arch Linux ISO and Connect to the Internet

1.  **Boot into the Arch Linux Live USB**:
    -   Insert the bootable USB and select it from your system's boot menu.

2.  **Verify network connection**:
    -   You can test if you're already connected by running:
        `# ping archlinux.org` 
        
    If there is no connection, proceed to the next steps to set up a Wi-Fi connection using `iwctl`.
    
3.  **Connect to Wi-Fi** (if necessary):
    -   Start the interactive `iwctl` tool:
	    `# iwctl` 
        
    -   List available Wi-Fi devices:
        `[iwd]# device list` 
        
    -   Enable the Wi-Fi adapter (if powered off):
        `[iwd]# device [device_name] set-property Powered on` 
        
    -   Scan for available networks:
        `[iwd]# station [device_name] scan` 
        
    -   Show available networks:
        `[iwd]# station [device_name] get-networks` 
        
    -   Connect to your preferred network:
        `[iwd]# station [device_name] connect [SSID]` 
        
    -   Exit `iwctl` and test your connection:
        `# ping 8.8.8.8` 

----------

### 2. Wipe the Disk (Optional but Recommended)

Wipe the disk to ensure no remnants of previous data remain. **This will irreversibly erase all data.**

1.  **List available disks**:
    `# lsblk` 
    
2.  **Wipe the disk with random data** (replace `/dev/sdX` with your target disk):
    `# dd if=/dev/urandom of=/dev/sdX bs=1M status=progress` 
    
    -   This will overwrite the disk with random data to improve security before encryption.

----------

### 3. Partition the Disk

Create the partitions required for the system. This guide assumes UEFI boot.

1.  **Open the disk in `gdisk`**:
    `# gdisk /dev/sdX` 
    
2.  **Create a new GPT partition table**:
    -   Type `o` to create a new empty GPT partition table, confirm by pressing `Enter`.

3.  **Create the EFI partition**:
    -   Type `n` to create a new partition. For the first sector, press `Enter` to use the default. For the last sector, type `+1G` to allocate 1GB.
    -   Set the partition type to EFI by entering `EF00`.

4.  **Create the root partition**:
    -   Type `n` again, use defaults for the first and last sectors to allocate the rest of the disk.
    -   The partition type for the root partition will remain the default (`8300`).

5.  **Write the partition table**:
    -   Type `w` and confirm to write the changes.

----------

### 4. Set Up Encryption and Format the Partitions

1.  **Format the EFI partition**:
    `# mkfs.fat -F32 /dev/sdX1` 
    
2.  **Encrypt the root partition**:
    -   Use LUKS to encrypt the root partition:
        `# cryptsetup luksFormat /dev/sdX2` 
        
    -   Confirm with "YES" and enter a passphrase when prompted.

3.  **Open the encrypted partition**:
    `# cryptsetup open /dev/sdX2 cryptroot` 
    
4.  **Format the root partition**:
    `# mkfs.ext4 /dev/mapper/cryptroot` 

----------

### 5. Mount the Partitions

1.  **Mount the root partition**:
    `# mount /dev/mapper/cryptroot /mnt` 
    
2.  **Create a directory and mount the EFI partition**:
    `# mkdir /mnt/boot`
    `# mount /dev/sdX1 /mnt/boot` 

---------- 

### 6. Configure Pacman for Faster Downloads 

To speed up package installation, enable parallel downloads and verbose output in Pacman. 

1. Open the Pacman configuration file: 
	`# nano /etc/pacman.conf` 

2. Uncomment the following lines: 
	- `#VerbosePkgLists` (remove the `#`) 
	- `#ParallelDownloads = 5` (remove the `#`) 

3. Save and exit. 

----------

### 7. Setup Mirrors for Faster Package Downloads 

To ensure you're downloading packages from fast and reliable servers, use the `reflector` tool to update your mirrorlist. 

1. Refresh the package list: `# pacman -Syy` 

2. Install the `reflector` tool: `# pacman -S reflector` 

3. Backup your current mirrorlist (just in case): `# cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak` 

5. Use `reflector` to select the fastest mirrors in the US: `# reflector -c "US" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist`

----------

### 8. Install Essential Packages

1.  **Install base system and necessary packages**:
    `# pacstrap /mnt base base-devel linux linux-firmware networkmanager lvm2 cryptsetup grub efibootmgr nano vim` 

----------

### 9. Generate fstab

1.  **Generate the fstab file**:
    `# genfstab -U /mnt >> /mnt/etc/fstab` 
    
2.  **Verify the fstab entries**:
    `# cat /mnt/etc/fstab` 

----------

### 10. Chroot into the New System

1.  **Change root into the installed system**:
    `# arch-chroot /mnt` 

----------

### 11. Set Up Timezone and Clock

1.  **Set your timezone**:
    `# ln -sf /usr/share/zoneinfo/Region/City /etc/localtime` 
    
2.  **Sync hardware clock**:
    `# hwclock --systohc` 
    
3.  **Check the system date**:
    `# date` 

----------

### 12. Configure Locale

1.  **Uncomment your preferred locale in the `/etc/locale.gen` file**:
    `# nano /etc/locale.gen` 
    
    -   Uncomment the line for your locale (e.g., `en_US.UTF-8 UTF-8`).

2.  **Generate the locale**:
    `# locale-gen` 
    
3.  **Create `/etc/locale.conf` and set your locale**:
    `LANG=en_US.UTF-8` 

----------

### 13. Set Hostname and Hosts File

1.  **Set your hostname**:
    `# echo "myhostname" > /etc/hostname` 
    
2.  **Configure `/etc/hosts`**:
    `# nano /etc/hosts` 
    
    Add the following lines:
   `127.0.0.1	localhost`
   `::1       	localhost`
   `127.0.1.1 	myhostname.localdomain myhostname` 

----------

### 14. Set Root Password and Create User

1.  **Set the root password**:
    `# passwd` 
    
2.  **Create a new user**:
    `# useradd -m -G wheel -s /bin/bash username` 
    
3.  **Set the user's password**:
    `# passwd username` 
    
4.  **Enable sudo for the user**:
    `# EDITOR=nano visudo` 
    
    -   Uncomment the line:
        `%wheel ALL=(ALL:ALL) ALL` 

----------

### 15. Configure mkinitcpio

1.  **Edit `/etc/mkinitcpio.conf`**:
    `# nano /etc/mkinitcpio.conf` 
    
    -   Add `encrypt` and `lvm2` hooks between `block` and `filesystems` in the HOOKS line.

2.  **Regenerate the initramfs**:
    `# mkinitcpio -P` 

----------

### 16. Install and Configure GRUB

1.  **Install GRUB**:
    `# grub-install --efi-directory=/boot --bootloader-id=GRUB /dev/sdX` 
    
2.  **Edit `/etc/default/grub` to enable encryption**:
    -   First, get the UUIDs of both the encrypted partition and the root filesystem:
        `# blkid -s UUID -o value /dev/sdX2  # for the encrypted partition`
        `# blkid -s UUID -o value /dev/mapper/cryptroot  # for the root filesystem` 

		To store these values in the configuration file:
        `# blkid -s UUID -o value /dev/sdX2 >> /etc/default/grub`  
        `# blkid -s UUID -o value /dev/mapper/cryptroot >> /etc/default/grub` 
        
    -   Open the GRUB configuration file:
        `# nano /etc/default/grub` 
        
    -   In the `GRUB_CMDLINE_LINUX_DEFAULT` line, add the following:
        `cryptdevice=UUID=[encrypted partition UUID]:cryptroot root=UUID=[root UUID]` 
        
        -   Replace `[encrypted partition UUID]` with the UUID of your encrypted root partition (`/dev/sdX2`).
        -   Replace `[root UUID]` with the UUID of your decrypted root filesystem (`/dev/mapper/cryptroot`).

3.  **Generate the GRUB config**:
    `# grub-mkconfig -o /boot/grub/grub.cfg`

----------

### 17. Enable NetworkManager and Reboot

1.  **Enable NetworkManager**:
    `# systemctl enable NetworkManager` 
    
2.  **Exit the chroot environment**:
    `# exit` 
    
3.  **Unmount the partitions**:
    `# umount -R /mnt` 
    
4.  **Reboot**:
    `# reboot` 

----------

## After Rebooting

Once your system reboots, log in with your newly created user account to finalize the setup of your Arch Linux installation with Hyprland.

----------

### 1. Connect to the Internet Using `nmtui`

To set up your network connection, use `nmtui`, a simple and user-friendly tool to manage network connections:

1.  **Launch `nmtui`**:
    `sudo nmtui` 
    
2.  In the interface, use the arrow keys to navigate and select **Activate a connection**.
    
3.  Select your Wi-Fi network from the list and press Enter.
    
4.  Enter the password for your network and confirm.
    
5.  Once connected, exit `nmtui` by selecting **Quit**.

----------

### 2. Repeat Pacman Configuration for Faster Downloads

Now that you're logged into your system, configure Pacman again to enable faster downloads and a more detailed package installation process:

1.  **Open the Pacman configuration file**:
    `sudo nano /etc/pacman.conf` 
    
2.  **Uncomment the following lines** by removing the `#`:
    -   `#VerbosePkgLists` (this enables detailed output during package installation)
    -   `#ParallelDownloads = 5` (this allows Pacman to download 5 packages in parallel, speeding up the process)

3.  **Save and exit** the file.
This will make future package installations faster and more informative.

----------

### 3. Hyprland Installation and Setup (Packages may differ depending on your needs)

1.  **Install essential packages for Hyprland**:
`sudo pacman -S gtk2 gtk3 gtk4 kitty hyprland`

2. **Install additional packages**:
`sudo pacman -S alsa-utils blueman bluez bluez-utils brightnessctl btop code fastfetch firefox git pipewire-pulse python-requests rofi swaybg ttf-jetbrains-mono-nerd waybar wireplumber xdg-desktop-portal-hyprland xdg-user-dirs yazi`

3.  **Initialize user directories**:
`xdg-user-dirs-update`

4. **Install Yay for AUR packages**:
To enable installation of AUR packages, follow these steps to install Yay: 
`sudo pacman -Syu`
`sudo pacman -S --needed base-devel git`
`git clone https://aur.archlinux.org/yay.git`
`cd yay makepkg -si`

5. **Install AUR packages**:
`yay -S spotify spicetify-cli webcord wlogout`

7.  **Enable and start Bluetooth**:
`sudo systemctl start bluetooth`
`sudo systemctl enable bluetooth`

----------

### 4a. Create and Enable a Swapfile on the Root Partition

1.  **Check if any swapfiles or partitions are already in use**:
    `sudo swapon -s` 
    
2.  **Turn off all swap devices**:
    `sudo swapoff -a` 
    
3.  **Create a 17GB swapfile** (adjust size as needed based on your system's RAM):
    `sudo dd if=/dev/zero of=/swapfile bs=1M count=17408` 
    
4.  **Secure and enable the swapfile**:
    `sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile` 
    
5.  **Make the swapfile persistent** by adding it to `/etc/fstab`:
    `echo '/swapfile none swap defaults 0 0' | sudo tee -a /etc/fstab` 
    
6.  **Verify the `/etc/fstab` changes**:
    `sudo mount -a` 
    
    If no errors appear, remove the backup of `/etc/fstab`:
    `sudo rm /etc/fstab.backup` 
    
----------

### 4b. Enable Hibernation and Update GRUB

To enable hibernation, we need to add the `resume` kernel parameter to the GRUB configuration.

1.  **Find the UUID of the root partition**:
    -   Use the `blkid` command to list the partitions and their UUIDs:

    -   Identify the root partition (usually mounted on `/`). The UUID will be listed next to it. For example:
        `/dev/sdX: UUID="f68ed3c5-da10-4288-890f-b83d8763e85e" TYPE="ext4"` 
        
    -   Make note of this UUID, as it will be used in the GRUB configuration.

2.  **Find the physical offset of the swapfile**:
    -   Use the `filefrag` command to get the physical offset of the swapfile:
        
        `sudo filefrag -v /swapfile` 
        
    -   Look for the first value under the `physical_offset` column. For example:
        `ext: 0   logical_offset: 0   physical_offset: 45731840` 
        
    -   Make note of this `physical_offset` value.
        
3.  **Edit the GRUB configuration**:
    -   Backup the current GRUB configuration:
        `sudo cp /etc/default/grub /etc/default/grub.backup` 
        
    -   Open `/etc/default/grub` for editing:

        `sudo nano /etc/default/grub` 
        
    -   Find the line starting with `GRUB_CMDLINE_LINUX_DEFAULT` and append the `resume` and `resume_offset` parameters using the information you gathered. For example:

       -   For LUKS-encrypted systems:

            `GRUB_CMDLINE_LINUX_DEFAULT="quiet cryptdevice=UUID=de70eb7e-3108-4a69-a58f-febdf925c39a:cryptroot root=UUID=f68ed3c5-da10-4288-890f-b83d8763e85e resume=UUID=f68ed3c5-da10-4288-890f-b83d8763e85e resume_offset=45731840"` 
            
4.  **Regenerate the GRUB configuration**:
    `sudo grub-mkconfig -o /boot/grub/grub.cfg` 

5. **Add the Resume Hook to Initramfs**:
    -   Open the `mkinitcpio.conf` file for editing:
        `sudo nano /etc/mkinitcpio.conf` 
        
    -   Find the line that starts with `HOOKS=()`. You will need to add `resume` to this line, preferably after the `filesystems` hook. For example:
        `HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems resume fsck)` 
        
    -   After saving your changes, regenerate the initramfs with the following command:
        `sudo mkinitcpio -P`
    
9.  **Reboot your system**:
    `sudo reboot` 
    
Once the system reboots, hibernation should now work. You can test it using:
`systemctl hibernate`

----------
