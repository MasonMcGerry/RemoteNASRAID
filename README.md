# NAS | RAID | RPi
### Current Features:
>RAID 1 configured with `mdadm`
>
>NAS configured `netatalk` on AFP
>
>This allows you to drag and drop files to and from your computer and the RPi's Storage over your WiFi
>
>RAID ensures that if one disk fails your data is still 'alive' on the other disk(s)

### Mininum Hardware Required:
>Raspberry Pi
![rpi_withcase](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/rpi_withcase.JPG?raw=true)
>
>microSD card, and adapter as needed (microSD -> SD card adapter)
![microSD_adapter](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/microSD_adapter.JPG?raw=true)
>
>2 USB thumbdrive/flashdrive's (drives/disks from now on)
>>similar size is best, that way you aren't wasting the larger drives capacity
>
>Your computer, desktop or laptop, for setting up
>
>Router, and Modem to connect to the internet and to connect to RPi
>
>Ethernet cable, to connect RPi directly to router/modem

### Minimum Software Downloads:
#### On your computer:
>Balena Etcher

#### On the RPi:
>mdadm
>
>netatalk

## Step 1 - Headless Install of RPi
>You can skip to 'Step 2' if you already have your RPi up and running

1. Download Raspberry Pi OS Lite (as it's called now) we don't need a GUI hence Lite
https://www.raspberrypi.org/downloads/raspberry-pi-os/

![rpi_downloads](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/rpi_downloads.png?raw=true)

![rpi_os](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/rpi_os.png?raw=true)

![os_todownload](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/os_todownload.png?raw=true)

2. Download Balena Etcher, for flashing the OS to the microSD
https://www.balena.io/etcher/
>you will have to install the `balenaEtcher-*.dmg` but that should be straight forward, it's an app like any other

![downloadbalena](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/downloadbalena.png?raw=true)

3. Connect microSD to your computer, via adapter
![SDadapterInComputer](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/SDadapterInComputer.JPG?raw=true)
4. Open Balena Etcher app and follow the Balena Etcher GUI steps

    ![selectImage](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/selectImage.png?raw=true)
    - select the RPiOS Lite Image, .img file
    ![selectImage2](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/selectImage2.png?raw=true)
    - select the disk to flash it to and "Flash!" it, be VERY CAREFUL, DO NOT FLASH your HD
    ![selectDrive](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/selectDrive.png?raw=true)
    ![flash](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/flash.png?raw=true)
    ![flashing](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/flashing.png?raw=true)
    - you can look at the Disk Utility app (native to Apple) and make sure it's the microSD you are flashing and NOT your computers HD
    ![checkThatDriveCorrect](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/checkThatDriveCorrect.png?raw=true)
    
        >notice how the APPLE SD Card Reader Media is only 32.01GB, that is one indicator for me because that is the specific size othe microSD I'm using
    
5. Once the flashing is complete, eject the microSD, I use Disk Utility to accomplish this
![ejectSDadapter](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/ejectSDadapter.png?raw=true)
6. Reconnect the microSD to your computer and open up your Shell, I use the Terminal app
![diskutilafterflash](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/diskutilafterflash.png?raw=true)
7. Create the `ssh` file by typing the following command into the Terminal

`touch /Volumes/boot/ssh`

>I would make sure your computers file structure is like mine before using this command, i.e. you have root `/`, `Volumes/`, and inside `Volumes/`, you have `boot/`, so `/Volumes/boot/` would be the full path, this shouldn't be a problem if you are using a MacOS
>
>all that is necessary for this part to work is that a blank file with no file extension named `ssh` is placed in the `boot` directory of your microSD

8. Eject the microSD
9. Connect the microSD to the RPi, connect ethernet from router to RPi, and power on the RPi
10. From  your computer `ssh` into the RPi
>both devices must be connected to the same network, and be on the same subnet

`ssh pi@raspberrypi.local`

>the password is `raspberry`, so change that as soon as you log in, you can change the password with the command `passwd`
>
>I would also change the host name of the RPi which can be done in `sudo raspi-config` or by editing the `sudo nano /etc/hosts` and `sudo nano /etc/hostname` files to have a host name of your choosing instead of the name `raspberrypi`
>>then restart the RPi to solidify these changes with `sudo reboot`

## Step 2 - Install `mdadm` and configure RAID
1. Log into your RPi
2. Update && Upgrade, then download mdadm, if you don't have it already

`sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get install mdadm -y`

3. Connect both drives to your RPi
>make sure there is NO DATA on the drives, we'll be wiping them
4. Identify the drives with `lsblk`
>your output should look something like the below
    
```
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda           8:0    1  3.8G  0 disk 
sdb           8:16   1  7.5G  0 disk 
mmcblk0     179:0    0 29.8G  0 disk 
├─mmcblk0p1 179:1    0  256M  0 part /boot
└─mmcblk0p2 179:2    0 29.6G  0 part /
```

> as you may notice the SIZE is 3.8G and 7.5G, this mismatch is fine, it just means we can only store, and mirror data in the size of the smallest disk on the RAID, 3.8G, in other words the largest disk is only utilizing as much space as the smallest disk

5. Wipe each disk with zeros
>this will take some time depending on the size, so for experimentation use small drives
>
>you can wipe both simultaneously, just open up another shell (Terminal) and wipe the other disk in there, so they'll be running concurrently in different windows
>
`sudo dd if=/dev/zero of=/dev/sdX bs=1M status=progress`
>where the `X` in `sdX` is either `a` in `sda`, or `b` in `sdb`, etc for all drives you use in this experiment
>
> there are best practices for `bs=someValue` but for this tutorial we'll keep it simple
>
>output will look something like this
>
`212860928 bytes (213 MB, 203 MiB) copied, 5 s, 42.3 MB/s`
>
>until it's complete, then it'll look like below

```
dd: error writing '/dev/sda': No space left on device
7651+0 records in
7650+0 records out
8021606400 bytes (8.0 GB, 7.5 GiB) copied, 860.048 s, 9.3 MB/s
```

>`error writing '/dev/sda': No space left on device` means that there were no more spaces to write in `0's`, so it's all finished don't be concerneds

6. Create RAID 1
>if you are adding more than 2 devices then change `--raid-devices=n` accordingly, where `n` is the number of devices

`sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdX /dev/sdY`

>you should get a similar output as below

```
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 3912704K
mdadm: largest drive (/dev/sdb) exceeds size (3912704K) by more than 1%
Continue creating array?
```
- type `yes` and press enter, you should see something like what's below

```
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

6. Monitor the syncing

`watch cat /proc/mdstat`

>to view the progress
>
>to get out of it this view hold ctrl and press C
>
>the output should be similar to what is below
    
```
Personalities : [raid1] 
md0 : active raid1 sdb[1] sda[0]
      3912704 blocks super 1.2 [2/2] [UU]
      [====>................]  resync = 20.8% (815424/3912704) finish=5.3min speed=9557K/sec
      
unused devices: <none>
```

>and like the below when complete

```
Personalities : [raid1] 
md0 : active raid1 sdb[1] sda[0]
      3912704 blocks super 1.2 [2/2] [UU]
      
unused devices: <none>
```

7. Create ext4 filesystem on RAID 1
`sudo mkfs.ext4 -F /dev/md0`

>you'll get a similar output to the below, just give it a moment, depending on disk size

```
mke2fs 1.44.5 (15-Dec-2018)
Creating filesystem with 978176 4k blocks and 244800 inodes
Filesystem UUID: f3e988bf-2f9a-4637-9ae6-f4e451b18526
Superblock backups stored on blocks: 
    32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done 
```

8. Create mount point for the RAID

`sudo mkdir -p /mnt/md0`

9. Mount the RAID

`sudo mount /dev/md0 /mnt/md0`

10. Check that drive is mounted and how much space it has
`sudo mdadm -D /dev/md0`

>output should look similar to below

`Array Size : 3912704 (3.73 GiB 4.01 GB)`

```
/dev/md0:
           Version : 1.2
     Creation Time : Tue Sep  1 01:10:40 2020
        Raid Level : raid1
        Array Size : 3912704 (3.73 GiB 4.01 GB)
     Used Dev Size : 3912704 (3.73 GiB 4.01 GB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Tue Sep  1 01:26:42 2020
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : cloudstore:0  (local to host cloudstore)
              UUID : ee7f51b7:d27ab855:b9c5e4a0:88ad4ef1
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8        0        0      active sync   /dev/sda
       1       8       16        1      active sync   /dev/sdb
```

`Used Dev Size : 3912704 (3.73 GiB 4.01 GB)`
>from what I've read this means unused space, which makes sense since it's all fresh, I wouldn't read into it too much


##  Step 3 - Install `netatalk` and configure NAS

1. Install netatalk, you should be logged into the RPi for these steps as well

`sudo apt-get install netatalk -y`

2. Configure netatalk

`sudo nano /etc/netatalk/afp.conf`

>defines home directory, allows users to access ‘home’ folder, I named mine `My RAID` with a directory name of `my_raid`
>
>`/media/my_raid/shared` is the full path to the accessible portion of the RAID
>
>unless you are using a different distribution than I am RPi OS Lite then you shouldn't have to worry about the path being incorrect, you can name the `my_raid` portion whatever you like

- edit the configuration file to reflect the below entry, either by the modifiying `[My AFP Volume]` entry or by adding your own, uncommented entry
>if it is 'commented out' then the configuration won't be read by the program

```
[My RAID]
path = /media/my_raid/shared
; are comments, again semi-colons are comments
```	

3. Make directory RAID

`sudo mkdir /media/my_raid`

4. List out RAID with disks UUID's

`lsblk --fs | grep md0`
- copy UUID
>mine looks like
>
>`f3e988bf-2f9a-4637-9ae6-f4e451b18526`
>
>will be similar to the below

```
└─md0       ext4                           f3e988bf-2f9a-4637-9ae6-f4e451b18526    3.4G     0% /mnt/md0
└─md0       ext4                           f3e988bf-2f9a-4637-9ae6-f4e451b18526    3.4G     0% /mnt/md0
```

5. Add copied UUID as entry to fstab

`sudo nano /etc/fstab`
- add `UUID=f3e988bf-2f9a-4637-9ae6-f4e451b18526  /media/my_raid  ext4  defaults  0  2` to the bottom of fstab, make sure the UUID is equal to what you copied in step 4

`/etc/fstab` file will look like the following

```
proc            /proc           proc    defaults          0       0
PARTUUID=853f83f4-01  /boot           vfat    defaults          0       2
PARTUUID=853f83f4-02  /               ext4    defaults,noatime  0       1
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that
```

6. Restart `netatalk`

`sudo systemctl restart netatalk`


7.  Mount RAID

`sudo mount -a`

8. Make shared directory and change permissions

`sudo mkdir /media/my_raid/shared`

`sudo chmod 777 /media/my_raid/shared`

9. `cat /proc/mdstat` is a great way to ensure the RAID is functioning but you can also inspect with the `lsblk --fs | grep md0` command
>if all disks are mirroring properly you should see the `%` change for all disks when you modify the data, i.e. add or remove files from the RAID
>e.g.

```
└─md0       ext4                           f3e988bf-2f9a-4637-9ae6-f4e451b18526    3.4G     0% /media/my_raid
└─md0       ext4                           f3e988bf-2f9a-4637-9ae6-f4e451b18526    3.4G     0% /media/my_raid

└─md0       ext4                           f3e988bf-2f9a-4637-9ae6-f4e451b18526    2.1G    36% /media/my_raid
└─md0       ext4                           f3e988bf-2f9a-4637-9ae6-f4e451b18526    2.1G    36% /media/my_raid
```

## Step 4 - Connect to RAID Storage from Computer
1. Open a Finder window, method differs for MacOS 10.13 and 10.14 or higher
- In the Finder's sidebar
![sidebar](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/sidebar.png?raw=true)
    - For 10.13 and below, under `Shared` look for the RPi's `hostname`
    
    ![shared](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/shared.png?raw=true)
    - For 10.14 and above, under `Locations` look for the RPi's `hostname` or `Network`
        - if you don't see the `hostname` under `Locations` then select `Network`, under `Locations` and select the `hostname`
        ![Network](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/Network.png?raw=true)

2. Select and open `hostname`
![cloudspace](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/cloudspace.png?raw=true)

> You'll be put into a Finder window that states it is 'Not Connected'
![notConnected](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/notConnected.png?raw=true)

3. Select `Connect As` and fill out credentials
![loginToRAID](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/loginToRAID.png?raw=true)

>Now you should be logged in, you can open `My Raid` and start storing data in it
![connected](https://github.com/MasonMcGerry/RemoteNASRAID/blob/master/Images/connected.png?raw=true)
>You can drag from your desktop to the RAID storage, and from the RAID storage to your desktop

### All Done!
- I'll be adding more to this README.md
    - how to remove and add disks
    - VPN setup to access from anywhere like a 'cloud'
    - using Windows to setup/administer
    - troubleshooting
- As I learn more I will create a more in depth ReadMe

# Sources:

https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-ubuntu-16-04

https://how-to.fandom.com/wiki/How_to_wipe_a_hard_drive_clean_in_Linux

https://www.cyberciti.biz/faq/linux-unix-dd-command-show-progress-while-coping/

https://zackreed.me/adding-an-extra-disk-to-an-mdadm-array/

https://www.tecmint.com/create-raid0-in-linux/

https://unix.stackexchange.com/questions/379705/mkfs-ext4-f-how-to-avoid-data-loss-in-the-future

https://unix.stackexchange.com/questions/549864/dev-tmpfs-question

https://pimylifeup.com/raspberry-pi-afp/

https://www.ionos.com/help/server-cloud-infrastructure/dedicated-server-for-servers-purchased-before-102818/rescue-and-recovery/software-raid-status-monitoring-linux/

https://www.kremkow.com/2018/07/raspberry-pi-raid-file-server/

https://serverfault.com/questions/347843/use-cases-for-mdadm-create-vs-mdadm-build

https://raid.wiki.kernel.org/index.php/A_guide_to_mdadm

https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0

https://desertbot.io/blog/headless-raspberry-pi-3-bplus-ssh-wifi-setup

https://www.raspberrypi.org/documentation/configuration/wireless/headless.md

https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes

https://www.ssh.com/ssh/keygen/

https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2

https://howtoraspberrypi.com/how-to-raspberry-pi-headless-setup/

https://weworkweplay.com/play/automatically-connect-a-raspberry-pi-to-a-wifi-network/

https://askubuntu.com/questions/98702/how-to-unblock-something-listed-in-rfkill

https://osric.com/chris/accidental-developer/2017/09/wifi-on-raspberry-pi-3/

https://markdownlivepreview.com

https://geek-university.com/raspberry-pi/change-raspberry-pis-hostname/

https://www.ducea.com/2009/03/08/mdadm-cheat-sheet/

https://serverfault.com/questions/650086/does-the-bs-option-in-dd-really-improve-the-speed

https://redhatlinux.guru/2016/08/24/how-to-remove-mdadm-raid-devices/

https://askubuntu.com/questions/703340/mdadm-disk-already-part-of-an-array

https://www.commandlinefu.com/commands/view/1335/monitor-linuxmd-raid-rebuild

https://ubuntuforums.org/showthread.php?t=2135395

https://medium.com/markdown-monster-blog/getting-images-into-markdown-documents-and-weblog-posts-with-markdown-monster-9ec6f353d8ec

https://stackoverflow.com/questions/52876285/embedding-an-image-stored-in-github

https://stackoverflow.com/questions/14494747/add-images-to-readme-md-on-github

https://www.linuxquestions.org/questions/red-hat-31/what-does-used-dev-size-mean-mdadm-detail-767177/

https://unix.stackexchange.com/questions/52215/determine-the-size-of-a-block-device

https://helpcenter.graphisoft.com/knowledgebase/86497/

https://support.apple.com/guide/mac-help/set-up-file-sharing-on-mac-mh17131/mac

https://eclecticlight.co/2019/12/09/can-you-still-use-afp-sharing/

https://askubuntu.com/questions/215505/how-do-you-monitor-the-progress-of-dd

https://www.digitalocean.com/community/tutorials/how-to-manage-raid-arrays-with-mdadm-on-ubuntu-16-04

https://superuser.com/questions/471327/how-to-force-mdadm-to-stop-raid5-array

https://beamtic.com/dd-no-space-left-on-device

https://support.apple.com/guide/mac-help/connect-shared-computers-file-servers-a-mchlp1140/10.13/mac/10.13

https://support.apple.com/guide/mac-help/connect-mac-shared-computers-servers-mchlp1140/10.15/mac/10.15
