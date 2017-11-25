TABLE OF CONTENTS:

PART 1: Preparation and Disk Partitioning

Preparation
-Items necessary
-Downloads necessary
-Details regarding enabling EFI mode via BIOS
-Details regarding booting from USB in EFI mode
-Verifying internet connectivity
-Verifying EFI mode is enabled

Disk Partitioning
-Finding all available drives
-Wiping the existing partition table
-Creating boot partition and the difference between EF00 and EF02 Hex codes
-Creating swap partition, the swap debate, choosing a swap size, and the swap 8200 hex code
-Creating root and home, the differences between them, and choosing whether to keep them on the same partition
-Telling linux which file systems to use for our partition

PART 2: Installing Arch and Making it Boot

-Mounting our partitions
-Setting up our Arch repository mirrorlist
-Installing the Arch base files
-Generating an fstab file
-Language
-Time
-Hostname
-Enabling multilib and Arch AUR community repositories
-Root password and user setup
-Setting up sudoers
-Adding bash-completion
-Installing the bootloader
-Creating the bootloader config file
-Enabling microcode for Intel processors
-Installing video card drivers

PART 3: Making it user friendly and adding a desktop environment

-Enabling internet connection
-Installing touchpad support
-Installing 3D support
-Installing X server display manager
-Installing a desktop manager
——————————-

***PART 1: Preparation and Disk Partitioning***

Preparation:

-Items necessary
1. A 4gb or higher USB stick

-Downloads necessary
2. Arch linux ISO image
https://www.archlinux.org/download/

For creating bootable usb on windows
3. Rufus

For creating bootable usb on linux
3a.

> sudo dd bs=4M if=/path/to/archlinux.iso of=/dev/sdX status=progress && sync

(sdX being your USB stick. You can find this withthe command: lsblk)
-Details regarding enabling EFI mode via BIOS
If you are not sure if your computer is booted/can boot in UEFI/EFI mode, check your motherboard manual. It will tell you how to enable it in the BIOS.

-Details regarding booting from USB in EFI mode
You will also need to plug in your new Arch USB stick, then reboot and either go in to your bios and set it as the first boot device, or use the hot key the motherboard specifies during booting to access the boot menu, then select the USB stick as the boot device from that menu. My boot device key is F11, and my BIOS access key is DEL.

Ok, so we have our USB stick, we plug it in. We boot into the USB stick. We select the first option that reads something like “Arch Linux archiso x86_64 UEFI USB”

We are now at a screen that reads
root@archiso ~ #

-Verifying internet connectivity
Before we do anything, we need to confirm we have an internet connection, and that we are actually in EFI mode. I use a wired connection, so linux should auto-detect it. I will post a wifi connection sublink when I am able to make it, for now we are going to just use our LAN connection. Test it by typing:


> ping -c 3 www.google.com

if you get a response, your internet is working properly.
-Verifying EFI mode is enabled
Now test if we are using UEFI mode by typing:

> efivar -l

if it spits out a list of stuff (uefi variables) then you are using UEFI mode.
——————————-
Disk Partitioning:
-Finding all available drives
First I need to set up my partitions. I wiped my current partitions to make a fresh install using the whole drive. I need to find out which partitions I want to use, by typing:

> lsblk

This shows me all of the drives availabe.
In my case, I have 3 drives
sda 238.5GB- this is my 256gb ssd drive
–sda1 – Existing Partition on drive
sdb 931.5GB- this is my 1tb storage drive
–sdb1 – Existing Partition on drive
sdc – this is the usb stick im using to install arch
–sdc1 – Existing Partition on drive

These are known to the system as /dev/sda, /dev/sda1, /dev/sdb, /dev/sdb1 and so forth.

-Wiping the existing partition table
For me, /dev/sda is the drive I want to install linux on, which I will be wiping/deleting partitions from. I need to “zap” the current partition table so that I can re-write it as a GPT partition table

