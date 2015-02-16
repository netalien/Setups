Gentoo Install
==============


### Partitions
```sh
mkfs.ext4 /dev/sda3
```

+ xfs for /
```sh
mkfs.xfs /dev/sda6
```

+ btrfs for /
```sh
mkfs.btrfs -f -L LINUX /dev/sda6
mkdir /mnt/newroot
mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag /dev/sda6
/mnt/newroot
btrfs subvol create /mnt/newroot/root
mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=root
/dev/sda6 /mnt/gentoo
```

Swap setup
```sh
mkswap /dev/sda5 && swapon /dev/sda5
```

### Mount all
```sh
mkdir /mnt/gentoo
mount /dev/sda5 /mnt/gentoo
mkdir /mnt/gentoo/boot
mount /dev/sda3 /mnt/gentoo/boot
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
```

### chroot
```sh
cd /mnt/gentoo
mount -t proc none proc
mount --rbind /sys sys
mount --rbind /dev dev
cp /etc/resolv.conf etc 
env -i HOME=/root TERM=$TERM chroot . bash -l
export PS1="(chroot) $PS1"
```

### fstab

  /dev/sda3               /boot           ext4            nodiratime      1 2
  /dev/sda5               none            swap            sw              0 0
  # xfs 
  /dev/sda6               /               xfs             nodiratime      0 1
  # btrfs
  /dev/sda6		/		btrfs defaults,noatime,compress=lzo,autodefrag,subvol=root	0 0

### timezone
```sh
ln -sf /usr/share/zoneinfo/America/Caracas /etc/localtime
```

### Configure make.conf

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
  VIDEO_CARDS="radeon fglrx"          
  INPUT_DEVICES="mouse keyboard evdev"


### hostname
```sh
sed -i '2s/localhost/desktop.normandy.lab/g' /etc/conf.d/hostname
```

### keymap
```sh
sed -i '3s/us/es/g' /etc/conf.d/keymaps
```

### clock
```sh
sed -i '5s/UTC/local/g' /etc/conf.d/hwclock
```

### language
```sh
echo 'es_VE.UTF-8 UTF-8' >> /etc/locale.gen
echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen
locale-gen
eselect locale set 3
```

### set desktop profile (check with eselect profile list)
```sh
eselect profile set 3
```

### kernel install
```sh
emerge gentoo-sources 
cd /usr/src/linux
make && make modules_install
cp arch/x86_64/boot/bzImage /boot/kernel-3.12.13-gentoo
cp System.map /boot/System.map-3.12.13-gentoo
```

### grub
```sh
emerge grub os-prober
```

### generate grub2 confs
```sh
grub2-install --no-floppy /dev/sda
grub2-mkconfig -o /boot/grub/grub.cfg
```

### Network configuration
```sh
emerge bridge-utils

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

### swap optimization
```sh
echo 'vm.swappiness = 20' > /etc/sysctl.d/swappiness.conf
```

### x11
```sh
echo 'app-shells/zsh doc examples' >> /etc/portage/package.use
echo 'x11-drivers/ati-drivers -qt4' >> /etc/portage/package.use
echo 'media-libs/libcanberra gtk3' >> /etc/portage/package.use
echo 'www-client/google-chrome' >> /etc/portage/package.keywords
echo 'dev-lang/python sqlite' >> /etc/portage/package.use

emerge -p xorg-x11 fluxbox vim gvim zsh chrony portage-utils gentoolkit eix \
ntfs3g firefox-bin thunderbird-bin pgadmin3 terminator pyxdg google-chrome \
google-perftools colordiff rsyslog logrotate lvm2
```

### fcron
```sh
echo 'sys-process/fcron -mta' >> /etc/portage/package.use
emerge fcron
emerge --config sys-process/fcron
rc-update add fcron default
```

### lvm
```sh
rc-update add lvm default
```

### remmina
```sh
echo 'net-misc/remmina' >> /etc/portage/package.keywords
echo 'net-misc/remmina freerdp ssh' >> /etc/portage/package.use
echo 'net-misc/freerdp' >> /etc/portage/package.keywords
emerge remmina
```

### logs
```sh
rc-update add rsyslog default
```

### user
```sh
useradd -m -s /bin/zsh -g users -G wheel,audio,cdrom,video,cdrw,usb netalien
passwd netalien
```

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

### extra apps
```sh
echo 'sys-process/glances' >> /etc/portage/package.keywords
echo 'sys-process/nmon' >> /etc/portage/package.keywords
echo 'net-analyzer/nmap -gtk -ncat ndiff nmap-update nping' >> /etc/portage/package.use
echo 'media-libs/gd fontconfig' >> /etc/portage/package.use
echo 'net-libs/libpcap netlink' >> /etc/portage/package.use
echo 'dev-vcs/subversion -webdav-neon' >> /etc/portage/package.use

