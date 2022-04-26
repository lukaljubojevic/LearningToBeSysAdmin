# Managing Computer Systems
## Literature
* Essential System administration O'Reilly,2002 - Frisch, Ae.
* Unix and Linux system administration handbook - Nemeth, E. Addison 2017
* Doleželová, M., Muehlfeld, M., Svistunov, M., Wadeley, S., Čapek, T., Hradílek, J., Silas, D., Heves, J., Kovář, P., Ondrejka, P., Bokoč, P., Prpič, M., Slobodová, E., Kopalová, E., Svoboda, M., O'Brien, D., Hideo, M., Domingo, D. & Ha, J. System administrator's guide. (Red Hat, 2018)
* Aoki, O. Debian reference. (Debian, 2018).
* The FreeBSD documentation project. FreeBSD handbook. (FreeBSD, 2018.)
* How Linux Works, 3rd Edition: What Every Superuser Should Know 3rd Edition, Brian Ward
* [How to properly document software?](https://www.doxygen.nl/manual/index.htm)
* [How to properly document systems?](https://group.miletic.net/hr/nastava/materijali/upravljanje-dokumentacijom-racunalnih-sustava-i-mreza/)
    * MkDocs - Markdown or VS Code with markdownlint extension    

**.markdownlint.json example:**
```json
{
 "MD007" : { "indent": 4 },
 "MD014" : false
}
```

## MD document example:
e-Mail server is configured on IP: 10.27.45.11.
```python
print("hello")
```

URL's and lists:
+ [www.miletic.net](https://www.miletic.net/) ili <https://www.miletic.net>
+ a
+ b
    + a
    + b
        1. third
        1. one more

Code:
```shell
$ ls
```
Graph (Graphviz):
```graphviz
a -> b;
```
Quoting:
> Registering and Subscribing Your System
    Register your system:
    ~]# `subscription-manager register`
    The command will prompt you to enter your Red Hat Customer Portal user name
    and password.
    Determine the pool ID of a subscription that you require:
    ~]# `subscription-manager list --available`
    This command displays all available subscriptions for your Red Hat account.
    For every subscription, various characteristics are displayed, including the
    pool ID.
    Attach the appropriate subscription to your system by replacing pool_id with
    the pool ID determined in the previous step:
    ~]# `subscription-manager attach --pool=pool_id`
For more information on registration of your system and attachment of the Red
Hat Content Delivery Network subscriptions, see Chapter 7, Registering the
System and Managing Subscriptions.

