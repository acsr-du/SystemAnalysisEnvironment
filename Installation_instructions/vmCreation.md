# Guide For VM Creation.


**Note:** It's only applicable if DRAKVUF is already installed.


## Step 1: To remove previous VM:
```
sudo lvremove windows7-sp1 vg
```

## Step 2: Navigate to Disks, select partition, access additional settings, choose format partition as shown in below images.

<img title="Image 1" alt="Figure 1" src="/Installation_instructions/images/disks.png" width="650" height="500">

<img title="Image 2" alt="Figure 2" src="/Installation_instructions/images/disks2.png" width="650" height="500">

<img title="Image 3" alt="Figure 3" src="/Installation_instructions/images/disks3.png" width="650" height="500">




<img title="Image 4" alt="Figure 4" src="/Installation_instructions/images/disks4.png" width="650" height="500">

- It will start formatting the partition.

### Step 3: After successfully formatting the partition, open terminal and run following commands:

**Note :** Here *"sda4"* is the partition id, Verify your sda partition id from Disks.

```
sudo pvcreate /dev/sda4
```

```
sudo vgcreate vg /dev/sda4
```

```
sudo lvcreate -L100G -n windows7-sp1 vg
```

```
sudo gedit /etc/xen/win7.cfg
```

- Configure the windows specification.

```
arch = 'x86_64'
name = "windows7-sp1"
maxmem = 2048
memory = 2048
vcpus = 2
maxvcpus = 2
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
disk = [ 'phy:/dev/vg/windows7-sp1,hda,w', 'file:/home/pc-1/Downloads/windows7.iso,hdc:cdrom,r' ]
```
**Note -** Here, *'maxmem'*, *'memory'* and *'vcpus'* are personal preferences. User can change it according to his need or system specification (Host Machine).


```
sudo xl create /etc/xen/win7.cfg
```

```
gvncviewer localhost
```

 - Install and Setup the windows.

<img title="Image 5" alt="windows installation" src="/Installation_instructions/images/windows.png" width="500" height="350"> <img title="Image 6" alt="windows installation" src="/Installation_instructions/images/windows_desktop.png" width="500" height="350"> 


- VM is successfully created.






