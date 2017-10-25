* Server was on Ubuntu 14.04.   Now migrating to arch linux
* O/S was on 2 LVM device.   One extended from the other
* New SSD drive added for Arch O/S, but the extended ubuntu LVM drive was also added. 
* Because ubuntu LVM drive has not been repartitioned yet, boot up error are showing about missing LVM.   They can be disreguarded at this time.

* Powered my other drives

The UUID were the same as the previous ubuntu config.
```sh
# cat old_fstab >> /etc/fstab
# vi /etc/fstab -- cleaned up data from old_fstab
# cd /mnt
# mkdir <folder1 folder2 folderN>  --create all mount points used in fstab for new drives
# mount -a
```
## Test drives and data is there

## reboot to test mounts are working
```sh
# shutdown -r now
```

-- Setup NFS
```sh
# vi /etc/exports
```
-- Example of allowing 2 networks to the share.
```sh
/mnt/foldertosharepath/foldertoshare 192.168.1.0/24(permissions here)  192.168.2.0/24(permissions here)
```
```sh
# exportsfs -arv
```
-- Enabling Docker
```sh
# tee /etc/modules-load.d/loop.conf <<< "loop"
# modprobe loop 
# pacman -S docker
# systemctl start docker
# systemctl enable docker
# usermod -aG docker <username>
```
-- nzbget
```sh
# docker create --name=nzbget -v /etc/localtime:/etc/localtime:ro -v /mnt/<folder>/docker/nzbget:/config -v /mnt/<folder>/downloads:/downloads -e PGID=1000 -e PUID=1000 -p <my port>:6789 linuxserver/nzbget:latest
```
-- before starting nzbget mv nzbget.conf to a new name.
```sh
# docker start nzbget
# docker logs nzbget
# docker stop nzbget
```
-- compare old nzbget.conf with new one.    merge the two together to keep the servers configs
-- add docker start nzbget to crontab

-- did a systemctl service to start all my dockers.

-- tvheadhend
-- Installed via yaourt.  Non-git version with hdhomerun
```sh
# yaourt tvheadend
```
-- I don't think the following section is required
```sh
# pacman -S xmltv
```
-- pacman all missing perl module for mc2xml zap2it.pl script
```sh
# pacman -S perl-html-parser
# pacman -S perl-http-cookies
# pacman -S perl-http-message
# pacman -S perl-json
# pacman -S perl-json-xs
# pacman -S perl-lwp-protocol-https
# pacman -S site_perl
```
-- END of I don't think this is required

```sh
# cd /home/<home user>
# git clone https://github.com/Rigolo/tv-grab-file.git
# cp tv_grab_file /usr/bin
```
-- change path to xmltv file after the cat command.
```sh
# vi /usr/bin/tv_grab_file
```
-- reboot

# Snapraid
```sh
$ yaourt snapraid
$ sudo vim /etc/fstab
```

```sh
# Parity Disks
UUID=<UID> /mnt/parity1 ext4 defaults 0 2
UUID=<UID> /mnt/parity2 ext4 defaults 0 2

# Snapraid Disks
UUID=<UID> /mnt/disk1 ext4 defaults 0 2
UUID=<UID> /mnt/disk2 ext4 defaults 0 2
UUID=<UID> /mnt/disk3 ext4 defaults 0 2
UUID=<UID> /mnt/disk4 ext4 defaults 0 2
UUID=<UID> /mnt/disk5 ext4 defaults 0 2
```
$ sudo vim /etc/snapraid.conf
```sh
parity /mnt/parity1/snapraid.parity
2-parity /mnt/parity2/snapraid.2-parity

content /var/snapraid.content
content /mnt/disk1/snapraid.content
content /mnt/disk2/snapraid.content

disk d1 /mnt/disk1/
disk d2 /mnt/disk2/
disk d3 /mnt/disk3/
disk d4 /mnt/disk4/
disk d5 /mnt/disk5/

exclude *.unrecoverable
exclude /tmp/
exclude *.pst 
exclude /lost+found/
blocksize 256
```


Automating Scripts systemd timers
```sh
# cd /etc/systemd/system
# touch snapraid.service
# touch snapraid.timers
```
Content of snapraid.service
```sh
[Unit]
Description=Run Snapraid daily script

[Service]
Type=oneshot
ExecStart=/root/scripts/snapraid.sh 

[Install]
WantedBy=default.target
cat: 1: No such file or director
```
Content of snapraid.timer
Run at 2h45AM daily
```sh
[Unit]
Description=Run Snapraid Daily Script

[Timer]
OnCalendar=*-*-* 02:45:00
Persistent=true

[Install]
WantedBy=timers.target
```

Update EPG for TVHeadENd
```sh
[Unit]
Description=Update EPG from zap2it

[Service]
Type=oneshot
ExecStart=/root/scripts/update_epg.sh 

[Install]
WantedBy=default.target
```
EPG Timer
```sh
[Unit]
Description=EPG Updater using zap2xml.pl script

[Timer]
OnCalendar=*-*-* 00:45:00
Persistent=true

[Install]
WantedBy=timers.target
```

Startup script - For any script that runs at boot up only
```sh
[Unit]
Description=My Startup Script

[Service]
Type=oneshot
ExecStart=/usr/local/bin/startup.sh 

[Install]
WantedBy=default.target
```
startup.sh content
```sh
#!/bin/sh
docker start nzbget
docker start sonarr
docker start couchpotato
docker start mysql
```
-- Mailing
