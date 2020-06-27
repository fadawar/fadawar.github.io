---
layout: post
title: How to mount NTFS partition on Ubuntu
---

Recently I had problem with my NTFS partition on Ubuntu 16.10. The problem was I couldn't copy file to it 
and also Insync (great cross-platform client for Google Drive) was creating duplicated directories
because it couldn't edit those directories. Another symptom was unability to set execute bit. Well, without
root privileges.

Solution is actually simple. First unmount your parition with `sudo umount /path/to/your/partition`. Then you have to change 
your `/etc/fstab` file. Just add `uid`, `gid` parameter on the line
with your ntfs partition with the value `1000`. I have also added `umask` with the value `022` paramater as 
you can see on the image (partition `/media/D`).

![fstab file in vim](http://i.imgur.com/nGkr4WU.png)

Parameter `uid` means your user id. You can find it with command `id -u`. 
Parameter `gid` is - as you already suspect - for group id. You can find it with `id -g`. By default it's `1000`.
`umask` is mask for your directories and files permission.

So the final line with your looks like this:
```
UUID=782B9DA75524938A /media/D        ntfs    defaults,umask=022,uid=1000,gid=1000 0
```

Don't forget to save your file and reboot or mount your partition back with `sudo mount /path/to/your/partition`.

**Do you have any questions? Write it down in the comments.**
