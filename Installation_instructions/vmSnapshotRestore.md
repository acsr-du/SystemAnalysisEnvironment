# VM Save-file/Snapshot creation 

Creating a save-file or snapshot is an important step for properly generating logs. VM has to be restored to the previous clean state after execution of each malware executable. Restoring the VM to a fresh and clean state removes any instance of the previous executable from memory.
<br>

## VM Snapshot creation

#### 1. Parse the VM and get the VMI process list.

- Create the VM
  ```bash
  sudo xl create /etc/xen/win7.cfg
  ```
- Wait for some time to let windows boot and then get process list
  ```bash
  sudo vmi-process-list windows7-sp1
  ```

  <img title="Parsing and VMI process list" alt="Parsing and VMI process list" src="/Installation_instructions/images/vmipsl.png" width="500" height="600">

  Note down the PID of the process which you want to hijack. In our case we use explorer.exe, whose PID is '1368'.
  
#### 2. Change directory to the folder where you want to store the snapshot file

- In our case we are storing the snapshot.sav file to the drakvuf folder.

  ```bash
  cd ~/drakvuf
  ```
#### 3. Using xen command make a snapshot file of the VM

- Make sure you have made all the necessary changes to your VM before executing this command. Try to avoid "xl create" command after this as we have faced system corruption issue.

  ```bash
  sudo xl save <domain-id> <snapshot-file-name> <vm's/config/file/loacation> 
  ```

  For example:
  ```bash
  sudo xl save 1 snapshot.sav /etc/xen/win7.cfg
  ```
  In this, <br>
  1 = Domain ID of the VM. <br>
  snapshot.sav = Name of the save file. <br>
  /etc/xen/win7.cfg = Absolute path of VM's configuration file. <br>
  
  <img title="Snapshot creation" alt="Snapshot creation" src="/Installation_instructions/images/vmSave.png" width="750" height="350">
  <img title="snapshot.sav" alt="snapshot.sav" src="/Installation_instructions/images/fss.png" width="650" height="500">

<br>
  
## Restoring VM from snapshot/save-file 

#### 1. Change directory to the folder where you have stored snapshot file

- In our case we have saved the snapshot.sav file in the drakvuf folder.

  ```bash
  cd ~/drakvuf
  ```
#### 2. Using xen command restore the snapshot of the VM

- While restoring the absolute path of the config-file come first and then the snapshot name.

  ```bash
  sudo xl restore <vm's/config/file/loacation>  <snapshot-file-name>
  ```

  For example:
  ```bash
  sudo xl restore /etc/xen/win7.cfg snapshot.sav
  ```
  In this, <br>
  /etc/xen/win7.cfg = Absolute path of VM's configuration file. <br>
  snapshot.sav = Name of the save file. <br>
  
  <img title="Snapshot restoration" alt="Snapshot restoration" src="/Installation_instructions/images/vmResore.png" width="650" height="550">

  - ***VM is now restored with all those same process in the memory with same PID which were there during the time of snapshot creation.***

  <img title="VMI process list after restore" alt="VMI process list after restore" src="/Installation_instructions/images/vmipsl2.png" width="650" height="500">

