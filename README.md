## Arch Installation - Connor Silks

**Partitioning the Disks:**

Firstly, I used the following command to find the name and path of my disk, /dev/sda/, so that I could begin partitioning it:
```
fdisk -l # lists disk and disk partitions
```

Then, I ran the following commands to create a 1 GiB /dev/sda1 partition to serve as an EFI system partition:
```
fdisk /dev/sda # creates a partition table for the disk for me to edit
n # creates a new partition
p # sets the partition type to primary
1 # sets the EFI boot partition to partition number 1
2048 # default value for starting sector for EFI partition
+1G # sets the last sector for the EFI partition to 1 GiB after the starting one
```

To create the root partition, I simply repeated the same steps, just with partition number 2, the starting sector as default (right after the EFI partition) and the last sector as the last possible value to end up with a ~39 GiB /dev/sda2 root partition.

Next, I formatted the root partition with an Ext4 file system using ``mkfs.ext4 /dev/sda2`` and mounted it with ``mount /dev/sda2 /mnt``. I formatted the EFI system partition as FAT32 using ``mks.fat -F 32 /dev/sda1`` and mounted it using ``mount --mkdir /dev/sda1 /mnt/boot``.

**Installing Basic Packages + nano:**

I chose to just stick with the default mirrors for installing the essential packages. Then, I installed the base, linux, linux-firmware, and nano packages with the following command:
``pacstrap -K /mnt base linux linux-firmware nano``

**Configuration:**

Following the arch wiki, I ran the following commands to configure Fstab, chroot, time, localization, and root password.
```
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt # changes root into the new file system
ln -sf /usr/share/zoneinfo/US/Central /etc/localtime # Set time to CST
hwclock --systohc # generates /etc/adjtime
locale-gen # after uncommenting en_US.UTF=9 UTF-8 in the locale.gen file
touch /etc/locale.conf # then used nano to set contents to LANG=en_US.UTF-8
touch /etc/hostname # set contents to "archlinux" as my hostname
passwd # followed by me actually inputting a root user password
```

For network configuration, I started  by running the following commands to check and enable my available network interface:
```
ip link # outputs the available network interfaces
ip link set ens33 up # enables ens33, the available "ethernet" network interface
touch /etc/system/network/20-wired.network # creates a 20-wired.network file to                                                 begin the DHCP/DNS process for it.
```

Then I used ``nano /etc/system/network/20-wired.network`` to set the contents to:
```
[Match]
Name=ens33

[Network]
DHCP=yes
```

Followed by running:
```
systemctl enable --now systemd-networkd.service # enables systemd-networkd
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf # links the two files
systemctl enable --now systemd-resolved.service # enables systemd resolved,                                                          creating/setting an IP
networkctl # to check if the network interface now has an IP and is properly up
```

In the following commands, I install NetworkManager and enable + start it so that I can later acces the networks in my Desktop Environment:
```
pacman -S networkmanager # installs NetworkManager package
systemctl enable NetworkManager.service # enables NetworkManager
systemctl start NetworkManager.service # starts NetworkManager
```

**Bootloader Installation:**

I ran the following commands to install the GRUB bootloader:
```
pacman -S grub # installs GRUB bootloader package
grub-install --target=i386-pc /dev/sda # installs GRUB in /dev/sda disk
grub-mkconfig -o /boot/grub/grub.cfg # generates GRUB config file
```

After running the ``reboot`` command, everything looked good and the portion of the install covered by the Arch Wiki was complete.

**Desktop Environment:**

I chose KDE Plasma as my Desktop Environment of choice and installed its (recommended) meta package with the following command:
```
pacman -S plasma-meta # installs plasma meta package
```

**User Accounts and Permissions:**

I ran the following commands to create a personal user account for me and a user account for codi and to subsequently provide them with sudo permissions:
```
useradd -m connor # creates connor user with a home directory
useradd -m codi # same for codi user
pacman -S sudo # installs sudo package
pacman -S vi # installs vi text editor to use visudo
sudo visudo /etc/sudoers # give codi and connor "ALL=(ALL:ALL) ALL" permissions
sudo usermod -aG wheel connor # adds connor user to wheel (sudo) group
sudo usermod -aG wheel codi # same for codi user
```