***PLEASE NOTE: THIS WILL WIPE THE ENTIRE DRIVE**

> gdisk /dev/sdX (x representing your drive. mine is sda)
> x
> z
> y
> y

boom. zapped.
-Creating boot partition and the difference between EF00 and EF02 Hex codes
Now to create my new partitions. So lets start partitioning:

> cgdisk /dev/sdX

Any key to continue
Note: the order you create the partitions is the order they will be listed in, so if I create three partitions in order such as boot, swap, root, home on /dev/sda, then “lsblk” again, they will be listed as:
sda
-sda1 (our boot partition)
-sda2 (our swap partition)
-sda3 (our root partition)
-sda4 (our home partition)

I’m going to create my boot partition first. I am using EFI, so EF00 will be our hex code (NOT EF02. I’ve racked my brain over this error before trying to figure out why EFI system wouldn’t boot). I generally dedicate 1Gb (1024MiB) of space to the boot sector so that I have room to breathe in case I need to change anything or add multiple boot kernels, although arch wiki recommends only 200-300Mb. I also will name it “boot”.

> [New] Press Enter
> First Sector: Leave this blank ->press Enter
> Size in sectors: 1024MiB ->press Enter
> Hex Code: EF00 press Enter
> Enter new partition name: boot ->press Enter
Note the 1007KiB existing before the boot partition we just made. This is where the Protective MBR is, and is present on all GPT partition tables. This cannot be removed/deleted. (This is yet another thing I’ve racked my brain over in the past while trying to create boot partitions). It is OK to ignore.

Now, arrow down to the next free space available, then go to [New] again.

-Creating swap partition, the swap debate, choosing a swap size, and the swap 8200 hex code
To swap, or not to swap, that is the question. There has always been a debate on whether or not to create a swap partition when using an SSD or if you have a large amount of memory.

Short Answer: Yes. Always create a swap partition.

Detailed Explanation:
The way the Linux kernel works, swap isn’t only used when you have exhausted all physical memory. The Linux kernel will take applications that are not active (sleeping) and after a period of time, move the application to swap from real memory. The result is that when you need that application, there will be a momentary delay (usually just a second or two) while the application’s memory is read back from swap to RAM.

If you’re running a laptop or a desktop that you might want to put in ‘hibernate’ mode (Suspend to Disk), then you always want at least as much swap as you have memory. The swap space will be used to store the contents of the RAM in the computer while it ‘sleeps’. Additionally, this allows you to put inactive applications to “sleep”, giving your active applications access to additional RAM.

In the unlikely event that you run out of RAM – perhaps opening a big file, perheps a long running tab in firefox, it doesn’t matter, in that event your kernel OOM killer will kick in and start killing applications to get memory back. From a developer perspective, you also need to have substantial swap space if you are running Java/Java apps.

References:
http://serverfault.com/questions/5841/how-much-swap-space-on-a-high-memory-system
http://askubuntu.com/a/49130

Ok, I’ll create a swap partition, but how big?
Do I want hibernation?

Here is the Redhat preferred reference table for linux swap partition sizes:
(Note: Redhat is another Linux Distribution, however linux partitions are utilized the same across all distros)

Amount of RAM in the system Recommended swap space Recommended swap space
if allowing for hibernation
————————— —————————- —————————
2GB of RAM or less 2 times the amount of RAM 3 times the amount of RAM
2GB to 8GB of RAM Equal to the amount of RAM 2 times the amount of RAM
8GB to 64GB of RAM 0.5 times the amount of RAM 1.5 times the amount of RAM
64GB of RAM or more 4GB of swap space No extra space needed

In my case I have 16GB of RAM, and wish to use hibernation, In cases where you have more than 8GB of ram, you don’t usually need a lot of swap since you have more physical memory to handle tasks, so I went with 0.5 x 16, which is 8GB of swap space:

> [New] Press Enter
> First Sector: Leave this blank ->press Enter
> Size in sectors: 8GiB ->press Enter
> Hex Code: 8200 ->press Enter
> Enter new partition name: swap ->press Enter

