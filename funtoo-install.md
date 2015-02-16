Funtoo Install
==============

### Disk partitions
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
mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag /dev/sda6 /mnt/newroot
btrfs subvol create /mnt/newroot/root
mount -t btrfs -o defaults,noatime,compress=lzo,autodefrag,subvol=root /dev/sda6 /mnt/funtoo
```

Swap setup
```sh
mkswap /dev/sda5 && swapon /dev/sda5
```

Mount all
```sh
mkdir /mnt/funtoo
mount /dev/sda6 /mnt/funtoo
mkdir /mnt/funtoo/boot
mount /dev/sda3 /mnt/funtoo/boot
```

### Fix date
```sh
date MMDDhhmmYYYY
```

### Install stage3
```sh
cd /tmp && wget -c http://build.funtoo.org/funtoo-current/x86-64bit/core2_64/stage3-latest.tar.xz

cd /mnt/funtoo && tar xpf /tmp/stage3-*.tar.xz
```

### chroot
```sh
cd /mnt/funtoo
mount -t proc none proc
mount --rbind /sys sys
mount --rbind /dev dev
cp /etc/resolv.conf etc 
env -i HOME=/root TERM=$TERM chroot . bash -l
export PS1="(chroot) $PS1"
```

### Download portage tree
```sh
emerge --sync
```

### fstab

> /dev/sda3               /boot           ext4            nodiratime      1 2
  /dev/sda5               none            swap            sw              0 0
  # For xfs 
  /dev/sda6               /               xfs             nodiratime      0 1
  # For btrfs
  /dev/sda6		/		btrfs defaults,noatime,compress=lzo,autodefrag,subvol=root	0 0


### timezone
```sh
ln -sf /usr/share/zoneinfo/America/Caracas /etc/localtime
```

### Configure make.conf

> CFLAGS="-march=core2 -O2 -pipe"
  CXXFLAGS="-march=core2 -O2 -pipe"
  MAKEOPTS="-j3"

  # accepts
  ACCEPT_LICENSE="*"

  # emerge opts
  EMERGE_DEFAULT_OPTS="--verbose"

  # hardware opts
  VIDEO_CARDS="radeon"
  INPUT_DEVICES="mouse keyboard evdev"


### Set up hostname
```sh
sed -i '2s/localhost/funtoo.normandy.lab/g' /etc/conf.d/hostname
```

### keymap
```sh
sed -i '3s/us/es/g' /etc/conf.d/keymaps
```

### clock
```sh
sed -i '5s/UTC/local/g' /etc/conf.d/hwclock
```

### set desktop flavor and X add-in
```sh
eselect profile set-flavor 8
eselect profile add 34
```

### Update currently installed packages
```sh
emerge -auDN @world
```

### Create package set for kernel
```sh
mkdir /etc/portage/sets
echo sys-kernel/gentoo-sources > /etc/portage/sets/kernel
```

### kernel install
```sh
emerge -1 @kernel
cd /usr/src/linux
make localmodconfig 
make menuconfig
make && make modules_install
cp arch/x86_64/boot/bzImage /boot/kernel-3.18.1-funtoo
cp System.map /boot/System.map-3.18.1-funtoo
```

### grub
```sh
emerge boot-update os-prober
```

### Edit /etc/boot.conf

> # with btrfs
  boot {
      generate grub
      default "Funtoo Linux" 
      timeout 3 
  }

  "Funtoo Linux" {
      kernel kernel[-v]
      params += root=/dev/sda6
      params += rootfstype=btrfs
      params += rootflags=subvol=root
  }

  "Windows 7" {
      type win7
      params root=/dev/sda1
  }

  # without btrfs
  boot {
	    generate grub
	    default "Funtoo Linux" 
	    timeout 3 
  }

  "Funtoo Linux" {
	    kernel kernel[-v]
    	params root=auto
  }

  "Windows 7" {
      type win7
      params root=/dev/sda1
  }


### generate grub2 confs
```sh
grub-install --no-floppy /dev/sda
boot-update
```

### Network configuration
```sh
emerge -av bridge-utils usermode-utilities

