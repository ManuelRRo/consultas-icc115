### Consulta configuracion netplan 
Mi máquina host(mi laptop) tiene configurado los siguientes bridges 
Utilizo nm-bridge para usar en kvm con la red WAN de opnsense
Utilizo br-lan para usar en kvm con la red LAN de opnsense
y ya instale en ambas máquinas virtuales los paqutes de vlan y bridge-utils

```
apt install vlan bridge-utils
```

```
network:
  version: 2
  ethernets:
    enxc8a362be49d8:
      dhcp4: no
    enx00e04c3601d5:
      optional: true
      dhcp4: false
      dhcp6: false
  bridges:
    nm-bridge:
      interfaces: [enxc8a362be49d8]
      dhcp4: no
      addresses: [192.168.31.15/24]
      routes:
        - to: default
          via: 192.168.31.1
      nameservers:
        addresses: [8.8.8.8,8.8.4.4]
    br-lan:
      dhcp4: false
      dhcp6: false
      interfaces: [enx00e04c3601d5]
      link-local: []
```
tengo dos máquinas virtuales(vm3,vm4) con la siguiente configuracion de vlans

VLAN20 -> 192.168.2.0/24

VLAN30 -> 192.168.3.0/24

Y en kvm estoy usando el bridge br-lan para la interfaz **enp7s0**
Al configurar de la siguiente manera las máquinas si recibo ping en las vlan
### vm3 
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
    enp7s0:
      dhcp4: no
    enp8s0:
      dhcp4: false
      addresses: [10.0.0.12/24]
  vlans:
    enp7s0-vlan20:
      id: 20
      link: enp7s0
      addresses: [192.168.2.7/24]
      routes:
        - to: default
          via: 192.168.2.1
    enp7s0-vlan30:
      id: 30
      link: enp7s0
      addresses: [192.168.3.7/24]
```
### vm4
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
    enp7s0:
      dhcp4: no
    enp8s0:
      dhcp4: false
      addresses: [10.0.0.13/24]
  vlans:
    enp7s0-vlan20:
      id: 20
      link: enp7s0
      addresses: [192.168.2.8/24]
      routes:
        - to: default
          via: 192.168.2.1
    enp7s0-vlan30:
      id: 30
      link: enp7s0
      addresses: [192.168.2.8/24]

```

Pero de la siguiente manera utilizando los bridge no logran comunicarse mediante ping las dos máquinas virtuales

### vm3
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
    enp7s0:
      dhcp4: false
    enp8s0:
      dhcp4: no
      addresses: [10.0.0.13/24]
  bridges:
    br0-vlan20:
      dhcp4: no
      interfaces: [enp7s0-vlan20]
      addresses: [192.168.2.7/24]
      routes:
        - to: default
          via: 192.168.2.1
          metric: 100
          on-link: true
      nameservers:
          addresses: [1.1.1.1,8.8.8.8]
    br0-vlan30:
      dhcp4: no
      interfaces: [enp7s0-vlan30]
      addresses: [192.168.3.8/24]
  vlans:
    enp7s0-vlan20:
      id: 20
      link: enp7s0
      accept-ra: no
    enp7s0-vlan30:
      id: 30
      link: enp7s0
      accept-ra: no
```

### vm4
```
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: no
    enp7s0:
      dhcp4: false
    enp8s0:
      dhcp4: no
      addresses: [10.0.0.13/24]
  bridges:
    br0-vlan20:
      dhcp4: no
      interfaces: [enp7s0-vlan20]
      addresses: [192.168.2.8/24]
      routes:
        - to: default
          via: 192.168.2.1
          metric: 100
          on-link: true
      nameservers:
          addresses: [1.1.1.1,8.8.8.8]
    br0-vlan30:
      dhcp4: no
      interfaces: [enp7s0-vlan30]
      addresses: [192.168.3.8/24]
  vlans:
    enp7s0-vlan20:
      id: 20
      link: enp7s0
      accept-ra: no
    enp7s0-vlan30:
      id: 30
      link: enp7s0
      accept-ra: no
```

Ya habilite todas las reglas del firewall en pass para que no haya problema con el trafico pero aun asi con la configuracion del bridge no funciona el ping entre hosts solo hace ping al gateway de cada vlan.

Qué otras configuraciones podría intentar para utilizar el bridge