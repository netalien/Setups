Ubuntu
======

# Give root a password
```sh
sudo passwd root
```

# Update repos url
```sh
sed -i 's/ve.archive.ubuntu.com/us.archive.ubuntu.com/g' /etc/apt/sources.list
```

# Create folders for custom partitions
```sh
mkdir -pv /media/data0{0,1,2}
```

# Add to fstab

  /dev/cinder-volumes/storage     /media/data00   xfs     defaults,noatime,nodiratime,norelatime  0       0
  /dev/sdb2                       /media/data01   ntfs-3g defaults        0       0
  /dev/sdb3                       /media/data02   ntfs-3g defaults        0       0

# mount custom partitions
```sh
mount -a
``` 

# Update repos and system
```sh
sudo su
apt-get update && apt-get dist-upgrade
```

# Personalize bash (add at bottom)

vi /etc/bash.bashrc
```sh
# Aliases

alias ll='ls -lh --color=auto'
alias la='ls -lhA --color=auto'
alias ls='ls --color=auto'
alias grep='grep -n --colour=auto'
alias su='su -'
alias cp='cp -v'
alias chmod='chmod -v'
alias chown='chown -v'
alias mv='mv -v'
alias rm='rm -v'
alias mkdir='mkdir -v'
alias rmdir='rmdir -v'
alias ln='ln -v'
alias startdayco='/etc/init.d/openvpn start dayco'
alias stopdayco='/etc/init.d/openvpn stop dayco'
alias startsigis='/etc/init.d/openvpn start sigis'
alias stopsigis='/etc/init.d/openvpn stop sigis'

# Configs
shopt -s histappend
history -a

# Exports
export EDITOR='/usr/bin/vim'
export HISTSIZE=2000
export HISTFILESIZE=$HISTSIZE
export HISTTIMEFORMAT="%d %b | %k:%M -> "
```

# Config interfaces
```sh
cat > /etc/network/interfaces <<_EOF
# interfaces(5) file used by ifup(8) and ifdown(8)
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet manual

auto br0
iface br0 inet static
        address 172.16.1.100
        netmask 255.255.0.0
        gateway 172.16.0.1
        dns-nameservers 172.16.0.1
        bridge_ports eth0
        bridge_stp off
        bridge_fd 0
        bridge_maxwait 0
_EOF
``` 

# Install important packages
```sh
apt-get install vim vim-addon-manager vim-syntastic vim-gnome vim-scripts wajig debtags ctags vim-doc cscope exuberant-ctags reportbug apt-move apt-file deborphan apt-show-versions debsums debconf-utils resolvconf
```

# Install extra packages
```sh
wajig install vim-syntax-docker vim-syntax-go htop iotop sysstat nethogs iptraf-ng wireshark nmap mtr bind9utils remmina remmina-plugin-vnc remmina-plugin-rdp terminator pgadmin3 tcpdump p7zip-full p7zip-rar fio git subversion ffmpegthumbnailer kffmpegthumbnailer thunderbird pavucontrol
```

# libvirt and friends
```sh
wajig install libvirt-bin qemu-kvm virt-manager libguestfs-tools
```

# vlc and moc
```sh
wajig install vlc moc moc-ffmpeg-plugin
```

# nested kvm
```sh
echo 'options kvm-intel nested=1' > /etc/modprobe.d/99-intel-nestedvirt.conf
```

# vim modifications

  set nocompatible        " Use Vim defaults (much better!)
  set bs=2                " Allow backspacing over everything in insert mode
  set ai                  " Always set auto-indenting on
  set history=50          " keep 50 lines of command history
  set ruler               " Show the cursor position all the time
  filetype plugin indent on               " Automatic file type detection
  set autoindent smartindent      " turn on auto/smart indenting
  set expandtab           " use spaces, no tabs
  set smarttab            " make <space> and <tab> smarter
  set tabstop=2           " tabstop
  set shiftwidth=2        " indents 4
  set showmatch       " Show matching parents/brackets
  set nohlsearch      " Disable search hilingthing
  set undolevels=1000     " a lot -go back- possibilities
  set hidden             " Hide buffers when they are abandoned
  set mouse=a             " Enable mouse usage (all modes)
  set showcmd             " Show (partial) command in status line.
  set number              " display line numbers
  " Theme settings
  set t_Co=256
  colorscheme skittles_dark

```sh
cp /media/data02/Linux/skittles_dark.vim /usr/share/vim/vim74/colors/
chmod 644 /usr/share/vim/vim74/colors/skittles_dark.vim
```
