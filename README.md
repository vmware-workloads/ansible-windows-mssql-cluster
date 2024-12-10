# Windows SQL Cluster Deployment with Aria & AAP

This will automate the creation of a Windows SQL Cluster, using Aria Automation and AAP and consuming vSAN iSCSI storage. 


## REQUIREMENTS:

* Windows image (see below)
* Existing NSX network segment in Aria to consume, with enough free IPs available
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
* Install VM tools
* Ensure the user 'Administrator' is enabled and a password is set
* Enable remote desktop
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

* **IMPORTANT**: To use the .ssh directory for authorized keys, comment out "Match Group administrators" in the file `%programdata%\ssh\sshd_config`
  

### 3. VC & Aria Actions

- Power down the VM
- Eject the CDROM & set to 'Client Device' / Passthrough on the VM
- Convert the VM to a Template
- Create a new image mapping to the template in Aria Automation
- Create a Blueprint/Template to deploy (see `aria_installer\aria_blueprint.yaml`)
    - The installer works (see the readme in the `aria_installer` directory) but will also install a vro workflow that isn't currently used
- Create a Customization spec `windows-template` that contains the credentials for the VM


### 4. Ansible Setup

- Create a Credential with:

    * User/password for the Windows VMs
	* Escalation method as `runas`
	* Priv. Escallation User: `Administrator`
	* Priv. Escallation Pw: `(Administrator's password)`
	
	
	
## TO DO

- Migrate vSAN iSCSI from ABX to vRO and pull into this project
- Make iSCSI optional and include an option for a network share
- Automate the Customization spec creation in vRO
- Capture inputs in the blueprint for the SQL scripts
- Create first 'release'


	






