Gentoo Install
==============

### Useful from sysrescuecd
Check current monitor settings
`sh
xset -q
`
Prevent scree blanking
`sh
xset s off
`
Disable monitor energy start (DPMS)
`sh
xset -dpms
`

### Partitions
+ /boot
```sh
mkfs.ext4 /dev/sda4
```

+  / (without btrfs)
```sh
mkfs.ext4 /dev/sda5
```

+ / (with btrfs)
```sh
mkfs.btrfs -f -L LINUX /dev/sda5
mkdir /mnt/newroot
mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag /dev/sda5
/mnt/newroot
btrfs subvol create /mnt/newroot/root
mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=root
/dev/sda6 /mnt/gentoo
```

Swap setup
```sh
mkswap /dev/sda6 && swapon /dev/sda6
```

### Mount all
```sh
mkdir /mnt/gentoo
mount /dev/sda5 /mnt/gentoo
mkdir /mnt/gentoo/boot
mount /dev/sda4 /mnt/gentoo/boot
mkdir /mnt/gentoo/boot/efi
mount /dev/sda1 /mnt/gentoo/boot/efi
mkdir /mnt/gentoo/boot/efi/EFI/gentoo
```

### Fix date
```sh
date MMDDhhmmYYYY
```

### Get files
```sh
cd /tmp && wget -c http://www.gtlib.gatech.edu/pub/gentoo/releases/amd64/autobuilds/current-stage3-amd64/stage3-amd64-20140619.tar.bz2
wget -c http://www.gtlib.gatech.edu/pub/gentoo/snapshots/portage-latest.tar.xz
```

### install stage3
```sh
cd /mnt/gentoo && tar xjpf /mnt/data/Linux/stage3-*.tar.bz2 
```

### install portage
```sh
cd /mnt/gentoo/usr && tar xpf /tmp/portage-latest.xz
emerge --sync
```

### chroot

Note: rslave is need for systemd only
```sh
mount -t proc proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
cp -L /etc/resolv.conf /mnt/gentoo/etc/  
env -i HOME=/root TERM=$TERM chroot /mnt/gentoo bash -l
export PS1="(chroot) $PS1"
```

### get UUID
`sh
for i in 1 {4..7}; do blkid /dev/sda$i; done
`

### fstab
`
cat > /etc/fstab <<_EOF
# /boot on /dev/sda4
UUID=c1008c3b-ab1a-4784-b474-c2a63fd5b82c /boot           ext4    defaults        0       2
# /boot/efi on /dev/sda1 
UUID=2ED0-0732  /boot/efi       vfat    umask=0077      0       1
# / on /dev/sda5
UUID=85de2405-ac8c-42d1-b380-336636d8b02e /               ext4     noatime         0       1
# swap on /dev/sda6
UUID=a4ae23d8-af36-4170-93c3-2a928cc71c04 none            swap    sw              0       0
# /home on /dev/sda7
UUID=a9b71978-3228-4127-bd42-3e54e2b8897c /home           xfs     noatime         0       2
_EOF
`


### timezone
```sh
ln -sf /usr/share/zoneinfo/America/Caracas /etc/localtime
```

### Configure make.conf

This was for an Intel Core 2 Duo E8400
`sh
cat > /etc/portage/make.conf <<_EOF
CFLAGS="-march=native -O2 -fpredictive-commoning -fexcess-precision=fast -mfpmath=sse -pipe"  
CXXFLAGS="${CFLAGS}"  
LDFLAGS="-Wl,-O1 -Wl,--as-needed -Wl,--hash-style=both -Wl,-z,now"  
CHOST="x86_64-pc-linux-gnu"  
MAKEOPTS="-j2"  
# accepts  
ACCEPT_KEYWORDS=""  
ACCEPT_LICENSE="*"  
# USEs  
USE="bash-completion bindist -bluetooth ccache -gdbm -gtk3 -handbook -introspection \  
  -ipv6 kde -ldap lzma lzo mmx pch python -spell sse sse2 sse3 ssse3 vim-syntax \  
  xattr -zeroconf"  
# emerge opts  
EMERGE_DEFAULT_OPTS="--verbose"  
# hardware opts  
LINGUAS="en en_US en_EN"  
VIDEO_CARDS="radeon"  
INPUT_DEVICES="mouse keyboard evdev"  
_EOF
`

