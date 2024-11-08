# GenRelayList / SMB Signing

PsMapExec supports SMB signing checks to determine which specified targets have signing enabled

Output for systems which do not require SMB signing will be stored in `$pwd\PME\SMB\SigningNotRequired.txt`

### Optional Parameters

<table data-full-width="false"><thead><tr><th width="193">Parameter</th><th width="128.33333333333331">Value</th><th>Description</th></tr></thead><tbody><tr><td>-Domain</td><td>Domain</td><td>Set the Domain for which to run against</td></tr><tr><td>-SuccessOnly</td><td>N/A</td><td>Only shows results where SMB signing is not required</td></tr></tbody></table>

### Usage

```powershell
PsMapExec -Targets All -Method GenRelayList
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

\
By default, the results are parsed and a list of hostnames are written to disk in the PME folder for all hosts which do not require signing.
