## WIP - This is incomplete and scattered info at this testing phase

# Efforts to get Windows 10 booting and using OEM Licensing as a guest via passthrough with QEMU on the same metal as it originates.
 _Specs: an existing UEFI, Secure Boot'd, Windows 10 installed on it's own dedicated SSD_
   ##### Goal: 
   * Booting direct from the HD 
   * Utilising a no-touch Secure Boot process, via passthrough, or separate keys
   * Utilise the OEM Licensing Info for Windows 10, via passthrough, or bust

#### Current Status: 
  * grub menu appears, but no reference to uefi boot code, so it won't boot the windows partition (Linux installed on second hard drive must have written to boot sector...)
  * might need to readjust the diskpath ID to point directly at the win partition number, or perhaps regen uefi boot stamps from the new VM uefi bios leaving current disk alone
  * Details appear as efforts proceed


## Attach Pass Through Disk[^1]:
#### Point physical hard drive to Qemu virtual machine to boot:
 ##### TLDR:
 * Build your base VM with virsh/virt-manager, when you get to the HD part, 
 * Use the HD path by device-id.
 #### Look for path to use by ID with:
  _you will be able to use something like this_
  ```
  ls -lhtra /dev/disk/by-id/
  /dev/disk/by-id/ata-ST2000DX001-1CM164_Z1E783H2
  ```

 #### Identify Disk:
  * _Before adding a physical disk to host make note of vendor, serial so that you'll know which disk to share in /dev/disk/by-id/_
  ```
  lshw -class disk -class storage
             *-disk
                  description: ATA Disk
                  product: ST3000DM001-1CH1
                  vendor: Seagate
                  physical id: 0.0.0
                  bus info: scsi@3:0.0.0
                  logical name: /dev/sda
                  version: CC27
                  serial: Z1F41BLC
                  size: 2794GiB (3TB)
                  configuration: ansiversion=5 sectorsize=4096
  ```
  _Note that device names like /dev/sdc should never be used, as this can change between reboots. Use the stable /dev/disk/by-id paths instead._ 
  
 #### Check by listing all of that directory then look for the disk added by matching serial number from lshw and the physical disk:
 ```
 ls -l /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC
 lrwxrwxrwx 1 root root 9 Jan 21 10:10 /dev/disk/by-id/ata-ST3000DM001-1CH166_Z1F41BLC -> ../../sda
 ```
 or try... 
 ```
 ls -l /dev/disk/by-id | grep Z1F41BLC
 ```

#### List disk by-id with lsblk:
 * The lsblk is pre-installed, you can print and map the serial and WWN identifiers of attached disks using the following two commands:
 ```
 lsblk -o +MODEL,SERIAL,WWN
 ls -l /dev/disk/by-id/
 ```
 * Extended one liner to get the path directly:
 ```
 lsblk |awk 'NR==1{print $0" DEVICE-ID(S)"}NR>1{dev=$1;printf $0" ";system("find /dev/disk/by-id -lname \"*"dev"\" -printf \" %p\"");print "";}'|grep -v -E 'part|lvm'

 NAME                         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT DEVICE-ID(S)
 sda                            8:0    0   7.3T  0 disk   /dev/disk/by-id/wwn-0x5000c500c35cd719 /dev/disk/by-id/ata-ST8000DM004-2CX188_ZCT1DNY1
 sdb                            8:16   1    29G  0 disk   /dev/disk/by-id/usb-Generic_STORAGE_DEVICE-0:0
 sdc                            8:32   0 931.5G  0 disk   /dev/disk/by-id/usb-JMicron_Generic_0123456789ABCDEF-0:0
 sdd                            8:48   0   1.8T  0 disk   /dev/disk/by-id/wwn-0x5000c500661eeebd /dev/disk/by-id/ata-ST2000DX001-1CM164_Z1E783H2
 ```
 * Make-lsblk-list-devices-by-id:
 ```
 find /dev/disk/by-id/ -type l|xargs -I{} ls -l {}|grep -v -E '[0-9]$' |sort -k11|cut -d' ' -f9,10,11,12

 /dev/disk/by-id/ata-ST8000DM004-2CX188_ZCT1DNY1 -> ../../sda
 /dev/disk/by-id/wwn-0x5000c500c35cd719 -> ../../sda
 /dev/disk/by-id/usb-Generic_STORAGE_DEVICE-0:0 -> ../../sdb
 /dev/disk/by-id/usb-JMicron_Generic_0123456789ABCDEF-0:0 -> ../../sdc
 /dev/disk/by-id/ata-ST2000DX001-1CM164_Z1E783H2 -> ../../sdd
 /dev/disk/by-id/wwn-0x5000c500661eeebd -> ../../sdd
 ```



## OEM bundling [^2]:
 ##### TLDR: 
 ```
 cat /sys/firmware/acpi/tables/SLIC > slic.bin
 cat /sys/firmware/acpi/tables/MSDM > msdm.bin
 strings /sys/firmware/acpi/tables/MSDM > produkey.txt
 dmidecode -t 0 -u | grep  $'^\t\t[^"]' | xargs -n1 | perl -lne 'use bytes; printf "%c", hex($_)' > smbios_type_0A.bin
 dmidecode -t 1 -u | grep  $'^\t\t[^"]' | xargs -n1 | perl -lne 'use bytes; printf "%c", hex($_)' > smbios_type_1A.bin
 mkdir Elitebook840G4_SLIC
 mv *.bin Elitebook840G4_SLIC/
 tar -cvzf Elitebook840G4_SLIC.tar.gz Elitebook840G4_SLIC/
 cp Elitebook840G4_SLIC/ /usr/share/seabios/
 ```


### //TODO: Configure Guest Domain XML with proper acpi argument options[^3]: 
 ##### TLDR: 
 ```
 ```
 


[^1]: https://pve.proxmox.com/wiki/Passthrough_Physical_Disk_to_Virtual_Machine_(VM)
[^2]: https://gist.github.com/Informatic/49bd034d43e054bd1d8d4fec38c305ec
[^3]: 



