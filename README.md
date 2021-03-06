# Xymon-VMware
Gathers information from VMware using PowerCLI and uploads it to Xymon

# Details
Queries the VMware ESXI Environment based on the host that is running the script. Gathers snapshot information such as Snapshot Name and Creation Date. Creates a new column in Xymon and depending on the age of the snapshot the icon will change to Red, Yellow or Green.

# Prerequisite
- Ubuntu 18.04 x64 
- Xymon 4.3.17
- PowerShell Core
- VMware PowerCLI

# Links
- [Installing Powershell Core on Linux](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-linux?view=powershell-6#ubuntu-1804)
- [Install PowerCLI on Ubuntu 18.04](https://www.altaro.com/vmware/install-powercli-ubuntu-linux-18-04-lts/)

# Installing PowerShell Core on Linux
Run the following commands on the linux terminal.

**1. Download the Microsoft repository GPG keys**

`wget -q https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb`

**2. Register the Microsoft repository GPG keys**

`sudo dpkg -i packages-microsoft-prod.deb`

**3. Update the list of products**

`sudo apt-get update`

**4. Enable the "universe" repositories**

`sudo add-apt-repository universe`

**5. Install PowerShell**

`sudo apt-get install -y powershell`

**6. Start PowerShell, verify that it runs**

`pwsh`

# Installing PowerCLI

1. Open a terminal in linux and run `pwsh` to start Powershell

2. Open a terminal and run the following command, select Y at the prompt to continue

	```PowerShell 
	Install-Module -Name VMware.PowerCLI
	```
	
3. To test if the installation was successful run

	```PowerShell 
	Get-Module VMware.PowerCLI -ListAvailable
	```

4. To avoid a having certificate errors prevent you from connecting to a esxi Server run

	```PowerShell 
	Set-PowerCLIConfiguration -InvalidCertificateAction Ignore
	```

# Connecting to a VCenter Server

1. From terminal run `pwsh`
	```PowerShell 
	Connect-VIServer -Server someviserver -User thehoff -Password Baywatch
	```

2. Get Virtual Machine Snapshot Information

	```PowerShell 
	Get-VM | Get-Snapshots
	```

# Cronjob
Runs the snapshots.ps1 script at 9am, 12pm, 3pm every day

`0 9,12,15 * * * /usr/bin/pwsh /home/someuser/snapshots.ps1`

How to determine if the cron job ran

`cd /var/log`

`cat syslog.1 syslog | grep snapshots`


# Alerts
I didn't want super noisy alerts. This will only allow RED alerts to be sent every 4 hours from 8am - 4pm every day. Because of the way I have my cronjob setup i'm getting alerts at 9am and 1pm.

The trick to getting the timing to work is to put the REPEAT on the 2nd line with the email.

`HOST=%.* SERVICE=snapshots COLOR=RED TIME=*:0800:1600`

`MAIL email@somedomain.com REPEAT=240`

# Snapshot Credentials
I recommend using a dedicated service account for the purposes of running this script and connecting to VMware. It should ONLY have permissions to access the snapshots on the VMware hosts you specify.

# Snapshot Age
- There should now be a new column on the Xymon Dashboard called Snapshots
- When a new snapshot is found in VMware the colored icon will change based on the age of the snapshot
- The age at which the colors change from GREEN to YELLOW to RED is configurable

__As of right now the colored icons correspond to the following:__

![Xymon Green](https://raw.githubusercontent.com/techspence/Xymon-VMSnap/master/readme-img/green.gif) No snapshots found

![Xymon Green Recent](https://raw.githubusercontent.com/techspence/Xymon-VMSnap/master/readme-img/green-recent.gif) snapshot < 3 days old (this icon is used the first time the snapshots.ps1 script runs even if no snapshots are found)

![Xymon Yellow Recent](https://raw.githubusercontent.com/techspence/Xymon-VMSnap/master/readme-img/yellow-recent.gif) snapshot > 3 days old (may also see  )

![Xymon red Recent](https://raw.githubusercontent.com/techspence/Xymon-VMSnap/master/readme-img/red-recent.gif) snapshot >= 5 days old (may also see  )

__**Reminder:**__ Old snapshots > 5 days old should be removed. If there is a need to have a snapshot longer than 5 days it may be more reasonable to take a point-in-time backup rather than preserve the snapshot. Snapshots can be come quite large if not removed shortly after they're taken.

# Example Xymon Snapshot Output
```
ESXi Server: myesxiserver
Virtual Machine: myvirtualmachine
Snapshot(s):

 - 04/03/2019 20:12:51 - "test-snapshot-04032019" - 4.08GB - 0.54 Days(s)

(Green < 3 days old)
```