emerge -puN iotop atop htop dstat glances sshpass mtr nethogs iptraf-ng nmap \ 
    arpwatch iputils swatch sysstat iftop latencytop nload procinfo-ng acct \
    tcpdump bind-tools rdiff-backup ntop p7zip fio nmon lsof
```

### logout
```sh
exit
cd
umount -l /mnt/gentoo/dev /mnt/gentoo/sys /mnt/gentoo/proc /mnt/gentoo/boot /mnt/gentoo
```

### qemu/kvm libvirt nested virtualization
```sh
echo 'options kvm-intel nested=1' > /etc/modprobe.d/kvm_intel_nested.conf

echo 'app-emulation/qemu' >> /etc/portage/package.keywords
echo 'app-emulation/libvirt' >> /etc/portage/package.keywords
echo 'app-emulation/virt-manager' >> /etc/portage/package.keywords
echo 'app-emulation/libguestfs' >> /etc/portage/package.keywords
echo 'app-emulation/libguestfs-appliance' >> /etc/portage/package.keywords
echo 'sys-firmware/seabios' >> /etc/portage/package.keywords
echo 'sys-apps/dtc' >> /etc/portage/package.keywords
echo 'app-admin/augeas' >> /etc/portage/package.keywords
echo 'dev-ml/ocaml-fileutils' >> /etc/portage/package.keywords
echo 'dev-ml/ocaml-gettext' >> /etc/portage/package.keywords
echo 'dev-ml/camomile' >> /etc/portage/package.keywords
echo 'app-emulation/qemu gtk spice sasl tci tls usb xfs' >> /etc/portage/package.use
echo 'app-emulation/spice client sasl' >> /etc/portage/package.use
echo 'app-emulation/libvirt fuse lvm lxc pcap sasl virt-network' >> /etc/portage/package.use
echo 'x11-libs/gtk+ introspection' >> /etc/portage/package.use
echo 'x11-libs/gdk-pixbuf introspection' >> /etc/portage/package.use
echo 'x11-libs/vte introspection'  >> /etc/portage/package.use
echo 'app-emulation/libvirt-glib introspection' >> /etc/portage/package.use
echo 'dev-libs/atk introspection' >> /etc/portage/package.use
echo 'net-misc/spice-gtk introspection gtk3' >> /etc/portage/package.use
echo 'sys-auth/polkit introspection' >> /etc/portage/package.use
echo 'net-libs/gtk-vnc introspection gtk3' >> /etc/portage/package.use
echo 'x11-libs/pango introspection' >> /etc/portage/package.use
echo 'app-misc/hivex perl' >> /etc/portage/package.use

emerge libvirt virt-manager libguestfs

cp /media/data02/Linux/ConfigFiles/polkit/20-libvirt.rules /etc/polkit-1/rules.d
chmod 644 /etc/polkit-1/rules.d/20-libvirt.rules
```

### moc
```sh
echo 'media-sound/moc aac ffmpeg musepack' >> /etc/portage/package.use
emerge moc
```

### vlc
```sh
echo 'media-video/vlc a52 aac bluray musepack matroska wma-fixed x264' >> /etc/portage/package.use
echo 'media-libs/libbluray aacs' >> /etc/portage/package.use
echo 'sys-libs/zlib minizip' >> /etc/portage/package.use
emerge vlc
```

### fontset
```sh
euse -E infinality
for i in {1..40}; do eselect fontconfig disable $i; done
emerge -uND world
for i in {32..36}; do eselect fontconfig enable $i; done # 62-croscore*
eselect fontconfig enable 52-infinality.conf
emerge liberation-fonts droid dejavu
for i in {29..31}; do eselect fontconfig enable $i; done # google's droid fonts
eselect fontconfig enable 33 # liberation fonts
eselect infinality set infinality
eselect lcdfilter set infinality
eselect fontconfig enable 70-no-bitmaps.conf # due to problem with google chrome v33
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