cd /etc/init.d/
ln -s netif.tmpl netif.eth0
ln -s netif.tmpl netif.br0
ln -s netif.tmpl netif.tap0 # only for kvm
rc-update add netif.br0 default
rc-update add netif.tap0 default

echo 'template="interface-noip"' > /etc/conf.d/netif.eth0

*only for kvm*
cat > /etc/conf.d/netif.tap0 <<_EOF
template="tap"
group="kvm"
mac_addr="10:20:30:40:50:66"
_EOF

cat > /etc/conf.d/netif.br0 <<_EOF
template="bridge"
ipaddr="172.16.1.100/16"
gateway="172.16.0.1"
nameservers="8.8.8.8 8.8.4.4"
slaves="netif.eth0 netif.tap0"
stp="on"
forwarding=1
_EOF
```

### swap optimization
```sh
echo 'vm.swappiness = 4' > /etc/sysctl.d/swappiness.conf
```

### install vim
```sh
emerge vim 
```

Set as default editor
```sh
eselect editor set 3
eselect vi set vim
```

Vim theme
```sh
cp /media/data02/Linux/skittles_dark.vim /usr/share/vim/vim74/colors/
chmod 644 /usr/share/vim/vim74/colors/skittles_dark.vim
```

### Install basic stuff
```sh
emerge tmux htop sudo eix gentoolkit portage-utils irssi glances ntfs3g
```

### User configuration
```sh
useradd -m -s /bin/bash -g users -G wheel,audio,cdrom,video,cdrw,usb netalien
passwd netalien
```

### X11 stuff
```sh
echo "x11-libs/cairo xcb" > /etc/portage/package.use
echo "app-editors/gvim gtk" >> /etc/portage/package.use
echo 'app-shells/zsh doc examples' >> /etc/portage/package.use

emerge xorg-x11 i3 i3status dmenu terminator gvim scrot feh numlockx radeon-ucode

emerge -p firefox-bin thunderbird-bin pgadmin3 google-chrome \
          media-fonts/droid google-perftools colordiff rsyslog \
          logrotate lvm2 strace 
```

### lvm
```sh
rc-update add lvm default
```

### logs
```sh
rc-update add rsyslog default
```

### fcron
```sh
emerge fcron
emerge --config sys-process/fcron
rc-update add fcron default
```

### remmina
```sh
echo 'net-misc/remmina freerdp ssh' >> /etc/portage/package.use
emerge remmina
```

### utilities
```sh
echo 'net-analyzer/nmap ncat ndiff nmap-update nping' >> /etc/portage/package.use
echo 'media-libs/gd fontconfig' >> /etc/portage/package.use
echo 'app-arch/p7zip rar' >> /etc/portage/package.use
echo 'dev-vcs/subversion -http' >> /etc/portage/package.use

emerge -puN iotop atop htop dstat glances mtr nethogs iptraf-ng nmap \
	arpwatch iputils swatch sysstat iftop latencytop nload procinfo-ng \
	acct tcpdump bind-tools p7zip fio nmon lsof
```

### openpvn
```sh
echo 'net-misc/openvpn down-root iproute2 pkcs11' >> /etc/portage/package.use

emerge openvpn

