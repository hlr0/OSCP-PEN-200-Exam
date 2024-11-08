# Session Hunter

PsMapExec supports Leo4j's [Invoke-SessionHunter](https://github.com/Leo4j/Invoke-SessionHunter). The SessionHunter method identifies systems with privileged or administrative user sessions, checks whether the current or provided user credentials have administrative access, and, if so, continues with command execution.&#x20;

This is an ideal method through which to filter target acquisition to isolate only the most pertinent targets.

### Optional Parameters

<table data-full-width="false"><thead><tr><th width="193">Parameter</th><th width="128.33333333333331">Value</th><th>Description</th></tr></thead><tbody><tr><td>-SuccessOnly</td><td>N/A</td><td>Only shows targets successful and relevent results</td></tr><tr><td>-Domain</td><td>[Domain]</td><td>Set the Domain for which to run against</td></tr><tr><td>-Command</td><td>[Command]</td><td>Will execute commands over WMI on candidate systems</td></tr><tr><td>-Module</td><td>[Module]</td><td>Will execute specified modules over WMI on candidate systems</td></tr></tbody></table>

### Usage

```powershell
# Without command execution
PsMapExec -Targets [Targets] -Method SessionHunter

# With command execution
PsMapExec -Targets [Targets] -Command ipconfig

# With modules
PsMapExec -Targets [Targets] -Module [Module]
```

<figure><img src="../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