This for AMD APU
`sh
cat > /etc/portage/make.conf <<_EOF
CFLAGS="-march=native -O2 -pipe"
CXXFLAGS="${CFLAGS}"
CHOST="x86_64-pc-linux-gnu"  
MAKEOPTS="-j2"  
# accepts  
ACCEPT_KEYWORDS=""  
ACCEPT_LICENSE="*"  
# USEs  
USE=""
# emerge opts  
EMERGE_DEFAULT_OPTS="--verbose"  
# hardware opts  
LINGUAS="en en_US"  
VIDEO_CARDS="radeon"  
INPUT_DEVICES="mouse keyboard evdev"  
_EOF
`

### set desktop profile (check with eselect profile list)

This sets the desktop/gnome/systemd profile
```sh
eselect profile set 5
```

### Create mtab symlink

Note: need by systemd
`sh
ln -sf /proc/self/mounts /etc/mtab
`

### kernel install
```sh
emerge gentoo-sources 
cd /usr/src/linux
make menuconfig
make && make modules_install
make install
cp /boot/vmlinuz-4.0.5-gentoo /boot/efi/EFI/gentoo/bootx64.efi
cp /boot/vmlinuz-4.0.5-gentoo /boot/efi/EFI/Boot/bootx64.efi
emerge linux-firmware
```

### Update packages
`sh
euse -D ldap
euse -E vim-syntax

emerge -pUDN @world
`

### grub (BIOS mbr)

```sh
emerge grub os-prober
```

generate grub2 confs
```sh
grub2-install --no-floppy /dev/sda
grub2-mkconfig -o /boot/grub/grub.cfg
```

### Efibootmgr (UEFI)

Install efibootmgr
`sh
emerge efibootmgr
`

Check current boot entries
`sh
efibootmgr
`

Delete obsolete boot entries (the bootnum *-b* comes from entries like Boot0004* something...)
`sh
efibootmgr -b 4 -B
`

Add gentoo's boot entry, params are: --create --disk --partition --label --loader
`sh
efibootmgr -c -d /dev/sda -p 1 -L "Gentoo" -l "\efi\gentoo\bootx64.efi" -u "root=/dev/sda5 init=/usr/lib/systemd/systemd"
`

### swap optimization
```sh
echo 'vm.swappiness = 20' > /etc/sysctl.d/swappiness.conf
```

### Base packages
`sh
echo 'app-shells/zsh doc examples' >> /etc/portage/package.use/zsh
echo 'app-editors/vim perl python' >> /etc/portage/package.use/vim 

emerge -p vim zsh portage-utils gentoolkit eix mlocate logrotate xfsprogs btrfs-progs ntfs3g chrony
`

### Cron

Enable vixie
`sh
systemctl enable vixie-cron
`

### Add user
```sh
useradd -M -s /bin/zsh -g users -G wheel,audio,cdrom,video,cdrw,usb netalien
passwd netalien
```


### Set systemd-networkd

`sh
emerge bridge-utils
`

Create bridge for systemd-networkd
`sh
cat > /etc/systemd/network/bridge0.netdev <<_EOF
[NetDev]
Name=bridge0
Kind=bridge
_EOF
`

`sh
cat > /etc/systemd/network/50-bridge0.network <<_EOF
[Match]
Name=bridge0

[Network]
Address=192.168.1.2/24
Gateway=192.168.1.1
DNS=192.168.1.1
_EOF

cat > /etc/systemd/network/51-slave.network <<_EOF
[Match]
Name=eth0

[Network]
Bridge=bridge0
_EOF

cat > /etc/systemd/system/bridge-stp-settings.service <<_EOF
[Unit]
Description=Sets stp settings on bridge
After=systemd-networkd.service
Requires=systemd-networkd.service

[Service]
Type=oneshot
ExecStart=/sbin/brctl setfd bridge0 0
ExecStart=/sbin/brctl sethello bridge0 1

[Install]
WantedBy=multi-user.target
_EOF

systemctl enable systemd-networkd
systemctl start systemd-networkd
systemctl enable bridge-stp-settings
systemctl start bridge-stp-settings

ln -snf /run/systemd/resolve/resolv.conf /etc/resolv.conf
systemctl enable systemd-resolved
systemctl start systemd-resolved
`

### Set hostname
`sh
hostnamectl set-hostname desktop.edmv.lab
`

### Set locale info
`sh
localectl set-locale LANG=en_US.utf8
localectl set-keymap en
localectl set-x11-keymap en
`

### Set time and date
`sh
timedatectl set-timezone America/Caracas
timedatectl set-ntp true
timedatectl --adjust-system-clock set-local-rtc 1

systemctl enable chronyd && systemctl start chronyd
`

### Network configuration

