# /etc/crypttab
enc_drv PARTLABEL=enc_part

# /etc/fstab
LABEL=home	/home	ext4	defaults,errors=remount-ro,noatime,nofail,x-systemd.device-timeout=10s	0	1
