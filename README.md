## Overview

Lingering objects are not desired entities in AD. If one or more domain controllers are disconnected from environment and back after a while exceeding [tombstone](http://technet.microsoft.com/pl-pl/library/cc784932%28v=ws.10%29.aspx) life time deleted objects can be reintroduced by them. Clearing of AD is serious challenge and requires complex solution in complex environment.

Microsoft prepared simple tool to perform proper removal. However using it depends on design of the environment. Because connections between all sites are not always fully meshed, lacks in “seeing” domain controllers each other is mitigated by simple trick: one domain controller in particular domains is chosen as reference server for its own domain partition and is used by any other domain controller with global catalog function from other domains as reference source. The best is PDC because it should be accessible at least from any domain controller in its own domain and in theory from other domains. However it’s not really manadatory and any DC can be used. In rare cases of communication issue there is needed additional step described below.

## Practice

Below procedure can be used for effective removal of lingering objects in entire forest. It bases on preparing reference domain controller with clean, writable domain partition, and using it as an authoritative source for any other domain controller holding write (DC) or read-only (GC) version of this partition.

Solution is covered by using following command:

```cmd
repadmin.exe /removelingeringobjects <targetDC> <sourceDCGUID> <partitionDN> | /advisory_mode
```

Note:

sourceDCGUID can be found in several ways:

```cmd
repadmin /showreps <myDC>
nslookup -q=CNAME _msdcs.<my>.<domain>
dsquery * "CN=NTDS Settings,CN=<myDC>,CN=Servers,CN=<mysite>,CN=Sites,CN=Configuration,DC=<my>,DC=<domain>" -scope base -attr objectGuid
```

This procedure requires to finish three steps in every domain in entire forest:

### Step 1: Cleaning up domain partition on reference DC

Series of commands run against one choosen DC allow to clean up its partition in reference to all other DCs in this domain:

```cmd
repadmin /removelingeringobjects <strong>DC1</strong> DC2guid DC=my,DC=domain
repadmin /removelingeringobjects <strong>DC1</strong> DC3guid DC=my,DC=domain
...
repadmin /removelingeringobjects <strong>DC1</strong> DCnguid DC=my,DC=domain
```

In case of communication issue (because of firewall restriction, etc.) finish clearing process of chosen DC with the rest of DCs and begin again Step 1 with failured ones:

<p align="center">
   <img src="/pics/linger1-300x111.jpg"/>
</p>

### Step 2: Cleaning up writable version of domain partition on remaining DCs

Series of commands run against all other DCs of affected domain allow to clean up their partitions in reference to DC choosen in Step 1:

```cmd
repadmin /removelingeringobjects DC2 DC1guid DC=my,DC=domain
repadmin /removelingeringobjects DC3 DC1guid DC=my,DC=domain
...
repadmin /removelingeringobjects DCn DC1guid DC=my
```

In case of communication issue repeat Step 2 with failured DCs:

<p align="center">
   <img src="/pics/linger2-300x113.jpg"/>
</p>

### Step 3: Cleaning up read-only version of domain partition on all GCs in entire forest

Series of commands run against all GCs located in different domains allow to clean up their read-only version of affected domain partitions in reference to any DCs from Step 1 or Step 2.

```cmd
repadmin /removelingeringobjects AB1 <strong>DC1guid</strong> DC=my,DC=domain
repadmin /removelingeringobjects CD2 <strong>DC1guid</strong> DC=my,DC=domain
...
repadmin /removelingeringobjects XYn <strong>DC1guid</strong> DC=my,DC=domain
```

In case of communication issue replace DC1guid with any other one from Step 1 or 2. If all DC guids don’t allow to establish proper communication between GC under clearing process and any DC which is owner of affected domain partition, use the nearest last GC which walked through Step 3 without failure, to re-host this partition:

```cmd
repadmin /unhost <myGC> DC=root,DC=local
repadmin /rehost <myGC> DC=root,DC=local <clean GC>
```

<p align="center">
   <img src="/pics/linger3-300x223.jpg"/>
</p>

## References
The following events are logged during clearing lingering objects:

Events logged on DC without lingering objects:
* 1388: SRC is off, lingering objects appeared
* 1988: SRC is on, lingering objects blocked
* 2042: Too long since source replication

Events logged on DC with lingering objects during:
```cmd
repadmin /removelingeringobjects … /advisory_mode
```
* 1938: Starting detection summary
* 1946: For each lingering object detected
* 1942: Final detection summary

Events logged on DC with lingering objects during:
```cmd
repadmin /removelingeringobjects …
```
* 1937: Starting removal summary
* 1945: For each lingering object detected and removed
* 1939: Final removal summary

[Fixing Replication Lingering Object Problems (Event IDs 1388, 1988, 2042)](http://technet.microsoft.com/en-us/library/cc738018%28v=ws.10%29.aspx)

[Event ID 1388 or 1988: A lingering object is detected](http://technet.microsoft.com/en-us/library/cc780362%28v=ws.10%29.aspx)

[Lingering objects may remain after you bring an out-of-date global catalog server back online](http://support.microsoft.com/kb/314282)

[Outdated Active Directory objects generate event ID 1988 in Windows Server 2003](http://support.microsoft.com/kb/870695)

[How to find and remove lingering objects in Active Directory](http://sandeshdubey.wordpress.com/2011/10/09/how-to-find-and-remove-lingering-objects-in-active-directory/)

[Clean that Active Directory forest of lingering objects](http://blogs.technet.com/b/glennl/archive/2007/07/26/clean-that-active-directory-forest-of-lingering-objects.aspx)

[Repadmin for Experts](http://technet.microsoft.com/en-us/library/cc811549%28v=ws.10%29.aspx)

[Enable strict replication consistency](http://technet.microsoft.com/en-us/library/cc784245%28v=ws.10%29.aspx)

[Clean that Active Directory forest of lingering objects](https://docs.microsoft.com/en-us/archive/blogs/glennl/clean-that-active-directory-forest-of-lingering-objects)