+ [source](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-getting_started#sec-Networking-Cockpit)


Single quotes about everything that appears on the screen!
MD can be versioned on GitHub!

Good practice for System Administrators is to use Password Managers e.g. KeepassXC while ensuring that the password length is 12 or more different characters (or let the Password Manager auto generate the password).

## Graph
VS Code extension: Graphviz Markdown Preview v0.0.8 or Mermaid

# Cloud-config
[ArchLinux VM Img Cloud-init ready](https://gitlab.archlinux.org/archlinux/arch-boxes/-/jobs/50048/artifacts/file/output/Arch-Linux-x86_64-cloudimg-20220310.50048.qcow2)
[SSH](https://www.ssh.com/academy/ssh/keygen)
[QEMU](https://wiki.archlinux.org/title/QEMU#Network)
[Cloud-Init](https://wiki.archlinux.org/title/Cloud-init)

## Cloud-config for a virtual machine:
## What is a virtual machine?
According to vmware documentation:
>A Virtual Machine (VM) is a compute resource that uses software instead of a physical computer to run programs and deploy apps. One or more virtual “guest” machines run on a physical “host” machine.

According to Wikipedia:
>Virtual machine (VM) is the virtualization/emulation of a computer system. Virtual machines are based on computer architectures and provide functionality of a physical computer. Their implementations may involve specialized hardware, software, or a combination. 
## How to create and setup a virtual machine?
**1. Generate SSH key:**
```shell
ssh-keygen -t rsa -b 4096 ssh-keygen -t dsa 
ssh-keygen -t ecdsa -b 521
```
**2. Create user-data and meta-data files:**

**Basic user-data with SSH key authentication:**
```shell
#cloud-config
users:
  - name: fidit
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZWfY7BinoB43gEXhlyvd97Yo9tKHpmx/+J0dkdyVSgX3fFlmMBpfulc7WsZ7tw3BOCgFPVHlX7awGylDmff3HE2+CntQuV0vwhqJ28yRu3LvxPm6Q4wbk+lNBO7qOg8wDroE0z0ikNg+DgqqJLXhPxX0R1lawZ0Vf+cL+M56roCMZl0CZy7TuZFcBCpEb/F5ipGQehVQVuixkA7G45+R5i7x8rZbi7wztS7oYzsgNKO/z66w4xVujLf2eMaIYEL2p7/CGc3U+nNULEUTwdJ6NsBmb2lv5HfRPrmmvIxJTSNMBEdVd2fhikFkfpzGyuKwy7p8iNRkQICURC+qEOEi4HYZvYtLsSeee0GQ5P0jTMp2Ya5vVhrz0fVn4byW8esOob7eiUX3E75iwtzUUmNLN8cuCZOhSTUlcqqJMYA701CJegWNDQw0f7lKvtvwFcuscK0+g80x8rXTngiLNTCXwVAFgSoRtI7RlQJcMOo9Z91P/NT3adC6I2G4Btmgoe+0= korisnik@ODJ-O365-117

```

**Basic user-data with SSH key authentication for root and specific user:**
```shell
#cloud-config
users:
  - name: myuser
    ssh_authorized_keys:
    - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDZWfY7BinoB43gEXhlyvd97Yo9tKHpmx/+J0dkdyVSgX3fFlmMBpfulc7WsZ7tw3BOCgFPVHlX7awGylDmff3HE2+CntQuV0vwhqJ28yRu3LvxPm6Q4wbk+lNBO7qOg8wDroE0z0ikNg+DgqqJLXhPxX0R1lawZ0Vf+cL+M56roCMZl0CZy7TuZFcBCpEb/F5ipGQehVQVuixkA7G45+R5i7x8rZbi7wztS7oYzsgNKO/z66w4xVujLf2eMaIYEL2p7/CGc3U+nNULEUTwdJ6NsBmb2lv5HfRPrmmvIxJTSNMBEdVd2fhikFkfpzGyuKwy7p8iNRkQICURC+qEOEi4HYZvYtLsSeee0GQ5P0jTMp2Ya5vVhrz0fVn4byW8esOob7eiUX3E75iwtzUUmNLN8cuCZOhSTUlcqqJMYA701CJegWNDQw0f7lKvtvwFcuscK0+g80x8rXTngiLNTCXwVAFgSoRtI7RlQJcMOo9Z91P/NT3adC6I2G4Btmgoe+0= korisnik@ODJ-O365-117
  - name: root
    ssh_authorized_keys:
    - ecdsa-sha2-nistp521 AAAAE2VjZHNhLXNoYTItbmlzdHA1MjEAAAAIbmlzdHA1MjEAAACFBAA5J/SRFcYnps9vCQQwnsXquEeKt9R7/7Ab6YovEoU5nE1vUSFHXoBNPhIXKOJU8G2pGi9nMFS27z9B7+oyLPTQhwFFbUOoG+/jTrc4CHy+b2rdx7x1CbR4fQSsgC/t0g+z05o9jar/t0Lea6/6hIAM2uGh+KGiKzGT/aug2VBU3SvcbQ== korisnik@ODJ-O365-117

```
[What else can you do with cloud-config?](https://cloudinit.readthedocs.io/en/latest/topics/examples.html#)
**3. Generate cloud-init.iso:**
```shell
xorriso -as genisoimage -output cloud-init.iso -volid CIDATA -joliet -rock user-data meta-data
```
**4. Create a virtual machine:**
4.1. Expand ArchLinux disk image if you need more space:
```shell
qemu-img resize aarch.qcow2 +10G
```
4.2. Create a virtual machine
```shell
qemu-system-x86_64 -m 512M aarch.qcow2 -cdrom cloud-init.iso -nic user,hostfwd=tcp::60022-:22
```
4.3. Add port for acessing the virtual machine from host PC:
```shell
ssh-keygen -f "/home/korisnik/.ssh/known_hosts" -R "[localhost]:60022"
```
**5. Connect to the virtual machine:**
```shell
ssh -v -p 60022 fidit@localhost
ssh -v -p 60022 root@localhost
```
## Some basic networking tasks after setting up a virtual machine or server enviroment:
**Check network connection**
1. Find your network interface name
```shell
[test@test-PC ~]$ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp7s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether e0:d5:5e:44:69:77 brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.17/24 brd 192.168.0.255 scope global dynamic enp7s0
       valid_lft 2613sec preferred_lft 2613sec
    inet6 2a05:4f44:111c:8900::18/128 scope global dynamic noprefixroute 
       valid_lft 410132sec preferred_lft 341012sec
    inet6 2a05:4f44:111c:8900:50bd:bfcb:7176:40fb/64 scope global dynamic noprefixroute 
       valid_lft 414713sec preferred_lft 345593sec
    inet6 2a05:4f44:111c:8900:e2d5:5eff:fe44:6977/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 414711sec preferred_lft 345591sec
    inet6 fe80::645f:28c4:cfb8:75e3/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
Network interfaces are listed by numbers 1:, 2:, 3:, etc. 
* In a virtual machine it's named mostly eth0
* In a PC it's named mostly enpXsY
2. View network interface status and data about the network connection
```shell
[lljubojevic@Ljubojevic-PC ~]$ networkctl status enp7s0
● 2: enp7s0                                                                                                 
                     Link File: /usr/lib/systemd/network/99-default.link
                  Network File: /etc/systemd/network/20-ethernet.network
                          Type: ether
                         State: routable (configured)
                  Online state: online                                                                      
                          Path: pci-0000:07:00.0
                        Driver: r8169
                        Vendor: Realtek Semiconductor Co., Ltd.
                         Model: RTL8111/8168/8411 PCI Express Gigabit Ethernet Controller (Onboard Ethernet)
              Hardware Address: XYZ
                           MTU: 1500 (min: 68, max: 9194)
                         QDisc: fq_codel
  IPv6 Address Generation Mode: none
          Queue Length (Tx/Rx): 1/1
              Auto negotiation: yes
                         Speed: 1Gbps
                        Duplex: full
                          Port: tp
                       Address: XYZ
                       Gateway: XYZ
                           DNS: XYZ
...

**Hostnamectl**
According to [Linuxhint](https://linuxhint.com/set-hostname-using-hostnamectl-command/):
>The hostname is an identity of the system and is used by the networks to search the system.
>The “hostnamectl” is a Linux command that is used to set the hostname in the terminal without even opening and editing in the etc/hostname file of a system.
>Using the “hostnamectl” command, the user can edit the static, pretty, and transient hostname as well.

Output of hostnamectl:
```shell
[test@test-PC ~]$ hostnamectl
 Static hostname: test-PC
       Icon name: computer-desktop
         Chassis: desktop 🖥
      Machine ID: 801cb2df0c5647d1835be225e0073015
         Boot ID: 4fe8134f95d041af8dc53ab247179c28
Operating System: Arch Linux                      
          Kernel: Linux 5.17.4-arch1-1
    Architecture: x86-64
 Hardware Vendor: Gigabyte Technology Co., Ltd.
  Hardware Model: Z370P D3
```

Hostnamectl displays:
* Computer name (hostname)
* Icon name is the machine identifying name according to XDG Icon Naming Specification.
* What type of system it is? (Desktop, laptop, workstation etc.)
* Machine and Boot ID, OS and Kernel version
* CPU Architecture 
* Motherboard vendor and model
* Deployment is the enviroment in which is the system used e.g.  "development", "integration", "staging", "production"...
* Location - useful when managing multiple servers/VM's

It can be used for setting up:

* Hostname:
```shell
[root@archlinux ~]# hostnamectl set-hostname myServer1
[root@archlinux ~]# hostnamectl hostname 
myServer1
```
* Icon-name
```shell
[root@archlinux ~]# hostnamectl set-icon-name computer
[root@archlinux ~]# hostnamectl icon-name
computer
[root@archlinux ~]# hostnamectl set-icon-name workstation
[root@archlinux ~]# hostnamectl icon-name
workstation
[root@archlinux ~]# hostnamectl set-icon-name chassis-laptop
[root@archlinux ~]# hostnamectl icon-name
chassis-laptop
[root@archlinux ~]# hostnamectl set-icon-name chassis-watch
[root@archlinux ~]# hostnamectl icon-name
chassis-watch
[root@archlinux ~]# hostnamectl set-icon-name embedded
[root@archlinux ~]# hostnamectl icon-name
embedded
```
* Deployment
```shell
[root@archlinux ~]# hostnamectl deployment staging
[root@archlinux ~]# hostnamectl deployment
staging
```
* Location
```shell
[root@archlinux ~]# hostnamectl location "Berlin, room 7, rack 6"
```
End result?
```shell
[root@archlinux ~]# hostnamectl
 Static hostname: myServer1
       Icon name: embedded
         Chassis: vm 🖴
      Deployment: staging
        Location: Berlin, room 7, rack 6
      Machine ID: 168fc71dc2b143fab51a45ad7b066d1b
         Boot ID: 5dd62398ce6a4a859179e8f2b25911bf
  Virtualization: qemu
Operating System: Arch Linux                       
          Kernel: Linux 5.16.13-arch1-1
    Architecture: x86-64
 Hardware Vendor: QEMU
  Hardware Model: Standard PC _i440FX + PIIX, 1996_
```
[more at](https://www.freedesktop.org/software/systemd/man/hostnamectl.html)

## Regional settings
Some services that run on servers depend on time zone settings to work correctly. 
It's important, when planing the virtual or server environment to set it up properly.

**localectl**
>localectl - Control the system locale and keyboard layout settings

[Full list of commands](https://man.archlinux.org/man/core/systemd/localectl.1.en#COMMANDS)

**How to set it up?**
1. Check current regional settings
```shell
[test@test-PC ~]$ localectl status
   System Locale: LANG=hr_HR.UTF-8
       VC Keymap: croat
      X11 Layout: hr
```
2. List avaliable locales and keymaps - keyboard layouts      
```shell
[test@test-PC ~]$ localectl list-locales
en_US.UTF-8
hr_HR.UTF-8

[test@test-PC ~]$ localectl list-keymaps
ANSI-dvorak
adnw
amiga-de
amiga-us
apple-a1048-sv
apple-a1243-sv
apple-a1243-sv-fn-reverse
apple-internal-0x0253-sv
apple-internal-0x0253-sv-fn-reverse
applkey
atari-de
atari-se
...
```
3. Set the needed locale configuration
```shell
[test@test-PC ~]$ localectl set-locale hr_HR.UTF-8
[test@test-PC ~]$ localectl set-x11-keymap hr
[test@test-PC ~]$ setxkbmap hr
```
4. Generate the locale config file
```shell
[test@test-PC ~]$ locale-gen
```
Also note, Regional setting can be defined in cloud-init configuration files.

## Managing users and permissions
Let's try managing users in a VM created using GUI tool -> virt-manager.

Requirements:
* Virt-manager and ArchLinux VM
* Cloud-init:

**user data:**
```shell
#cloud-config
users:
  - default

system_info:
   default_user:
     name: myuser
     plain_text_passwd: '1234'
     gecos: arch Cloud User
     groups: [wheel, adm]
     sudo: ["ALL=(ALL) NOPASSWD:ALL"]
     shell: /bin/bash
     lock_passwd: False
     
```
**Note: Cloud-init is sensitive with tab spaces and requires a empty line at the end of file**

Make cloud-init.iso file:
```shell
xorriso -as genisoimage -output cloud-init.iso -volid CIDATA -joliet -rock user-data meta-data
```
Copy downloaded Arch system image (qcow2) in home directory and rename it to aarch.qcow2

Add more space to image:
```shell
qemu-img resize aarch.qcow2 +10G
```
Then in virt-manager:

* Use existing image -> aarch.qcow2 -> archlinux -> 1GB/2 cores -> Customize configuration before install -> Add cloud-init.iso and make it a CDROM type

Then start ArchLinx VM and login with SSH from terminal on host to it:
```
ssh myuser@IPadressOfVM
```
Be careful when configuring users -> detect which user has permissions and what permissions does he have.
* ! in /etc/shadow file mean that the user is forbiden to login into the system
* User has permission to access ports from 1024, system uses ports to 1024.
* kvm group in etc/groups exists for using nested virtualization
* /dev/vd* - Linux way of mapping path to hard drives -> user dosen't have acess to disk, but has to filesystem!

Working with users:
>In Unix-like operating systems, the chmod command sets the permissions of files or directories.

Syntax:
```shell
chmod ugo -rwx
```
Where:
* ugo specifies a user, group of users or others
* -rwx specifies allowed operations: read, write, execute
>On Unix-like operating systems, the chown command changes ownership of files and directories in a filesystem.

Syntax:
```shell
chown parameters user
```
Removing ownership:
```shell
[myuser@archlinux ~]$ chmod -R go-r ftp ftp/ 
[myuser@archlinux ~]$ chmod -R go-- ftp ftp/ 
```
Where:
* go-r removes reading permissions
* go-- removes all permisions

>On Unix-like operating systems, the usermod command modifies a user account.

Syntax:
```shell
[myuser@archlinux ~]$ sudo usermod -aG video myuser
```
* Adds user "myuser" to group "video"
```shell
[myuser@archlinux ~]$ grep video /etc/group
video:x:985:myuser
```
>On Unix-like operating systems, the useradd command creates a new user or sets the default information for new users.

Syntax:
```shell
useradd user -c "Basic User" -s /bin/bash -m
```
* Where "Basic User" is description of the user, -s which shell will he use, -m creates a home directory for him.
To add password for user "user" use:
```shell
passwd user
```
To block user loging into the system use:
```shell
usermod -s /bin/false user
```
>/bin/false does nothing and it just exits with a status code indicating failure when a user attempts to login to the machine.

To view all devices attached to system use:
```shell
ls -la /dev
```
To view all users in the system use:
```shell
[test@test-PC ~]$ userdbctl
NAME                   DISPOSITION   UID   GID REALNAME                       HOME              SHELL             
root                   intrinsic       0     0 -                              /root             /usr/bin/zsh
nobody                 intrinsic   65534 65534 Nobody                         /                 /usr/bin/nologin
bin                    system          1     1 -                              /                 /usr/bin/nologin
daemon                 system          2     2 -                              /                 /usr/bin/nologin
mail                   system          8    12 -                              /var/spool/mail   /usr/bin/nologin
ftp                    system         14    11 -                              /srv/ftp          /usr/bin/nologin
rpc                    system         32    32 Rpcbind Daemon                 /var/lib/rpcbind  /usr/bin/nologin
http                   system         33    33 -                              /srv/http         /usr/bin/nologin
named                  system         40    40 BIND DNS Server                /                 /usr/bin/nologin
uuidd                  system         68    68 -                              /                 /usr/bin/nologin
dbus                   system         81    81 System Message Bus             /                 /usr/bin/nologin
polkitd                system        102   102 PolicyKit daemon               /                 /usr/bin/nologin
partimag               system        110   110 Partimage user                 /                 /usr/bin/nologin
rtkit                  system        133   133 RealtimeKit                    /proc             /usr/bin/nologin
usbmux                 system        140   140 usbmux user                    /                 /usr/bin/nologin
nvidia-persistenced    system        143   143 NVIDIA Persistence Daemon      /                 /usr/bin/nologin
cups                   system        209   209 cups helper user               /                 /usr/bin/nologin
saned                  system        962   962 SANE daemon user               /                 /usr/bin/nologin
libvirt-qemu           system        964   964 Libvirt QEMU user              /                 /usr/bin/nologin
sddm                   system        968   968 Simple Desktop Display Manager /var/lib/sddm     /usr/bin/nologin
openvpn                system        969   969 OpenVPN                        /                 /usr/bin/nologin
nbd                    system        970   970 Network Block Device           /var/empty        /usr/bin/nologin
git                    system        971   971 git daemon user                /                 /usr/bin/git-shell
dnsmasq                system        972   972 dnsmasq daemon                 /                 /usr/bin/nologin
dhcpcd                 system        973   973 dhcpcd privilege separation    /                 /usr/bin/nologin
avahi                  system        974   974 Avahi mDNS/DNS-SD daemon       /                 /usr/bin/nologin
tss                    system        975   975 tss user for tpm2              /                 /usr/bin/nologin
systemd-timesync       system        976   976 systemd Time Synchronization   /                 /usr/bin/nologin
systemd-resolve        system        977   977 systemd Resolver               /                 /usr/bin/nologin
systemd-journal-remote system        978   978 systemd Journal Remote         /                 /usr/bin/nologin
systemd-oom            system        979   979 systemd Userspace OOM Killer   /                 /usr/bin/nologin
systemd-network        system        980   980 systemd Network Management     /                 /usr/bin/nologin
systemd-coredump       system        981   981 systemd Core Dumper            /                 /usr/bin/nologin
test                   regular      1000  1000 Test                           /home/test        /bin/bash

```
To kill all processes for a user session and deallocate the resources use:
```shell
loginctl kill-user user
```
* For those who want to know more:
Users interact with the system using GUI or by terminal interface.
[Here](https://www.golinuxcloud.com/difference-between-pty-vs-tty-vs-pts-linux/) you can find out more about terminal devices avaliable to users.
More simultaneous users with keyboards and screens on one PC? No problem! [Multi-seat](https://www.freedesktop.org/wiki/Software/systemd/multiseat/).
