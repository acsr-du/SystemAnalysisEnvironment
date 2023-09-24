# Guide For Making Windows Vulnerable

This is a guide for making windows vulnerable for successful execution of malware. This is generally done by *"Cyber Security Researchers"* for *"Malware Analysis"* purposes.  

## Step 1: Disable Windows Firewall And Defender.

- Navigate to control panel and set view by to small icons.

<img title="Image 1" alt="Figure 1" src="/Installation_instructions/images/view-by.PNG" width="650" height="500">

- Now, click on windows defender.

<img title="Image 2" alt="Figure 2" src="/Installation_instructions/images/control-defender1.png" width="650" height="500">

- After opening defender, click on tools and then select options.

<img title="Image 3" alt="Figure 3" src="/Installation_instructions/images/defender1.PNG" width="650" height="500">

- From options tab select administrator and untick *"Use this program"*.

<img title="Image 4" alt="Figure 4" src="/Installation_instructions/images/defender2.PNG" width="650" height="500">

- Click on save.

<img title="Image 5" alt="Figure 5" src="/Installation_instructions/images/defender3.PNG" width="650" height="500">

- Windows defender is successfully turned off.

<img title="Image 6" alt="Figure 6" src="/Installation_instructions/images/defender4.PNG" width="650" height="500">

- Now go back to control panel and click on windows firewall.

<img title="Image 7" alt="Figure 7" src="/Installation_instructions/images/firewall1.PNG" width="650" height="500">

- Click on *"Turn Windows Firewall on or off"*

<img title="Image 8" alt="Figure 8" src="/Installation_instructions/images/firewall2.PNG" width="650" height="500">

- Select turn off windows firewall on both *"Home and Public network location settings"* and click ok.

<img title="Image 9" alt="Figure 9" src="/Installation_instructions/images/firewall3.PNG" width="650" height="500">

- Now windows firewall is successfully turned off.

## Step 2: Changes To Group Edit Policy.

- Goto start and type *"group"* in search bar then click on *"Edit group policy"*.
- Expand Computer Configuration -> Windows Settings -> Security Settings -> Local Policies -> Security Options.
- Scroll down to the User Account Control options.

<img title="Image 10" alt="Figure 10" src="/Installation_instructions/images/group-edited1.png" width="650" height="500">

- Right click the *“User Account Control: Behavior of the elevation prompt for administrators in Admin Approval Mode”* and select Properties.

<img title="Image 11" alt="Figure 11" src="/Installation_instructions/images/group-edited2.png" width="650" height="500">

- Select *"Elevate without prompting"* from drop down menu then click apply and ok.

<img title="Image 12" alt="Figure 12" src="/Installation_instructions/images/group-edit2.PNG" width="650" height="500">

- Right click the *“User Account Control: Detect application installations and prompt for elevation”* and select Properties.

<img title="Image 13" alt="Figure 13" src="/Installation_instructions/images/group-edited3.png" width="650" height="500">

- Click on *"Disabled"* then apply and hit ok.

<img title="Image 14" alt="Figure 14" src="/Installation_instructions/images/group-edit3.PNG" width="650" height="500">

- Right click the *“User Account Control: Run all administrators in Admin Approval Mode”* and select Properties.

<img title="Image 15" alt="Figure 15" src="/Installation_instructions/images/group-edited4.png" width="650" height="500">

- Click on *"Disabled"* then apply and hit ok.

<img title="Image 16" alt="Figure 16" src="/Installation_instructions/images/group-edit4.PNG" width="650" height="500">

- Changes to edit policy is done.

## Step 3: Give Full Permission Control To *"E"* Drive

- Right click on New Volume(E:) then click on properties.

<img title="Image 17" alt="Figure 17" src="/Installation_instructions/images/Picture1.png" width="650" height="500">

- Click on security then select edit.

<img title="Image 18" alt="Figure 18" src="/Installation_instructions/images/Picture2.png" width="650" height="500">

- Now click on each “user names” and allow “Full control” for all “user names”, then apply and hit ok.

<img title="Image 19" alt="Figure 19" src="/Installation_instructions/images/Picture3.png" width="650" height="500">

- Finally restart the windows.
- Windows is now vulnerable and ready for malware execution.












