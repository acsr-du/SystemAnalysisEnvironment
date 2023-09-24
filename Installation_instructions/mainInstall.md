# About

This repository contains the installation procedure of the DRAKVUF and an automated installation debian package. DRAKVUF is a VMI based black-box malware analysis tool that allows the execution of the malware binaries without using any third party tools. It uses the XEN Hypervisor which is installed in the DOM0 environment and Analysis part of the malware is done in the DOM1, DOM2 and so on.

# DRAKVUF Installation

DRAKVUF is a black box binary dynamic malware analysis tool. It works on the principle of the VMI (Virtual Machine Introspection).

1. Make sure to disable the "Secure Boot" from the BIOS.
2. While Installating the ubuntu make at least 100 GB space free for LVM group.

## Installation
⚠ Disclaimer: This installation procedure has been tested on our PCs and works perfectly fine acording to our requirements. Changes can be made to setup and build according to your preferences.

### Our system configuration
- CPU: Intel(R) Core(TM) i7-10700 @ 2.90GHz
- Memory: 16GiB DDR4 3200 MHz
- Storage: HP SSD S700 500G
- OS: Ubuntu 20.04.6 LTS Focal Fossa

## Operating System Configuration

- Before installing the DRAKVUF you have to make sure, that partition in the system for the LVM and system space.

- In Installation type, select the "Something else" option.

<img title="Installation Type" alt="Select 'something else' option." src="/Installation_instructions/images/3.png" width="650" height="500">

- Now create the EFI system partition, swap memory partition and the main system space for DOM0 XEN installation.
  Note: While allocating the space for the VM or LVM make sure you will not mount that partition.

<img title="Image 1" alt="Size partion of disk" src="/Installation_instructions/images/size.png" width="650" height="500">

- Click on the 'Install Now' button.


## Dependencies and Packages Installation

These commands works fine with Debian based linux distro. First install the required dependencies.

First Update your linux system:

```bash
sudo apt update
sudo apt upgrade -y
```

Now install the required Dependencies.

```bash
sudo apt-get install wget git bcc bin86 gawk bridge-utils iproute2 libcurl4-openssl-dev bzip2 libpci-dev build-essential make gcc clang libc6-dev linux-libc-dev zlib1g-dev libncurses5-dev patch libvncserver-dev libssl-dev libsdl-dev iasl libbz2-dev e2fslibs-dev git-core uuid-dev ocaml libx11-dev bison flex ocaml-findlib xz-utils gettext libyajl-dev libpixman-1-dev libaio-dev libfdt-dev cabextract libglib2.0-dev autoconf automake libtool libjson-c-dev libfuse-dev liblzma-dev autoconf-archive kpartx python3-dev python3-pip golang python-dev libsystemd-dev nasm ninja-build lvm2 bridge-utils net-tools virt-manager gvncviewer -y
```

pip3 command is used to install those dependency packages which are old and cannot be installed from apt command.

```bash
sudo pip3 install pefile construct
```

## Cloning of DRAKVUF from Official repository

Cloning the DRAKVUF directory from the official Github Repository.

```bash
cd ~
git clone https://github.com/tklengyel/drakvuf
cd drakvuf
git submodule update --init
cd xen
./configure --enable-githttp --enable-systemd --enable-ovmf --disable-pvshim
make -j4 dist-xen
make -j4 dist-tools
make -j4 debball
```

## XEN Installation

Now we have to install "Xen" with dom0 getting sufficient RAM and CPU cores. You can modify these configuration according to your need. At last update the Grub and reboot the system.

```bash
sudo su
apt-get remove xen* libxen*
dpkg -i dist/xen*.deb
echo "GRUB_CMDLINE_XEN_DEFAULT=\"dom0_mem=4096M,max:4096M dom0_max_vcpus=8 dom0_vcpus_pin=1 force-ept=1 ept=ad=0 hap_1gb=0 hap_2mb=0 altp2m=1 hpet=legacy-replacement smt=0\"" >> /etc/default/grub
echo "/usr/local/lib" > /etc/ld.so.conf.d/xen.conf
ldconfig
echo "none /proc/xen xenfs defaults,nofail 0 0" >> /etc/fstab
echo "xen-evtchn" >> /etc/modules
echo "xen-privcmd" >> /etc/modules
systemctl enable xen-qemu-dom0-disk-backend.service
systemctl enable xen-init-dom0.service
systemctl enable xenconsoled.service
update-grub
```

