Debian 8.0 (testing) 
====================

### NetworkManager GTFO

Install bridge utils
```sh
apt-get install bridge-utils
```

```sh
systemctl disable NetworkManager && systemctl stop NetworkManager
```

Set resolv.conf
```sh
echo 'nameserver 8.8.8.8' >> /etc/resolv.conf
echo 'nameserver 8.8.4.4' >> /etc/resolv.conf
```

Create bridge
```sh
cat > /etc/network/interfaces.d/br0 <<_EOF
iface eth0 inet manual

iface br0 inet static
        address 172.16.1.100
        broadcast 172.16.1.255
        netmask 255.255.0.0
        gateway 172.16.0.1
	bridge_stp off       # disable Spanning Tree Protocol
        bridge_waitport 0    # no delay before a port becomes available
        bridge_fd 0          # no forwarding delay
	bridge_ports eth0    # bridge eth0 only
        #bridge_ports none    # if you do not want to bind to any ports
        #bridge_ports regex eth* # use a regular expression to define ports
_EOF
```

Restart networking
```sh
systemctl stop networking && ip addr flush eth0 && systemctl start networking
```

### Set sources.list

Add *contrib non-free* to all ports and deactivate deb cdrom repo

Update repos and system
```sh
apt-get update && apt-get upgrade
```

### Bash customization

Add at the bottom of /etc/bash.bashrc:

  # Aliases
  alias ll='ls -lh --color=auto'
  alias la='ls -lhA --color=auto'
  alias ls='ls --color=auto'
  alias grep='grep --colour=auto'
  alias su='su -'
  alias cp='cp -v'
  alias chmod='chmod -v'
  alias chown='chown -v'
  alias mv='mv -v'
  alias rm='rm -v'
  alias mkdir='mkdir -v'
  alias rmdir='rmdir -v'
  alias ln='ln -v'
  
  # Config
  shopt -s histappend
  history -a

  # Exports
  export EDITOR='/usr/bin/vim'
  export HISTSIZE=2000
  export HISTFILESIZE=$HISTSIZE
  export HISTTIMEFORMAT="%d %b | %k:%M -> "


### Important packages

```sh
apt-get install vim vim-syntax-docker vim-syntax-go vim-haproxy vim-puppet vim-gnome vim-scripts vim-addon-manager vim-syntastic vim-tlib vim-doc wajig debtags apt-move apt-file deborphan apt-show-versions debsums debconf-utils cscope exuberant-ctags htop iotop sysstat nethogs iptraf-ng wireshark nmap mtr bind9utils remmina remmina-plugin-vnc remmina-plugin-rdp terminator pgadmin3 tcpdump p7zip-full p7zip-rar fio git subversion pavucontrol sudo strace ltrace golang-go debian-keyring openvpn resolvconf icedove glances numlockx irssi firmware-linux-nonfree libcanberra-gtk-module libcanberra-gtk3-module
```

### libvirt and friends

```sh
wajig install libvirt-bin qemu-kvm virt-manager libguestfs-tools
```

Add netalien to virt groups
```sh
gpasswd -a netalien libvirt
gpasswd -a netalien kvm
```

### vlc and moc

```sh
wajig install vlc moc moc-ffmpeg-plugin
```

Copy moc config (as netalien)
```sh
mkdir .moc
cp /media/data02/Linux/ConfigFiles/Moc/config .moc/config && chmod 644 .moc/config
```

### nested kvm

```sh
echo 'options kvm-intel nested=1' > /etc/modprobe.d/99-intel-nestedvirt.conf
```

### vim modifications

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
set showmatch           " Show matching parents/brackets
set nohlsearch          " Disable search hilingthing
set undolevels=1000     " a lot -go back- possibilities
set hidden              " Hide buffers when they are abandoned
set mouse=a             " Enable mouse usage (all modes)
set showcmd             " Show (partial) command in status line.
set number              " display line numbers
" Theme settings
set t_Co=256
colorscheme skittles_dark

Copy vim theme
```sh
cp /media/data02/Linux/skittles_dark.vim /usr/share/vim/vim74/colors/
chmod 644 /usr/share/vim/vim74/colors/skittles_dark.vim
```

### i3 setup

Install i3
```sh
wajig install i3
```

Copy config files (as netalien)
```sh
mkdir .i3
cp /media/data02/Linux/ConfigFiles/i3/config .i3/config && chmod 644 .i3/config
cp /media/data02/Linux/ConfigFiles/i3/i3status.conf .i3status.conf && chmod 644 .i3status.conf
```

Fix missing cursor upon loading i3 with gnome-settings-daemon (as netalien)
```sh
gsettings set org.gnome.settings-daemon.plugins.cursor active false
```

Set nautilus to sort folders first
```sh
gsettings set org.gnome.nautilus.preferences sort-directories-first true
```

### Unneeded services

```sh
systemctl disable ModemManager exim4 pppd-dns glances speech-dispatcher avahi-daemon mdadm-raid packagekit lvm2-activation-early saned lvm2-activation lvm2-monitor
```