Ok, boot and swap: done.
-Creating root and home, the differences between them, and choosing whether to keep them on the same partition

Root:
Last I’m going to create my root partition. I normally don’t create a home partition, I just store /home inside the root partition since I prefer to not have to worry about the partition size for /home being limited. However
I will explain both for the sake of user options.

What is the difference?
Root-
In comparison to windows, Root is like your C: drive. Generally you don’t wanna mess with anything inside it unless you know what you’re doing.

Home-
Home is where your user files are stored. In windows that would be C:\Users\Someusername, which would then contain My Documents, Downloads, Pictures, Videos etc. All of the folders pertaining to that user. Some may
prefer to have /home on a seperate partition for security or storage sake.

Alright, so what size do I need to make them?

If you are only using root, and storing /home inside of it, you can create root using all of the default values (except giving it the name of “root”) for the new partition options by just pressing Enter.

If you are creating a seperate /home partition, arch wiki recommends the root partition be 15-20GB, and that the /home partition be whatever size you like (I would use the remainder of the disk). You can do this by first creating the root partition, and specifying 20GiB when choosing the size, naming it “root” (all other options just press Enter), then creating the home partition again using all default options and naming it “home”.

Arrow down to the next free space available, then go to [New] again.

For root with /home inside:

> [New] Press Enter
> First Sector: Leave this blank ->press Enter
> Size in sectors: Leave this blank ->press Enter
> Hex Code: Leave this blank ->press Enter
> Enter new partition name: root ->press Enter
> For root with seperate /home partition:

> [New] Press Enter
> First Sector: Leave this blank ->press Enter
> Size in sectors: 20GiB ->press Enter
> Hex Code: Leave this blank ->press Enter
> Enter new partition name: root ->press Enter

Arrow down to the next free space available, then go to [New] again.

> [New] Press Enter
> First Sector: Leave this blank ->press Enter
> Size in sectors: Leave this blank ->press Enter
> Hex Code: Leave this blank ->press Enter
> Enter new partition name: home ->press Enter

Arrow over to [Write] to save your new partitions, hit enter, type “yes”, hit enter again.
Lastly, Arrow over to [Quit] and press enter.
Reboot
-Telling linux which file systems to use for our partition

I now need to let linux know the file system for our partitions. For EFI with GPT, boot needs to be Fat32. For swap we simply use mkswap. The rest are default ext4 file systems:

> mkfs.fat -F32 /dev/sda1
> mkswap /dev/sda2
> swapon /dev/sda2
> mkfs.ext4 /dev/sda3
> mkfs.ext4 /dev/sda4

root and home done.
——————————-
***PART 2: Installing Arch and Making it Boot***
-Mounting our partitions
Ok, so we have our partitions. We need to mount them.

> mount /dev/sda3 /mnt
> mkdir /mnt/boot
> mkdir /mnt/home
> mount /dev/sda1 /mnt/boot
> mount /dev/sda4 /mnt/home
-Setting up our Arch repository mirrorlist
Before we initiate the install process let’s select the closest mirror so that you get the best speed while downloading packages. I’ve found the easiest method to do this came via the arch wiki.
Make a backup:

> cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup

Now, instead of going through the list with nano and trying to scroll down a million mirrors, we’re going to do something different. Run the following sed line to uncomment every mirror:

> sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup

Lets sort that backup. This command will run a check for the top 6 mirrors you have the best connection to, and leave them uncommented while commenting out the rest:

> rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist

-Installing the Arch base files
Now we install the arch base files and development files. This will take some time.

> pacstrap -i /mnt base base-devel

For any options that come up just press enter or type y and press enter
-Generating an fstab file
and now we generate our fstab file

> genfstab -U -p /mnt >> /mnt/etc/fstab

now edit it to make sure an entry is listed for each partition. by typing

> nano /mnt/etc/fstab