Note: Make sure that you are running a relatively recent kernel. In Ubuntu 20.04 the 5.10.0-1019-oem kernel has been verified to work, anything newer would also work. Older kernels in your dom0 will not work properly.

```bash
uname -r
```

After above step make the XEN to boot before the linux kernal.

```bash
cd /etc/grub.d/;
mv 20_linux_xen 09_linux_xen
```

Update the changes in the Grub and then reboot the system.

```bash
update-grub
reboot
```

Verify the XEN installation. The output will show the "Running in PV context on Xen v4.7" message on the screen

```bash
sudo xen-detect
```

This command will list the running VM.

```bash
sudo xl list
```

The output should have to be similar to this.

```bash
Name                                        ID   Mem    VCPUs 	 State	 Time(s)
Domain-0                                       0  4096     2       r-----    614.0
```

## Logical Volume Manager(LVM Setup)


Note: Before creating physical volume (PV), Go inside Disk partition and create a volume. Never give the whole path of disk like /dev/sda otherwise your os will be crashed down. So when you will create a volume then it has named like /dev/sd2 or /dev/sd3 and so on. So pick only a free volume that you have made for windows then move ahead.

1. Create partition of the free space that you left for windows installation.
2. Note down the partition ID (In this case: sda4).

<img title="Installation Type" alt="Select 'something else' option." src="/Installation_instructions/images/disk.png" width="500" height="350"> <img title="Installation Type" alt="Select 'something else' option." src="/Installation_instructions/images/ds.png" width="500" height="350">

Create physical volume. Here "sda" is disk volume, it can vary accordingly.

```bash
sudo pvcreate /dev/sda4
```

Create one volume Group.

```bash
sudo vgcreate vg /dev/sda4
```

Create logical volume group. You can specify the size, here we allocating 100 GB space, you can change it by altering the the size of '100'.

```bash
sudo lvcreate -L100G -n windows7-sp1 vg
```

#### Note: Sometimes VM may get corrupted due to some reason, in this case we just need to remove that logical volume and create it again. [Guide to re-create VM](/Installation_instructions/vmCreation.md)

<br>

## Check the VMM Utility tool And Networking tool.

#### ***Double check in Ubuntu store if Virtual Machine Manager is Installed***

<img title="Installation Type" alt="Select 'something else' option." src="/Installation_instructions/images/VMM.png" width="650" height="500">



## Networking Configuration

Next we need to set up our system so that we can attach virtual machines to the external network. This is done by creating a virtual switch within dom0. The switch will take packets from the virtual machines and forward them on to the physical network so they can see the internet and other machines on your network.

The piece of software we use to do this is called the Linux bridge and its core components reside inside the Linux kernel. In this case, the bridge acts as our virtual switch. The Debian kernel is compiled with the Linux bridging module.

Management of the bridge is usually done using the brctl command. The initial setup for our Xen bridge, though, is a "set it once and forget it" kind of thing, so we are instead going to configure our bridge through Debian’s networking infrastructure. It can be configured via /etc/network/interfaces.

Open this file with the editor of your choice. If you selected a minimal installation, the nano text editor should already be installed. Open the file:

Note: nano is a text file editor for linux. You can also use the vi, vim editor also. If command show error then run "sudo apt install nano -y"

```bash
sudo nano /etc/network/interfaces
```

This file is very simple. Each stanza represents a single interface.

Breaking it down,