**Shell and ssh Installation:**

I ran the following two simple commands to install the fish (shell) package and to install ssh:
```
pacman -S fish # installs fish, my different shell of choice
pacman -S openssh # installs ssh
```

**Color Coding of the Terminal:**

I put the following command in the ~/.bashrc file contents to color code my terminal:
```
export PS1="\e[36m[\e[34m\u@\e[36m\h \e[32m\W\\e[36m]\e[34m$\e[0m "
```
The command uses ANSI escape sequences to color different parts of the text.

The color coding in the arch terminal looks like:
![[Pasted image 20251102213943.png]]

The color coding in Konsole looks like this (Added After Desktop Environment Launch): 
![[Pasted image 20251102223940.png]]

**Booting into the Desktop Environment:**

The following commands utilize the sddm display manager to set the syste to boot into the GUI desktop environment by default:
```
pacman -S sddm # installs the sddm display manager package
sudo systemctl status sddm # checks if sddm, my display manager, is active
sudo systemctl enable sddm.service # activates/enables sddm
sudo systemctl start sddm.service # boots into KDE Plasma through sddm
```

Below is an image of the log-in screen, with both added users, connor and codi showing up.
![[Pasted image 20251102215658.png]]

Below is an image of the KDE Plasma Desktop Environment after being booted into:
![[Pasted image 20251102221340.png]]

**Aliases:**

I ran the following command to edit the .bashrc file for the connor user to set aliases for it:
```
nano ~/.bashrc
```

The contents, after editing, of the .bashrc file are:
```
export PS1="\e[36m[\e[34m\u@\e[36m\h \e[32m\W\\e[36m]\e[34m$\e[0m "

alias ls='ls --color=auto' # aliases ls to a color-coded version
alias grep='grep --color=auto' # same but with grep
alias egrep='egrep --color=auto' # same but with egrep
alias fgrep='fgrep --color=auto' # same but with fgrep
alias mkdir='mkdir -pv' # aliases mkdir to automatically make parent directories                             and output the path to the new directory
alias root='sudo -i' # aliases "root" to the command to switch to root user
alias c='clear' # aliases "c" as the clear command
```

**Questions and Answers:**

Q: What Operating System do I choose in the VMware prompt after inputting the Arch ISO?
A: Choose the "Other Linux 6.x kernel" option. 

Q: How do I properly set up my network? The installation wiki is very vague.
A: Utilize the systemd-networkd and systemd-resolved built-in network managers to use DHCP and DNS to properly set up a connection with available network interfaces. Process is outlined in the systemd-networkd Arch wiki entry.

Q: How do I color code text?
A: Use ANSI escape sequences, typically indicated by "\e3(color code)m". The various capabilities are displayed in the Bash/Prompt customization section of the Arch Wiki. The PS1 variable contains the prompt text, which can be color coded.

Q: How do I provide sudo permissions? The slides in class give an overview but end with saying that editing /etc/sudoers is not how you fully provide sudo permissions.
A: Editing /etc/sudoers is a critical step, but you must also add the user(s) to the wheel group, which is the default sudoer group in a fresh install.

**Errors/Issues:**

I was initially confused of how to go about setting up the network connection. I remember hearing it was an important step in class the wiki itself, as previously mentioned in the questions section, was quite vague and the Network Configuration section it guides you to is confusing at first glance. However, once looking at the Network Manager section and looking up the ideal manager, I discovered that I could use systmd-networkd to establish a connection to my available ethernet network interface.

I couldn't get the color-coding to stick at first after a restart, then I realized through the aliasing step and searching online that ~/.bashrc is how to set the PS1 environment variables, and later aliases, on restart properly.

I did not realize I needed to use NetworkManager, so while I did have connection to install packages while in the arch command line, I had no network connection listed after booting the Desktop Environment. I solved this by going back and installing the NetworkManager package and enabling it, as listed in my installation process. I retroactively added this step to where it would make sense, alongside the rest of the network configuration, which hopefully is fine!

