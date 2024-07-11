# Windows SQL Cluster Deployment with Aria & AAP

This will automate the creation of a Windows SQL Cluster, using Aria Automation and AAP and consuming vSAN iSCSI storage. 


## REQUIREMENTS:

* Windows image (see below)
* Linux jump server (we used an Ubuntu VM)
* Ansible Automation Platform (configured with credentials, etc. as below)
* Aria Configured with the ABX Extensions for [Ansible API](https://github.com/vmware-workloads/aap-api) & [vSAN iSCSI](https://github.com/vmware-workloads/vSAN-iSCSI-ABX/tree/main)
* vSAN with the iSCSI service enabled (just the service, we will automate LUN/target creation)
* Patience



## STEPS


### 1. Obtain Image

This has been tested using a Vanillia Windows Server 2022 Eval Image from Microsoft. <br>
At the time of writing, the image can be obtained from: <br>
[https://go.microsoft.com/fwlink/p/?LinkID=2195280&clcid=0x409&culture=en-us&country=US](https://go.microsoft.com/fwlink/p/?LinkID=2195280&clcid=0x409&culture=en-us&country=US)

Download the ISO & create a new VM in vCenter


### 2. Image Prep

Broadly, the steps to prepare the image are as follows:

* Upload image to VC. Create a new VM & Power on.
* Launch a Remote Console to the VM
* Ensure the user 'Administrator' is enabled and a password is set
* Enable remote desktop
* Install Cloudbase Cloud-Init:

  * Download from here: <br>
                  https://github.com/cloudbase/cloudbase-init?src=so_5703fb3d92c20&cid=70134000001M5td
                
  * Follow the specific instructions as below, update the config files, but **DO NOT** run sysprep: <br>
                   https://docs.vmware.com/en/VMware-Aria-Automation/8.17/Using-Automation-Assembler/GUID-6A17EBEA-F9C3-486F-81DD-210EA065E92F.html


* Install and configure OpenSSH, inserting the public key of the Linux jump-server, as below:

``` powershell
# install openssh
Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*' | Add-WindowsCapability -Online
Start-Service sshd
Set-Service -Name sshd -StartupType 'Automatic'

# make sure fw rule is in place (will error if already exists)
New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22

# generate the key-pair and configure agent
ssh-keygen -t ed25519
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
ssh-add $env:USERPROFILE\.ssh\id_ed25519

# set the default shell to be pwsh
New-ItemProperty -Path "HKLM:\SOFTWARE\OpenSSH" -Name DefaultShell -Value "C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -PropertyType String -Force

# add to authorized keys
$authorized_key = "< public key of jump-host > "

New-Item -Force -ItemType Directory -Path $env:USERPROFILE\.ssh; Add-Content -Force -Path $env:USERPROFILE\.ssh\authorized_keys -Value $authorizedKey

```

* **IMPORTANT**: Comment out "Match Group administrators" in the file `%programdata%\ssh\sshd_config`
  

### 3. VC & Aria Actions

- Eject CDROM & set to 'Client Device' on the VM
- Convert VM to Template
- Create a new image mapping to the template in Aria
- Create a Blueprint/Template to deploy (see example_blueprint.yaml)


### 4. Ansible Setup

- Create a Credential with:

	* The private key of the jumpserver
	* Escalation method as 'runas'
	* Priv. Escallation User: 'Administrator'
	* Priv. Escallation Pw: (Administrator's password)


	






