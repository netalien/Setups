OpenSUSE
========

### update
```sh
zypper dup
```

### nested kvm
```sh
echo 'options kvm-intel nested=1' > /etc/modprobe.d/99-intel-nestedvirt.conf
```

### better swapiness
```sh
echo 'vm.swappiness = 20' > /etc/sysctl.d/swappiness.conf
sysctl -p /etc/sysctl.d/swappiness.conf
```

### better disk cache
```sh
echo 'vm.dirty_ratio = 10' > /etc/sysctl.d/diskcache.conf
echo 'vm.dirty_background_ratio = 3' >> /etc/sysctl.d/diskcache.conf
sysctl -p /etc/sysctl.d/diskcache.conf
```sh

### packman repo
```sh
zypper ar -f -n packman http://packman.inode.at/suse/openSUSE_Tumbleweed/ packman
```

### Important packages
```sh
zypper in htop iotop nethogs iptraf-ng wireshark vim-data vim-plugin-colorschemes \
	go-vim gvim sysstat sysstat-isag nmap nping ncat arpwatch iputils latencytop \
	tcpdump bind-utils terminator pgadmin3 remmina-plugin-vnc remmina-plugin-rdp \
	pavucontrol ntop p7zip unrar fio gfio collectl git-core MozillaThunderbird \
	okular ffmpegthumbnailer kffmpegthumbnailer gtk3-engine-oxygen gtk3-theme-oxygen
```

### vlc
```sh
zypper in vlc vlc-codecs
```

### Virtualization newest repo
```sh
zypper ar -f -n virtualization http://download.opensuse.org/repositories/Virtualization/openSUSE_Factory/ virtualization
```

### KVM

Install kvm and tools from yast, additionally install:
```sh
zypper in guestfs-tools virt-utils virt-v2v
```

### Activate libvirt
```sh
systemctl enable libvirtd ksm
```

### Polkit config
```sh
cp /media/data02/Linux/ConfigFiles/polkit/*.rules /etc/polkit-1/rules.d/
chmod 644 /etc/polkit-1/rules.d/19-udisk2-mount.rules 
chmod 644 /etc/polkit-1/rules.d/20-libvirt.rules 
```

### bash customizations
```sh
cat > /etc/bash.bashrc.local <<_EOF
### Aliases

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

### Configs
shopt -s histappend
history -a

### Exports
export EDITOR='/usr/bin/vim'
export HISTSIZE=2000
export HISTFILESIZE=$HISTSIZE
export HISTTIMEFORMAT="%d %b | %k:%M -> "
_EOF
```

### Config vim
```sh
cp /media/data02/Linux/skittles_dark.vim /usr/share/vim/current/colors/
chmod 644 /usr/share/vim/vim74/colors/skittles_dark.vim
cp /media/data02/Linux/ConfigFiles/Vim/vimrc_opensuse /etc/vimrc
chmod 644 /etc/vimrc
```