Note: only when not using systemd-networkd
```sh
cat > /etc/conf.d/net <<_EOF
brctl_br0="setfd 0
sethello 2
stp on"

bridge_br0="enp1s0"

config_enp1s0="null"

config_br0="dhcp"

rc_net_br0_need="net.enp1s0"
_EOF

cd /etc/init.d/
ln -s net.lo net.br0
ln -s net.lo net.enp1s0
rc-update add net.br0 default
```


### x11
```sh
euse -D bluetooth qt4
euse -E ffmpeg

echo 'dev-libs/keybinder python' >> /etc/portage/package.use/keybinder
echo 'dev-db/postgresql perl python threads' >> /etc/portage/package.use/postgresql
echo 'media-plugins/alsa-plugins' >> /etc/portage/package.use/alsa-plugins
echo 'x11-libs/vte python' >> /etc/portage/package.use/vte
echo 'x11-xterms/terminator -gnome' >> /etc/portage/package.use/terminator
echo 'app-editors/gvim -gnome perl python' >> /etc/portage/package.use/gvim 
echo 'media-sound/moc musepack wavpack' >> /etc/portage/package.use/moc
echo 'app-admin/sudo offensive -sendmail' >> /etc/portage/package.use/sudo
echo 'dev-vcs/git doc -webdav' >> /etc/portage/package.use/git
echo 'www-client/google-chrome' >> /etc/portage/package.keywords/google-chrome
echo 'x11-xterms/terminator' >> /etc/portage/package.keywords/terminator
echo 'gnome-extra/zenity -webkit' >> /etc/portage/package.use/zenity

emerge -p xorg-x11 i3 i3status i3lock firefox-bin thunderbird-bin pgadmin3 \
terminator google-chrome google-perftools colordiff gvim numlockx feh moc \
scrot irssi tmux git subversion dmenu alsa-utils zenity mupdf libtxc_dxtn \
sysutils jq collectl 
```

### Extra config for i3

This is done because the default terminal app is terminator which set WM_CLASS to *x-terminal-emulator* no matter the console app running within it
```sh
cat > /usr/bin/tmocp <<_EOF
terminator -c mocp -e mocp
_EOF

chmod +x /usr/bin/tmocp

cat > /usr/bin/tirssi <<_EOF
terminator -c irssi -e irssi
_EOF

chmod +x /usr/bin/tirssi

cat > /usr/bin/tglances <<_EOF
terminator -c glances -e glances
_EOF

chmod +x /usr/bin/tglances

cat > /usr/bin/thtop <<_EOF
terminator -c htop -e htop
_EOF

chmod +x /usr/bin/thtop
```

### Openvpn
`sh
echo 'net-misc/openvpn down-root iproute2' >> /etc/portage/package.use/openvpn

emerge openvpn

mkdir -p /etc/openvpn/keys/{one,two}
cp /media/data00/OpenVPN/{one,two}.conf /etc/openvpn/
chmod 644 /etc/openvpn/{one,two}.conf
cp /media/data00/OpenVPN/one/* /etc/openvpn/keys/one/  
cp /media/data00/OpenVPN/two/* /etc/openvpn/keys/two/  
chmod 600 /etc/openvpn/keys/one/*
chmod 600 /etc/openvpn/keys/two/*

ln -s /usr/lib/systemd/system/openvpn@.service /usr/lib/systemd/system/openvpn@one.service
ln -s /usr/lib/systemd/system/openvpn@.service /usr/lib/systemd/system/openvpn@two.service
`

### remmina
```sh
echo 'net-misc/remmina freerdp ssh' >> /etc/portage/package.use/remmina
echo 'net-misc/remmina **' >> /etc/portage/package.keywords/remmina
echo 'net-misc/remmina' >> /etc/portage/package.unmask/remmina
emerge -1 remmina freerdp
```

### extra apps
```sh
echo 'sys-process/glances' >> /etc/portage/package.keywords/glances
echo 'net-analyzer/nmap ndiff nmap-update nping' >> /etc/portage/package.use/nmap
echo 'net-analyzer/openbsd-netcat' >> /etc/portage/package.keywords/openbsd-netcat
echo 'media-libs/gd fontconfig' >> /etc/portage/package.use/gd
echo 'app-arch/p7zip rar -wxwidgets' >> /etc/portage/package.use/p7zip

emerge -puN iotop atop htop dstat glances sshpass mtr nethogs iptraf-ng nmap \ 
    arpwatch iputils sysstat iftop nload procinfo-ng tcpdump bind-tools \
    p7zip fio lsof telnet-bsd openbsd-netcat pip the_silver_searcher strace
```

