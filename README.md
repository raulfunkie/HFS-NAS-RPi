# HFS-NAS-RPi
 Set up an HFS+ disk as a NAS using Raspberry Pi + SAMBA (For File sharing). aka. Seedbox

# Steps
These main steps are taken from [this guide](https://sammit.net/how-to-make-a-raspberry-pi-nas-using-samba-hfs/) with some minor modifications here and there to accommodate some changes or requirements that I might need.

## Disable Automount
First we need to disable automount in `pcmanfm.conf`:
* `cd ~/.config/pcmanfm/LXDE-pi/`
* `sudo nano pcmanfm.conf`
```
[volume]
mount_on_startup=0
mount_removable=0
```
* Save and Exit (<kbd>ctrl</kbd> + <kbd>X</kbd> > <kbd>Y</kbd> > <kbd>Enter</kbd>)

## Mount the correct HFS+ partition
* Run `sudo apt-get update` and then `sudo apt-get upgrade`
* Add HFS+ support `sudo apt-get install hfsprogs`
* HFS+ uses GPT (GUID Partition Table) so we need to `sudo apt-get install gdisk`
* Create a directory to mount the drive to. In my case I use DEF: `sudo mkdir /media/DEF`
* Make sure the drive is detected, run `sudo fdisk -l`
* Check which partition is the correct one to mount: `sudo gdisk -l /dev/sda`
* The partition to mount in my case is `/dev/sda1`
* Find the UUID of your drive: `sudo blkid`
* Copy the UUID, in my case it is: `175bce55-43ef-32d6-adca-ce3fa28945fb`
* Mount the drive `sudo mount -t hfsplus -o force,rw /dev/sda1 /media/DEF`
* Now that the drive is mounted, let's make sure the RPi mounts it everytime it boots. Let's do `sudo nano /etc/fstab`
* Add a new line to fstab: `UUID="175bce55-43ef-32d6-adca-ce3fa28945fb"     /media/DEF      hfsplus uid=1000,nofail,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=1ms,force,rw      0       0`
* Save and Exit (<kbd>ctrl</kbd> + <kbd>X</kbd> > <kbd>Y</kbd> > <kbd>Enter</kbd>)
* run `sudo chmod -R 777 /media/DEF`
* Test if the disk is writtable: `echo “test” > /media/DEF foo.txt`

If after a reboot you face some errors such as "device is read-only" or something among those lines, run this command, reboot and try again:
`sudo fsck.hfsplus /dev/sda2`.

## Install SAMBA
* `sudo apt-get install samba samba-common-bin`
* Backup the original config file before making any changes: `sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bkup`
* `sudo nano /etc/samba/smb.conf`
* Remove the "#" symbol from the line `security = user`. (If it's not there, then just add it.)
* Go to the bottom of the config file and add your share details:
```
[ShareName] comment = Raspberry Pi NAS
path = /media/DEF/
valid users = @users
force group = users
create mask = 0660
directory mask = 0771
read only = no
```
* Save and Exit (<kbd>ctrl</kbd> + <kbd>X</kbd> > <kbd>Y</kbd> > <kbd>Enter</kbd>)
* Restart SAMBA: `sudo /etc/init.d/samba restart`

## Create SAMBA user
* `sudo useradd *username* -m -G users`
* `sudo passwd *username*`
* You’ll be prompted to type in the password twice to confirm. Now we can add this user as a samba user. 
* `sudo smbpasswd -a *username*`