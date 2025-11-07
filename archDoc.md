Preinstallation: 
Download Arch HTTP mirror off https://archlinux.org/download/#checksums. I chose the BitTorrent download. After that I had to actually download BitTorrent which I did through the link https://www.bittorrent.com/products/win/web-free/#comparison-panels and clicking the free version. After completing the setup wizard I use it to download the mirror.

I then went to VMware and created a new virtual machine where I put in the mirror and chose Linux as the OS and Other Linux 6.x kernel 64-bit as the version. I gave it 2GBs of ram and 20 GB of HDD space and chose default for all other options.

I then booted up the VM and clicked enter to enter the ISO boot environment. 
After using the command cat /sys/firmware/efi/fw_platform_size  to verify the boot mode it said no such file or directory so I added firmware=”efi” to the second line of the .vmx file and went to setting and changed the bootmode to UEFI and then powered back on the VM. When I redid the command it said 64.

At first the internet wasn’t working so I changed the VM settings to allow bridging. I then used the command ping [ping.archlinux.org](http://ping.archlinux.org/) to verify the connection and it worked. To confirm it more I used the command timedatectl to check if the time was correctly synced and it was.

To partition correctly to make 500mb for EFI under sda1 and the rest for arch os under sda2 I started with command fdisk -l to check the current layout and all 20gb were under sda. To change this I used fdisk /dev/sda. I entered g to make a GPT partition table, n to make the partition, clicked enter for partition number 1, enter for first sector, and +500m for 500mb. Then I entered t and 1 to set type to EFI. I then pressed n to make another partition and enter 3 times to use number 2, to use the first sector and to use the rest of the disk as these are all the default. I clicked w to make changes and exit and the command fdisk -l again to check whether the changes worked. 

To format the partitions I started with the commands mkfs.fat -F32 /dev/sda1 to format sda1 as FAT32 and mkfs.ext4 /dev/sda2 to format sda2 into ext. To mount the file systems I used mount /dev/sda2 /mnt to mount sda2 and the commands mkdir -p /mnt/boot and mount /dev/sda1 /mnt/boot to mount sda1.

Installation: I used command pacstrap -K /mnt base linux linux-firmware to install essential packages.

Configure the system: 
Used command genfstab -U /mnt >> /mnt/etc/fstab to get necessary file systems
Used arch-chroot /mnt to directly interact with the file system. 
To set the correct time zone I started with the command ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime then hwclock --systohc to set the time, and timedatectl to verify. It didn’t work at first so I used timedatectl set-timezone America/Chicago and it worked.
For network configuration I started with the command echo MichaelVM > /etc/hostname to name the computer. Then i used nano /etc/hosts and added the line  MichaelVM.localdomain  MichaelVM. Then I used pacman -S networkmanager to install network manager and systemctl enable NetworkManager to enable it and ping -c 3 [archlinux.org](http://archlinux.org/) to test it.
Initially I did this step wrong and had no internet in the post install version so to remedy this during the boot phase I had to manually select the version of the boot to load and chose UEFI. This allowed me to redo this step to be correct and have internet access.
I used the passwd command to create a password 
To install a boot loader I used the command bootctl install. Then I used nano /boot/loader/entries/arch.conf to open file and added lines title Arch Linux, linux /vmlinuz-linux,initrd  /initramfs-linux.img,options root=/dev/sda2 rw to it. Then i used nano /boot/loader/loader.conf and added lines default arch.conf, timeout 3, console-mode max, editor no. Then I used command bootctl status to check. 


Reboot: I typed ctrl D and reboot. I then logged back in.

Post-intall:
  GUI: Used command pacman -S lxqt xorg lightdm lightdm-gtk-greeter and clicked enter a bunch of times for all defaults. I then used systemctl enable lightdm.service to enable display manager and pacman -S xorg to install a graphics sy1stem. After rebooting I ended up with a working GUI.
 
  User Account: I started by using the pacman account to install sudo. Then I used commands useradd -m -G wheel -s /bin/bash [name] for both michael and codi creating home directories for each and adding them to a wheel group. Then I used the passwd command to give both a password. I then used to command EDITOR=nano visudo to edit the file and unncommented the line # %wheel ALL=(ALL:ALL) ALL to give those accounts sudo permissions. I then used su to get into both accounts and tried sudo pacman -Syu to see whether it’d give the password prompt then download the item like it should. This confirmed that the accounts work properly.

Shell: I decided to download ksh which involved the command pacman -S ksh. I then typed ksh to switch to that shell and echo $0 to confirm I was using that shell.

  SSH: I started with the command pacman -S openssh to download ssh. I then used the commands systemctl enable sshd and systemctl start sshd to get it running and then the command systemctl status sshd to confirm that its running
  Color Coding: I started with the command pacman -S --needed bash-completion coreutils grep less util-linux to install color support then nano /etc/pacman.conf to edit the file which allowed me to remove # from the line #Color to allow me to enable color in pacman. Then I used the command nano ~/.bashrc and added these lines.
    This allows for color to be added by adding the line PS1='\[\e[1;32m\]\u@\h \[\e[1;34m\]\w\[\e[0m\]\$ ' under these lines leads to a green username and blue working directories. Then I used source ~/.bashrc to save it and the shell had color coding.

  Boot into GUI: As everything was already installed in previous steps I only needed to use command systemctl enable lightdm to automatically boot into the GUI. After rebooting the system everything worked correctly.
  Add allies to shell config file: I started by following the example shown in “https://www.cyberciti.biz/tips/bash-aliases-mac-centos-linux-unix.html” by typing alias c='clear’ which is allow you to clear the screen by typing c. This command worked properly but this method only saves aliases during the current login so I used nano ~/.bashrc to be able to save the rest permanently and started trying out the aliases listed in the article.
alias ll='ls -la': Command ll displays the long listing format of ls.
alias ..='cd ..', alias ...='cd ../../', ect: The cd commands gets you out of the current directory and into parent ones based on how many ../ are in the command so I made it so you’d only have to type periods to get to parent directories with the amount for periods deciding how far it goes.
alias ports='netstat -tulanp': ports command now lists all TCP and UDP ports.
alias nbashrc=‘nano ~/.bashrc’: Makes the command to edit ~/.bashrc simpler.
  After adding all of these commands I used source ~/.bashrc to be able to use all the commands and they all were successful.
