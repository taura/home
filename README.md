# Set up a new drive

See https://www.baeldung.com/linux/encrypt-partition

## find disk drives you want to encrypt
```
lsblk
``` 
Let's say it is `/dev/sdb`

## make a partition table
```
sudo parted /dev/sdb mklabel gpt
```

## make a partition and give it a name
Let's say we want to create a partition `/dev/sdb1` in `/dev/sdb` and name it `enc_part`

When you have a GUI, easiest is 
```
gparted /dev/sdb
```
and use default partition start/size provided. 
Just don't forget to give it a name (e.g., `enc_part`).

If you do not have a GUI, use parted
```
$ sudo parted -a optimal /dev/sdb mkpart
Partition name?  []? enc_part
File system type?  [ext2]? ext4
Start? 0%
End? 100%
```

make sure the partition is properly created.
```
$ sudo parted /dev/sdb print
Model: Samsung PSSD T5 EVO (scsi)
Disk /dev/sdb: 8002GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name      Flags
 1      33.6MB  8002GB  8002GB               enc_part

```

## check the partition label
```
sudo blkid /dev/sdb1
```
and make sure it shows `PARTLABEL=enc_part`

## format the drive
It asks you to set a password. Enter one.
```
sudo cryptsetup luksFormat --type luks1 /dev/sdb1
```

## make a mapped drive
```
sudo cryptsetup luksOpen /dev/sdb1 enc_drv
sudo mkfs -t ext4 /dev/mapper/enc_drv
sudo e2label /dev/mapper/enc_drv enc_home
```

# Set up a Linux host

## /etc/crypttab
```
enc_drv PARTLABEL=enc_part none
```

## /etc/fstab
```
LABEL=enc_home	/home	ext4	defaults,errors=remount-ro,noatime,nofail,x-systemd.device-timeout=10s	0	1
```

# test (optional)
```
sudo mkdir -p /tmp/enc_drv
sudo mount /dev/mapper/enc_drv /tmp/enc_drv
```

# when done
```
sudo cryptsetup luksClose enc_drv
```