1. “auto eth0” means that eth0 will be configured when ifup -a is run (which happens at boot time). This means that the interface will automatically be started/stopped for you. ("eth0 is its traditional name - you'll probably see something more current like "ens1", "en0sp2" or even "enx78e7d1ea46da")

2. “iface eth0” then describes the interface itself. In this case, it specifies that it should be configured by DHCP - we are going to assume that you have DHCP running on your network for this guide. If you are using static addressing you probably know how to set that up.

We are going to edit this file so it resembles such:

Note: Change according to your network interface <br> ***Run “ifconfig” in a new terminal tab***.

```bash
ifconfig
```

Copy the following text and paste it in the interfaces files.

```bash
auto lo
iface lo inet loopback

auto enp1s0
iface enp1s0 inet manual

auto virbr0
iface virbr0 inet dhcp
     bridge_ports enp1s0
```

Make sure to add the bridge stanza, be sure to change dhcp to manual in the iface eth0 inet manual line, so that IP (Layer 3) is assigned to the bridge, not the interface. The interface will provide the physical and data-link layers (Layers 1 & 2) only.

Restart the Networking service.

```bash
sudo service network-manager restart
```

Now turn on the network bridge service.

```bash
sudo gedit /etc/NetworkManager/NetworkManager.conf
```
The file would take a while to open. In the config file change the following line : \
manages = false ---> manages = true

```bash
sudo service network-manager restart
```

To show network Vm interfaces.

```bash
brctl show
```

The output will show something like this.

```bash
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.4ccc6ad1847d       yes              virbr0-nic
```

The networking service can now be set to start automatically whenever the system is rebooted. Please review the installation instructions once again if you are having trouble getting this type of output.

## Download link for Windows iso:

Before proceeding furthur we need 64-bit windows 7 iso image. You can download from anywhere or can find the already tested image from <a href="https://drive.google.com/drive/folders/1DPBfN2B1LzVWRNDHC8sLFfAl7WrJlGP6?usp=sharing" target="_blank">HERE</a>

## VM Creation and Configuration

Next step is to edit the xen VM's configuration file.

```bash
sudo gedit /etc/xen/win7.cfg
```

This template is used for creating the Configuration for Windows 7 VM from the download ISO file. You can modify the number of cpu, max memory, VM behaviour and other system tuning.

```bash
arch = 'x86_64'
name = "windows7-sp1"
maxmem = 2048
memory = 2048
vcpus = 4
maxvcpus = 4
builder = "hvm"
boot = "cd"
hap = 1
on_poweroff = "destroy"
on_reboot = "destroy"
on_crash = "destroy"
vnc = 1
vnclisten = "0.0.0.0"
vga = "stdvga"
usb = 1
usbdevice = "tablet"
audio = 1
soundhw = "hda"
viridian = 1
altp2m = 2
shadow_memory = 32
vif = [ 'type=ioemu,model=e1000,bridge=virbr0,mac=48:9e:bd:9e:2b:0d']
disk = [ 'phy:/dev/vg/windows7-sp1,hda,w', 'file:/home/pc-01/Downloads/windows7.iso,hdc:cdrom,r' ]
```

***Note: Make changes according to your file path of windows iso image and mac address.<br>Your mac address value is the ether value of your ethernet interface. Run: ifconfig***

## Clone LibVMI and Installation

Now, Enter into the LibVMI folder and build it.

```bash
cd ~/drakvuf/libvmi
autoreconf -vif
./configure --disable-kvm --disable-bareflank
```

Output of the above command should look something this:

```bash
Feature         | Option
----------------|---------------------------
Xen Support     | --enable-xen=yes
KVM Support     | --enable-kvm=no
File Support    | --enable-file=yes
Shm-snapshot    | --enable-shm-snapshot=no
Rekall profiles | --enable-rekall-profiles=yes
----------------|---------------------------

OS              | Option
----------------|---------------------------
Windows         | --enable-windows=yes
Linux           | --enable-linux=yes


Tools           | Option                    | Reason
----------------|---------------------------|----------------------------
Examples        | --enable-examples=yes
VMIFS           | --enable-vmifs=yes        | yes
```

Build and install LibVMI

```bash
make
sudo make install
sudo echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/lib" >> ~/.bashrc
```

## Clone Volatility and Installation

```bash
cd ~/drakvuf/volatility3
python3 ./setup.py build
sudo python3 ./setup.py install
```

## Create VM and Configure VM from DOM0 for Windows installation

Last step of this configuration is to create Windows 7 VM using the following command.

```bash
sudo xl create /etc/xen/win7.cfg
```

We have noticed that sometimes the PDB GUID are different before installation and after installation of Windows. So we prefer Installing it first and then creatng  JSON file of kernel.

Now login to Virtual Machine and install the windows.
<br>
Do not set a password to your VM as it will create problems during automation.

```bash
gvncviewer localhost
```

**Perform Windows installation until you are booted to the desktop. You will need to run xl create and gvncviewer command 2-3 times.**

When the Windows Installation is finished ***Create a partition of 10G. (A seperate Disk drive)***



## JSON File Creation using LibVMI vmi-win-guid tool and Volatility Framework

Now we will create the JSON configuration file for the Windows domain. First, we need to get the debug information for the Windows kernel via the LibVMI vmi-win-guid tool. For example, in the following my domain is named windows7-sp1.

```bash
sudo xl list
```
You will get an output like this -
```bash
Name                                        ID   Mem VCPUs	State	Time(s)
Domain-0                                     0  4024     4     r-----     848.8
windows7-sp1                                 7  3000     1     -b----      94.7
```

Get the debug information.

```bash
sudo vmi-win-guid name windows7-sp1
```

The ouput should look like this : 
```bash
Windows Kernel found @ 0x2604000
  Version: 32-bit Windows 7
  PE GUID: 4ce78a09412000
  PDB GUID: f794d83b0f3c4b7980797437dc4be9e71       <---------
  Kernel filename: ntkrpamp.pdb                     <---------
  Multi-processor with PAE (version 5.0 and higher)
  Signature: 17744.
  Machine: 332.
  # of sections: 22.
    # of symbols: 0.
  Timestamp: 1290242569.
  Characteristics: 290.
  Optional header size: 224.
  Optional header type: 0x10b
  Section 1: .text
  Section 2: _PAGELK
  Section 3: POOLMI
  Section 4: POOLCODE
  Section 5: .data
  Section 6: ALMOSTRO
  Section 7: SPINLOCK
  Section 8: PAGE
  Section 9: PAGELK
  Section 10: PAGEKD
  Section 11: PAGEVRFY
  Section 12: PAGEHDLS
  Section 13: PAGEBGFX
  Section 14: PAGEVRFB
  Section 15: .edata
  Section 16: PAGEDATA
  Section 17: PAGEKDD
  Section 18: PAGEVRFC
  Section 19: PAGEVRFD
  Section 20: INIT
  Section 21: .rsrc
  Section 22: .reloc
```

Note: If found error in running the above commands, run following command then rerun the above command.

```bash
sudo /sbin/ldconfig -v
```

Copy the following string from the terminal output

```bash
PDB GUID: f794d83b0f3c4b7980797437dc4be9e71
Kernel filename: ntkrnlmp.pdb
```

Now run the following commands by changing the paramaters accordingly and providing the correct PDB guid and Kernal filename to create LibVMI config profile:

```bash
cd /tmp
python3 ~/drakvuf/volatility3/volatility3/framework/symbols/windows/pdbconv.py --guid f794d83b0f3c4b7980797437dc4be9e71 -p ntkrnlmp.pdb -o windows7-sp1.json
sudo mv windows7-sp1.json /root
```

## Generate the LibVMI profile.

```bash
sudo su
printf "windows7-sp1 {\n\tvolatility_ist = \"/root/windows7-sp1.json\";\n}" >> /etc/libvmi.conf
exit
```

## Build the DRAKVUF using the following commands

```bash
cd ~/drakvuf
autoreconf -vi
```
In Windows 7 "***Etwmon plugin***" fails , therefore we need to disable this plugin while configuration. This plugin works fine with Windows 10 so no need to disable it if using a windows 10.

```bash
CC=clang ./configure --enable-debug --disable-plugin-etwmon
```
Run ```make``` command to finalize the build
```bash
make
```

Run the following to check if LibVMI is properly configured with the VM.<br>
As an output you will get the PID's of the processes running in your VM.

***Note- Make sure your VM is running and logged in to desktop.***

```bash
sudo vmi-process-list windows7-sp1
```

## Now test if DRAKVUF is correctly generating system logs.

***NOTE: Every DRAKVUF relate command execute only when your terminal path in in 'drakvuf' folder.***

```bash
cd ~/drakvuf
sudo ./src/drakvuf -r /root/windows7-sp1.json -d id
```
You can stop the command by using 'Ctrl+C'. Avoid using 'Ctrl+X' as it freezes the VM.

To get id of virtual machine (use sudo xl list command).

<br>

## Post DRAKVUF steps

After you have setup Xen, DRAKVUF and Windows, we need to create a proper environment to run and analyse malware logs. For setting up the system for automatic log generation we need to perform the following steps: 

1. Install different softwares e.g. Chrome browser, some compilers, Microsoft .NET framework, setup windows media player, etc. The main point is to make the VM look like a actual physical machine. Also many of malware executables requires some other applications to run on. 
2. After you have installed all the softwares and packages you need to make it vulnerable purposely. This is done by disabling Windows defender, update and firewall and also making changes to UAC policies. This step is important for executing malware without failure and in admin approval. <br> ***You can refer to this guide to make your VM vulnerable.*** <br> [Guide for making VM vulnerable](/Installation_instructions/winVulnerable.md)
3. Now make a snapshot of your non infected VM and store it in drakvuf folder. This step is necessary so each executable is injected on a fresh instance of the VM and memory traces of previous executable is removed. <br> ***You can refer to this guide to create snapshot of your VM.*** <br> [Guide for creating snapshot of VM](/Installation_instructions/vmSnapshotRestore.md)

<br>

#### Note: Sometimes VM may get corrupted due to some reason, in this case we just need to remove that logical volume and create it again. [Guide to re-create VM](/Installation_instructions/vmCreation.md)



## Program Execution Tracing Log Generation using DRAKVUF

- **Malware Tracing Command (If file is already present inside the VM)**

```bash
sudo ./src/drakvuf -r /root/windows7-sp1.json -d 1 -x socketmon -t 120 -i 1300 -e “E://zbot.exe” > output.txt
```

- **Injecting and executing executables directly from host(dom0)**

```bash
sudo ./src/drakvuf -r /root/windows7-sp1.json -d 1 -x socketmon -t 120 -i 1300 --write-file /home/pc-03/Desktop/npp.exe E://zbot.exe -e E://zbot.exe > /home/pc-03/Desktop/output.txt
```
Here,

1= ID of virtual machine (use sudo xl list command).<br>
120 = Time duration of log generation in seconds.<br>
1300 = change according to pid of explorer.exe (This PID is of File explorer and not Internet explorer).<br>
***--write-file <absolute/path/of/file/to/be/copied/to/VM> <path/where/file/will/be/stored>***<br>
“E://zbot.exe”= Location of ".exe" file in the created windows VM.<br>
output.txt= Location of the output file. By default it is drakvuf folder location.

<br>

#### Usefull Commmands

Xen version:

```bash
sudo xen-detect
```

List all domains:

```bash
sudo xl list
```

CREATE VM:

```bash
sudo xl create /etc/xen/win7.cfg
```

Connect to QEMU to get VM display:

```bash
gvncviewer localhost
```

(This command does not boots up your VM instead it just open a QEMU emulator and connects it to localhost for VM display.)

Destroy VM forcefully:

```bash
sudo xl destroy id
```

Debug output: (With process injection)

```bash
sudo ./src/drakvuf -r /root/windows7-sp1.json -d 1 -x socketmon -t 120 -i 1300 -e “E:\\zbot\\zbot_1.exe” -v 1> zbot_1.txt
```

<br>

### All XEN and DRAKVUF commands
XEN - [XEN project](https://xenbits.xen.org/docs/unstable/man/xl.1.html)

DRAKVUF - [Website](https://drakvuf.com/)<br>
CLI help:
```bash
./src/drakvuf --help
```