check to see if there is an entry in fstab for swap since we are here. if there is, it should look something like:
/dev/sda2 none swap defaults 0 0
or
UUID=some-crazy-long-random-id none swap defaults 0 0
Now we are going to chroot into our newly installed system and begin to configure its booting, time, and language

> arch-chroot /mnt

-Language
Create locale file:

> nano /etc/locale.gen

Uncomment your locale. I uncommented en_US.UTF-8. You can search the file for it by typing ctrl+W, type en_US.UTF-8, hit enter, then uncomment it and press ctrl+x to exit
Now generate that locale by typing:

> locale-gen

and then set it as your language with:

> echo LANG=en_US.UTF-8 > /etc/locale.conf
> export LANG=en_US.UTF-8

-Time
List the available time zone info with

> ls /usr/share/zoneinfo/

Then link the appropriate one via something like:

> ln -s /usr/share/zoneinfo/your-time-zone > /etc/localtime

Mine for example was:

> ln -s /usr/share/zoneinfo/America/New_York > /etc/localtime

Now the hardware clock:

> hwclock --systohc --utc

-Hostname
This is the name of your machine, when used it will show @hostnameyoupick.

> echo somehostname > /etc/hostname
Now, many people have SSDs, which have TRIM support. For safe, weekly TRIM service on SSDs and all other devices that enable TRIM support:

systemctl enable fstrim.timer
-Enabling multilib and Arch AUR community repositories
If you are running a 64bit system then you need to enable the multilib repository. Open the pacman.conf file using nano:

> nano /etc/pacman.conf

Scroll down and un-comment the multilib repo:

> [multilib]
> Include = /etc/pacman.d/mirrorlist

While we are still inside pacman.conf file, let’s also add the AUR repo so we can easily install packages from AUR. Add these lines at the bottom of the file:

> [archlinuxfr]
> SigLevel = Never
> Server = http://repo.archlinux.fr/$arch

then save and close, and update with:

> pacman -Sy

optionally, update the system with -Syu instead. I will explain at the end of this guide how to use the AUR.
-Root password and user setup
First set a password for root with:

> passwd
Now add a default user with:

> useradd -m -g users -G wheel,storage,power -s /bin/bash someusername
and set a pass for that user:

> passwd someusername

-Setting up sudoers
Now we have to edit the sudoers file to give this user the much needed sudo powers. Don\92t open this file with a regular editor; it must be edited with visudo command.

> EDITOR=nano visudo

Uncomment:

> %wheel ALL=(ALL) ALL

And we’re going to make sudoers require typing the root password instead of their own password by adding:

> Defaults rootpw

Save and close the file.
Lastly we’re going to install bash-completion which makes it easier with auto-complete of commands and package names.

> pacman -S bash-completion

-Installing the bootloader
Now to actually make our install boot without a usb drive.
First we need to double check to see if our EFI variables have already been mounted or not

> mount -t efivarfs efivarfs /sys/firmware/efi/efivars

If this says already mounted just ignore and keep following this guide.
As we are doing an UEFI installation of archlinux we are going to use Gummiboot as our boot manager, which has now been incorporated into bootctl/systemd-boot.

> bootctl install

Now you will need to manually create a configuration file to add an entry for Arch Linux to the gummiboot manager:

> nano /boot/loader/entries/arch.conf

Type the following, make sure sdaX is your root partition (mine is sda3):

> title Arch Linux
> linux /vmlinuz-linux
> initrd /initramfs-linux.img

save and exit.
Next, we need to add the PARTUUID of the /root partition to our bootloader configuration.
You can get a list of hard drive partitions by typing
> lsblk
look for the partition that looks like this:

> sda3        8:19   0 229.5G  0 part /

In particular the one that has / as the mountpoint. In our case it was sda3. To add this to our boot loader we next type:

> echo "options root=PARTUUID=$(blkid -s PARTUUID -o value /dev/sdb3) rw" >> /boot/loader/entries/arch.conf