mkdir /etc/openvpn/keys
cp /media/data02/Linux/OpenVPN/{one,two}.conf /etc/openvpn/
chmod 644 /etc/openvpn/{one,two}.conf
cp /media/data02/Linux/OpenVPN/{one,two}/* /etc/openvpn/keys 
chmod 600 /etc/openvpn/keys/*

cd /etc/init.d
ln -s openvpn openvpn.one
ln -s openvpn openvpn.two
```

### qemu/kvm libvirt nested virtualization
```sh
echo 'options kvm-intel nested=1' > /etc/modprobe.d/kvm_intel_nested.conf
```

### default uri
```sh
sudo sh -c 'echo export LIBVIRT_DEFAULT_URI="qemu:///system" >> /etc/profile.d/libvirt-uri.sh'
```

### USEs configuration and install
```sh
euse -E policykit

echo 'app-emulation/libvirt-glib introspection' >> /etc/portage/package.use
echo 'sys-libs/libosinfo introspection' >> /etc/portage/package.use
echo 'app-arch/unzip natspec' >> /etc/portage/package.use
echo 'app-emulation/qemu spice systemtap tci tls usb virtfs xfs' >> /etc/portage/package.use
echo 'app-emulation/libvirt audit fuse lvm lxc pcap uml virt-network' >> /etc/portage/package.use
echo 'net-dns/dnsmasq script' >> /etc/portage/package.use
echo 'app-emulation/spice client' >> /etc/portage/package.use
echo 'app-emulation/virt-manager gtk' >> /etc/portage/package.use
echo 'x11-libs/gtk+ introspection' >> /etc/portage/package.use
echo 'x11-libs/gdk-pixbuf introspection' >> /etc/portage/package.use
echo 'x11-libs/vte introspection'  >> /etc/portage/package.use
echo 'dev-libs/atk introspection' >> /etc/portage/package.use
echo 'net-libs/gtk-vnc introspection' >> /etc/portage/package.use
echo 'net-misc/spice-gtk introspection usbredir gtk3' >> /etc/portage/package.use
echo 'x11-libs/pango introspection' >> /etc/portage/package.use
echo 'sys-auth/polkit introspection' >> /etc/portage/package.use 
echo 'app-emulation/libguestfs ocaml' >> /etc/portage/package.use

emerge -ND libvirt virt-manager libguestfs

rc-update add consolekit default

cp /media/data02/Linux/ConfigFiles/polkit/20-libvirt.rules /etc/polkit-1/rules.d
chmod 644 /etc/polkit-1/rules.d/20-libvirt.rules
```

### moc
```sh
echo 'media-sound/moc musepack' >> /etc/portage/package.use
emerge moc
```

### vlc
```sh
echo 'media-video/vlc musepack vaapi' >> /etc/portage/package.use
emerge vlc
```

### fontconfig
```sh
euse -E infinality

emerge -uND @world

for i in {1..45}; do eselect fontconfig disable $i; done
for f in 10-autohint 10-sub-pixel-rgb 11-lcdfilter-default 70-no-bitmaps; do eselect fontconfig enable $f.conf; done
for i in {29..31}; do eselect fontconfig enable $i; done #google-droid

eselect fontconfig enable 52-infinality.conf
eselect infinality set infinality
eselect lcdfilter set infinality
```

### Themes for gtk

lxappearance is used to set the themes
```sh
emerge gtk-engines-murrine lxappearance
tar xf MediterraneanNight-*.tar.gz 
mkdir ~netalien/.themes
```

### For wallpaper change

This is set in a cron every one hour
```sh
feh --bg-scale -z /media/data01/Images
```

### File manager
```sh
echo 'gnome-base/gvfs fuse udisks' >> /etc/portage/package.use 
echo 'xfce-base/thunar fuse udisks' >> /etc/portage/package.use
emerge thunar
```

### Extra config for i3

This is done because the default terminal app is terminator which set WM_CLASS to *x-terminal-emulator* no matter the console app running within it
```sh
sudo sh -c "cat > /usr/bin/tmocp <<_EOF
terminator -c mocp -e mocp
_EOF
"
sudo chmod +x /usr/bin/tmocp

sudo sh -c "cat > /usr/bin/tirssi <<_EOF
terminator -c irssi -e irssi
_EOF
"
sudo chmod +x /usr/bin/tirssi

sudo sh -c "cat > /usr/bin/tglances <<_EOF
terminator -c glances -e glances
_EOF
"
sudo chmod +x /usr/bin/tglances

sudo sh -c "cat > /usr/bin/thtop <<_EOF
terminator -c htop -e htop
_EOF
"
sudo chmod +x /usr/bin/thtop
```sh


### For mtp (s4)
```sh
gpasswd -a netalien plugdev

sed -i 's/^#user_allow_other/user_allow_other/g' /etc/fuse.conf

emerge simple-mtpfs
```

As regular user
```sh
mkdir ~/Android
simple-mtpfs ~/Android
```

### For kde...
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
