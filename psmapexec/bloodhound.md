# BloodHound

## Description

PsMapExec facilitates the creation of cipher queries that can be easily pasted into the BloodHound GUI. When employing PsMapExec, cipher queries are automatically built into a single query file in `$PWD\PME\BloodHound\Query.txt`.

<figure><img src="../.gitbook/assets/image (2134).png" alt=""><figcaption></figcaption></figure>

The query file is a single file that expands overtime with usage of PsMapExec. The query file automatically populates the following information

* Owned Users
* Owned Computers
* Credential information (RC4, AES256, Plaintext Passwords)
* AdminTo paths
* RDP Paths

Using PsMapExec for the following activities will update the query file as it finds information

* `-Method Spray:` Mark user as owned, updated node property with credential)
* `-Method RDP:` Builds a "CanRDP" path between the user and accessible system
* `-Method WMI/WinRM/SMB:` Mark the computer as owned, build a "AdminTo" path from user to computer
* `-Module eKeys/LogonPasswords:` Dumps credential information on target system, marks the system as owned, marks any dumped users as owned and updates their node properties with any found credentialed information.

For example, lets say we have managed to dump encryption keys and logon passwords on a remote host and found the yap-yap's credential information in memory, after pasting the updated query file into BloodHound GUI, we see the node properties are updated to contain credentialed information, as well as marking the user as owned.\


<figure><img src="../.gitbook/assets/image (2129).png" alt=""><figcaption></figcaption></figure>

In another instance, lets say we check admin access across the domain with Yap-Yap's now discovered credentials, we find Yap-Yap has administrative access to MDTSRV@Security.local. Again, the query file will update to mark MDTSRV as owned and also build a "AdminTo" path between Yap-Yap and MDTSRV.\


<figure><img src="../.gitbook/assets/image (2130).png" alt=""><figcaption></figcaption></figure>

Checking acess cross the domain with Yap-Yap's account with RDP also updates the query file to build a path from the user to the target system for "CanRDP". In this case, the system SRV2012 was not marked as owned as RDP access does not imply administrative control.

<figure><img src="../.gitbook/assets/image (2131).png" alt=""><figcaption></figcaption></figure>

###
