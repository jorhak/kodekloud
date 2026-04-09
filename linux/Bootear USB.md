# Listar dispositivo
```
sudo fdisk -l
```

OUTPUT
```
Disco /dev/sdc: 3.8 GiB, 4083351552 bytes, 7975296 sectores
```


```
lsblk
```

OUTPUT
```
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS  
loop0    7:0    0 167.8M  1 loop /snap/jimbodicomviewer/9  
loop1    7:1    0   105M  1 loop /snap/core/17272  
loop2    7:2    0  48.1M  1 loop /snap/snapd/25935  
loop3    7:3    0 105.1M  1 loop /snap/core/17284  
loop4    7:4    0  48.4M  1 loop /snap/snapd/26382  
sda      8:0    0 931.5G  0 disk    
├─sda1   8:1    0  1024M  0 part /boot/efi  
├─sda2   8:2    0     8G  0 part    
├─sda3   8:3    0   500G  0 part /  
└─sda4   8:4    0   320G  0 part /home  
sdb      8:16   0 149.1G  0 disk    
sdc      8:32   1   3.8G  0 disk    
└─sdc1   8:33   1   3.8G  0 part /media/jon/PRACTICE  ##AQUI
sr0     11:0    1  1024M  0 rom     
zram0  251:0    0   7.6G  0 disk [SWAP]
```

# Desmontar USB
```
sudo umount /dev/sdc1
```

```
lsblk
```

# Bootear USB
```
sudo dd bs=4M if=/home/jon/Downloads/minios-docker-xfce-minunux-aufs-amd64.iso of=/dev/sdc conv=fdatasync
```