Please note it is VERY important that you type >> and NOT >, >> adds a line to a file while > overwrites the file.
NOTE: Intel processors

Processor manufacturers release stability and security updates to the processor microcode. While microcode can be updated through the BIOS, the Linux kernel is also able to apply these updates during boot. These updates provide bug fixes that can be critical to the stability of your system. Without these updates, you
may experience spurious crashes or unexpected system halts that can be difficult to track down.

Users of CPUs belonging to the Intel Haswell and Broadwell processor families in particular must install these microcode updates to ensure system stability. But all Intel users should install the updates as a matter of course.

For AMD processors the microcode updates are available in linux-firmware, which is installed as part of the base system. No further action is needed for AMD users.

If you own a Haswell processor or higher (such as a 4770k or 6700k):

> pacman -S intel-ucode

then we will have to update our gummiboot by adding another initrd line for intel-ucode as follows:

> sudo nano /boot/loader/entries/arch.conf

add the intel-ucode line so it looks like this:

> initrd /intel-ucode.img
> initrd /initramfs-linux.img
Next, let’s make sure our wired network connection is automatically turned on when we start the machine:

First lets see what network adapters were working with via

> ip link

Ignore the one listed as lo, that is loopback and is always listed.
mine was listed as
enp5s0
so we’re going to enable it via systemctl

> sudo systemctl enable dhcpcd@enp5s0.service

for the sake of having a simple graphical interface that works across Desktop Environments, we’ll also install and enable NetworkManager:

> sudo pacman -S NetworkManager
> sudo systemctl enable NetworkManager.service
That’s it! If you need wireless, please see my wireless guide here:

2016 Arch Linux NetworkManager / Wifi Setup guide.


Now before we reboot, we are also going to set up our graphics drivers. Reason being Arch’s kernel is set to use nouveau drivers by default for nvidia cards, and some cards don’t work properly and will cause a freeze/hang (such as my gtx 980 ti..)

I assume you know which GPU you are using. Arch wiki has done a great job at documenting which drivers you need to install for your hardware.

We are using the dkms module so that we don’t have to reinstall nvidia drivers for every different kernel if we decide to try another kernel later. To install dkms modules we need the headers for our kernel:

> sudo pacman -S linux-headers
I have an Nvidia GTX 980 TI, so my latest drivers will just be nvidia. I also want the multilib drivers and all dependencies required for the nvidia package, so I will install all of the following:

> sudo pacman -S nvidia-dkms libglvnd nvidia-utils opencl-nvidia lib32-libglvnd lib32-nvidia-utils lib32-opencl-nvidia nvidia-settings
We will also want to set nvidia drm kernel modules:

> sudo nano /etc/mkinitcpio.conf

find MODULES=
change it so it looks like:

> MODULES="nvidia nvidia_modeset nvidia_uvm nvidia_drm"

We also need to make sure these are loaded during boot, so next we do this:

> sudo nano /boot/loader/entries/arch.conf

find the line that looks like this:

> options root=PARTUUID=bada2036-8785-4738-b7d4-2b03009d2fc1 rw 

add nvidia-drm.modeset=1
like this

> options root=PARTUUID=bada2036-8785-4738-b7d4-2b03009d2fc1 rw nvidia-drm.modeset=1
Lastly, we need to make a pacman hook, so that any time the kernel is updated, it automatically adds the nvidia module. This will save us a LOT of headache later on.

> sudo nano /etc/pacman.d/hooks/nvidia.hook

add this content, save, and close:

> [Trigger]
> Operation=Install
> Operation=Upgrade
> Operation=Remove
> Type=Package
> Target=nvidia

> [Action]
> Depends=mkinitcpio
> When=PostTransaction
> Exec=/usr/bin/mkinitcpio -P
Now you should be able to reboot into your system without the USB stick!
Type the following commands and then remove the USB stick:

> exit
> umount -R /mnt
> reboot
You should now be able to boot into your system and be at a black login screen.

——————————-

***PART 3:Making it user friendly and adding a desktop environment***

-Installing touchpad support
If you are on a laptop and need touchpad support also type

> sudo pacman -S xf86-input-synaptics

-Installing 3D support
now we add 3d support

> sudo pacman -S mesa

-Installing X server display manager
now install X, which is our display manager

> sudo pacman -S xorg-server xorg-apps xorg-xinit xorg-twm xorg-xclock xterm
Now we need to test if X runs:

> startx
If you get a screen with a few terminals and a clock, it works! You can type “exit” in the terminals to drop back to the command line.

-Installing a desktop manager
Let’s give ourselves an actual interface to log in to, so we will finally feel at home with our new Arch install:

> sudo pacman -S plasma sddm
> sudo systemctl enable sddm.service

This is for KDE, I use KDE for a few reasons:
1. Ease of use for beginners
2. Aesthetics are nice
3. The window management options are amazing for gaming. If you need to make a game hide a border or not minimize in full screen mode, you can do so by opening the game, then hitting Alt+F3
After this, you should be able to reboot and safely log into your system!

POST INSTALLATION TWEAKS (IMPORTANT – YOU WILL WANT TO DO THESE):

Once you are in KDE, if you use NVIDIA you will want to get rid of some screen tearing. Open a terminal, run:

> sudo nvidia-settings

Click X Server Display Configuration
For each monitor:
click “Advanced”
check Force Composition Pipeline
check Force Full Composition Pipeline
Then click Save to X Configuration File, and quit.

Wwe need a way to install packages from the AUR. If you recall, earlier we added the AUR repos to our pacman.conf. Now we need a way to install them. yaourt is a program that comes in Arch’s main packages, so we need that to get started:


> sudo pacman -S yaourt
What the AUR is, is a collection of USER created packages for Arch Linux users to pull from. These can be game installers, programs compiled from git repositories, beta drivers, or other programs that aren’t included in Arch’s main repos. That being said, it is VERY smart to LOOK at the PKGBUILD of a package before installing it, to see what it does, where it installs things, and if the auther has added any notes about installing it. If you find a package on the AUR you want to install, you use yaourt the same way as pacman. AUR packages can be found here: https://aur.archlinux.org/

Many AUR packages compile from source, so you will want to speed up compile times. I have a few small tips for that as well. First, you will need to know the amount of cores your processor has, and will need to know if your processor supports Hyperthreading or SMT. For example, if you own an i7 4770k, you have 4 cores and support hyperthreading, essentially giving you 8 cores. Ryzen 1700x 8 cores, 16 threads – counts as 16 cores. If your processor does NOT support SMT/hyperthreading, such as an AMD FX-8350, you would just need to know the core number. FX-8350 has 8 cores.

First install ccache:

> sudo pacman -S ccache
Next lets enable ccache and set our makeflags for makepkg:

> sudo nano /etc/makepkg.conf

find BUILDENV=
remove the ! in front of ccache so the line looks like this:

> BUILDENV=(!distcc color ccache check !sign)

find MAKEFLAGS=
change it so it looks similar to

> MAKEFLAGS="-j17 -l16"

replace 17 with your number of cores +1, and 16 with your number of cores. At the time of this edit, I currently am using a Ryzen 1700x, which is why I use -j17 -l16
save, close
Next we need to make sure ccache and makeflags are set at all times in case we compile something without using a package manager:
> nano ~/.bashrc
add these lines, save, close:

> export PATH="/usr/lib/ccache/bin/:$PATH"
> export MAKEFLAGS="-j17 -l16"

again, replace 17 with your number of cores +1, and 16 with your number of cores. At the time of this edit, I currently am using a Ryzen 1700x, which is why I use -j17 -l16
DONE! ENJOY YOUR NEW ARCH LINUX SYSTEM! I HOPE THIS GUIDE HELPED!!!
