Here is reference to a few quick AD queries.

Dump of AD:

```cmd
csvde -f ad.csv
```

List of Domain Controllers:

```cmd
NLTEST /dclist:<myDomain>
NETDOM QUERY /D:<myDomain> DC
DSQUERY SERVER -o rdn
```

List of FSMO holders:

```cmd
NETDOM QUERY /D:<myDomain> FSMO
DSQUERY SERVER -hasfsmo SCHEMA
DSQUERY SERVER -hasfsmo NAME
DSQUERY SERVER -domain <myDomain> -hasfsmo RID
DSQUERY SERVER -domain <myDomain> -hasfsmo PDC
DSQUERY SERVER -domain <myDomain> -hasfsmo INFR
DCDIAG /s:<myDC> /test:KnowsOfRoleHolders
```

List of Global Catalog holders:

```cmd
DSQUERY SERVER -domain <myDomain> -isgc
NLTEST /dsgetdc:<myDomain> /GC
repadmin /options *
nslookup gc._msdcs.<myDomain>
```

List of Sites:

```cmd
DSQUERY * "CN=Sites,CN=Configuration,DC=<my>,DC=<domain>" -scope onelevel -attr cn
```

Site where myDC belongs:

```cmd
NLTEST /server:<myDC> /DsGetSite
```
```Powershell
Get-WmiObject -Namespace root\rsop\computer -Class RSOP_Session | select site
```
```cmd
reg query \\<myDC>\HKLM\SYSTEM\CurrentControlSet\services\Netlogon\Parameters /v "DynamicSiteName"
```

List of preffered bridgeheads:

```cmd
DSQUERY * "CN=IP,CN=Inter-Site Transports,CN=Sites,CN=Configuration,DC=<my>,DC=<domain>" -attr bridgeheadServerListBL
```

Domain Controller which authenticated my:

```cmd
User account:
    NLTEST /dsgetdc:<myDomain>
    ECHO %LOGONSERVER%	

Computer account:
    NLTEST /sc_query:<myDomain>
    NETDOM verify <myComputer> /domain:<myDomain>
```

All users:

```cmd
DSQUERY * -filter "(&(objectCategory=Person)(objectClass=User)) -attr sAMAccountName
```

Total number of users:

```cmd
DSQUERY USER forestroot -o dn -limit 0 -name * | find /C /V "~~~~"
```

All active users:

```cmd
DSQUERY * -filter "&(objectCategory=user)(userAccountControl=512)" -limit 0
512 - active
514 - disabled
```

Locked users:

```cmd
DSQUERY * -filter "(&(objectCategory=Person)(objectClass=User)(lockoutTime>=1))"
```