### lvm
```sh
emerge lvm2

rc-update add lvm default
```

### rsyslog (if not using systemd)
`sh
emerge rsyslog

rc-update add rsyslog default
`

### ATI Drivers
`sh

echo 'x11-drivers/ati-drivers -qt4' >> /etc/portage/package.use

emerge ati-drivers
`

### If kde
```sh
echo 'dev-qt/qtgui gtkstyle' >> /etc/portage/package.use
echo 'dev-python/PyQt4 webkit declarative sql script' >> /etc/portage/package.use
echo 'media-plugins/gst-plugins-meta ffmpeg musepack wavpack' >> /etc/portage/package.use
echo 'media-video/ffmpeg bluray vaapi vdpau' >> /etc/portage/package.use
emerge kdebase-meta
emerge oxygen-gtk oxygen-gtk:3
rc-update add consolekit default
sed -i '10s/xdm/kdm/g' /etc/conf.d/xdm
rc-update add xdm default
```


### logout
```sh
exit
cd
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -l /mnt/gentoo{/boot,/sys,/proc,}
reboot
```

### Docker

Install
`sh
cat > /etc/portage/package.keywords/docker <<_EOF
app-emulation/docker
_EOF

cat > /etc/portage/package.use/docker <<_EOF
app-emulation/docker btrfs overlay -device-mapper
_EOF
`

Enable btrfs driver
`sh
mkdir /etc/systemd/system/docker.service.d

cat > /etc/systemd/system/docker.service.d/docker.conf <<_EOF
[Service]
ExecStart=
ExecStart=/usr/bin/docker -d -H fd:// --graph /media/data02/docker --storage-driver btrfs
_EOF

systemctl daemon-reload
`

### qemu/kvm libvirt nested virtualization
```sh
echo 'options kvm-amd nested=1' > /etc/modprobe.d/kvm_amd_nested.conf

echo 'net-libs/gtk-vnc python' >> /etc/portage/package.use/gtk-vnc
echo 'net-misc/spice-gtk python usbredir' >> /etc/portage/package.use/spice-gtk
echo 'app-emulation/libvirt-glib python' >> /etc/portage/package.use/libvirt-glib
echo 'net-dns/dnsmasq script' >> /etc/portage/package.use/dnsmasq
echo 'app-emulation/qemu spice usbredir virtfs xattr' >> /etc/portage/package.use/qemu
echo 'app-emulation/libvirt pcap virt-network' >> /etc/portage/package.use/libvirt

emerge libvirt virt-manager 

cp /media/data00/Linux/Setups/ConfigFiles/polkit/20-libvirt.rules /etc/polkit-1/rules.d
chmod 644 /etc/polkit-1/rules.d/20-libvirt.rules
```

### vlc
```sh
echo 'media-video/vlc' >> /etc/portage/package.keywords/vlc
echo 'dev-qt/qtx11extras' >> /etc/portage/package.keywords/qtx11extras
echo 'dev-qt/qtcore' >> /etc/portage/package.keywords/qtcore     
echo 'dev-qt/qtgui' >> /etc/portage/package.keywords/qtgui 
echo 'dev-qt/qtdbus' >> /etc/portage/package.keywords/qtdbus
echo 'dev-qt/qtwidgets' >> /etc/portage/package.keywords/qtwidgets
echo 'sys-libs/zlib minizip' >> /etc/portage/package.use/zlib
echo 'dev-libs/libpcre pcre16' >> /etc/portage/package.use/libpcre

echo 'media-video/vlc -cdda -encode -gnome libass matroska modplug musepack qt5' >> /etc/portage/package.use/vlc

emerge vlc
```

### fontset
```sh
euse -E infinality
for i in {1..32}; do eselect fontconfig disable $i; done
emerge -uND @world
for i in {25..28}; do eselect fontconfig enable $i; done # 62-croscore*
eselect fontconfig enable 52-infinality.conf
emerge liberation-fonts droid dejavu
for i in {29..31}; do eselect fontconfig enable $i; done # google's droid fonts
eselect fontconfig enable 33 # liberation fonts
eselect infinality set infinality
eselect lcdfilter set infinality
```

### overlays
```sh
echo 'dev-vcs/subversion' >> /etc/portage/package.keywords
echo 'app-portage/layman subversion' >> /etc/portage/package.use

emerge layman

echo '/var/lib/layman/make.conf' >> /etc/portage/make.conf

layman -L
layman -a <overlay> # add
layman -d <overlay> # delete
layman -S # sync all packages from overlay
```
