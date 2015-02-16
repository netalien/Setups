Fedora Install
==============


# Eliminar el daemon automatico que busca los updates
```sh
killall -9 packagekitd
```

# No requerir el refresh de packagekit
```sh
sed -i '2s/1/0/g' /etc/yum/pluginconf.d/refresh-packagekit.conf
```

# Activar nested kvm virt
```sh
echo 'options kvm-intel nested=1' > /etc/modprobe.d/kvm_intel_nested.conf
```

# deshabilitar ipv6, en Network Manager, tab ipv6 seleccionar Ignore
```sh
cat > /etc/sysctl.d/ipv6.conf << _EOF
net.ipv6.conf.all.disable_ipv6=1
net.ipv6.conf.default.disable_ipv6=1
_EOF

sysctl -p /etc/sysctl.d/ipv6.conf

sed -i '2s/^/#/g' /etc/hosts
```

# Optimizar swappiness
```sh
echo 'vm.swappiness = 20' > /etc/sysctl.d/swappiness.conf
sysctl -p /etc/sysctl.d/swappiness.conf
```

# Optimizar cache de disco
```
echo 'vm.dirty_ratio = 10' > /etc/sysctl.d/diskcache.conf
echo 'vm.dirty_background_ratio = 3' >> /etc/sysctl.d/diskcache.conf
sysctl -p /etc/sysctl.d/diskcache.conf
```

# Configurar red

+ Usando NetworkManager

Crear bridge, interface y reiniciar
```sh
sed -i 's/plugins=.*/plugins=keyfile/g' /etc/NetworkManager/NetworkManager.conf

cat > /etc/NetworkManager/system-connections/br0.ini <<_EOF

[connection]
id=br0
uuid=42f72868-7e5d-4ec1-89ff-539b77f52de4
interface-name=br0
type=bridge
autoconnect=true

[bridge]
interface-name=br0

[ipv4]
method=manual
dns=192.168.1.1
addresses=192.168.1.100;24;192.168.1.1

[ipv6]
method=ignore
_EOF

cat > /etc/NetworkManager/system-connections/em1.ini <<_EOF
[connection]
id=em1
uuid=4b5a2867-1b09-47f2-8eca-04d106b4818d
type=802-3-ethernet
master=42f72868-7e5d-4ec1-89ff-539b77f52de4
slave-type=bridge

[802-3-ethernet]
mac-address=00:25:22:71:2f:07
_EOF

chmod 600 /etc/NetworkManager/system-connections/*.ini

systemctl restart NetworkManager
```

+ Usando network.service

Crear bridge, desactivar NetworkManager y activar network
```sh
cat > /etc/sysconfig/network-scripts/ifcfg-bridge0 <<_EOF
DEVICE=bridge0
TYPE=Bridge
BOOTPROTO=dhcp
NAME=bridge0
ONBOOT=yes
NM_CONTROLLED=no
_EOF

cat > /etc/sysconfig/network-scripts/ifcfg-em1 <<_EOF
IPV6INIT="no"
IPV4_FAILURE_FATAL="no"
HWADDR="00:25:22:71:2F:07"
BOOTPROTO="none"
TYPE="Ethernet"
ONBOOT="yes"
DEVICE="em1"
NAME="em1"
NM_CONTROLLED="no"
BRIDGE="bridge0"
_EOF

systemctl disable NetworkManager && chkconfig network on
systemctl stop NetworkManager && service network start
```

# Update
```sh
yum -y distro-sync
```

# Instalar rpmfusion
```sh
yum -y localinstall --nogpgcheck http://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm http://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

# instalar paquetes varios
```sh
yum -y install htop iotop nethogs iptraf-ng wireshark vim-enhanced vim-X11 dstat sysstat nmap glances sysstat netmonitor atop arpwatch fspy ibmonitor iputils swatch sysusage checkdns iftop latencytop nload procps-ng psacct tcpdump bind-utils terminator pgadmin3 remmina-plugins-vnc remmina-plugins-rdp rdiff-backup pavucontrol ntop p7zip p7zip-plugins unrar fio collectl nmon mtr ffmpegthumbs git thunderbird
```

# Instalar vlc y moc
```sh
yum -y install vlc moc
```

# livirt y demas
```sh
yum -y groupinstall @virtualization; yum -y install libguestfs-tools virt-top ksm
```

# Activar ksm
```sh
for s in ksm ksmtuned; do systemctl enable $s && systemctl start $s; done
```

# Activar libvirtd
```sh
systemctl start libvirtd
```

# Archivos de configuracion de polkit
```sh
cp /media/data02/Linux/ConfigFiles/polkit/*.rules /etc/polkit-1/rules.d/
restorecon -vv /etc/polkit-1/rules.d/*.rules
chcon -u system_u /etc/polkit-1/rules.d/19-udisk2-mount.rules
chcon -u system_u /etc/polkit-1/rules.d/20-libvirt.rules
chmod 644 /etc/polkit-1/rules.d/19-udisk2-mount.rules 
chmod 644 /etc/polkit-1/rules.d/20-libvirt.rules 
```

# Copiar custom bash y vimrc
```sh
\cp /media/data02/Linux/ConfigFiles/Bash/bashrc_fedora /etc/bashrc
restorecon -vv /etc/bashrc
\cp /media/data02/Linux/ConfigFiles/Vim/vimrc_fedora /etc/vimrc
restorecon -vv /etc/vimrc
\cp /media/data02/Linux/skittles_dark.vim /usr/share/vim/vim74/colors/
restorecon -vv /usr/share/vim/vim74/colors/skittles_dark.vim 
chmod 644 /usr/share/vim/vim74/colors/skittles_dark.vim
```

# Optimizar grub

Eliminar rhgb quiet y agregar al final de la linea del kernel en /etc/default/grub

  plymouth.enable=0 selinux=0

# Desactivar selinux en config
```sh
sed -i '7s/enforcing/disabled/g' /etc/selinux/config
```

# Desactivar servicios varios
```sh
for s in fedora-configure fedora-loadmodules fedora-readonly avahi-daemon.service avahi-daemon.socket bluetooth fprintd livesys-late livesys ModemManager nfs-lock rngd iscsid.socket iscsiuio.socket dmraid-activation iscsi mdmonitor multipathd vmtoolsd dm-event proc-fs-nfsd.mount var-lib-nfs-rpc_pipefs.mount rpcbind; do systemctl disable $s && systemctl mask $s; done
```

# Reconfigurar grub menu
```sh
grub2-mkconfig -o /boot/grub2/grub.cfg
```

# Java config

Luego de la instalacion del JDK de Oracle (rpm)
```sh
update-alternatives --install /usr/bin/java java /usr/latest/java/bin/java 1
update-alternatives --install /usr/bin/javac javac /usr/latest/java/bin/javac 1
update-alternatives --install /usr/bin/javaws javaws /usr/latest/java/bin/javaws 1
```
