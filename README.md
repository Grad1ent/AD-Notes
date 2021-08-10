## Overview

Lingering objects are not desired entities in AD. If one or more domain controllers are disconnected from environment and back after some period of time (called: [tombstone](http://technet.microsoft.com/pl-pl/library/cc784932%28v=ws.10%29.aspx)), deleted objects can be reintroduced by them. Clearing of AD is serious challenge and requires complex solution in complex environment.

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

Step 1: Cleaning up domain partition on reference DC

Series of commands run against one choosen DC allow to clean up its partition in reference to all other DCs in this domain:

```cmd
repadmin /removelingeringobjects <strong>DC1</strong> DC2guid DC=my,DC=domain
repadmin /removelingeringobjects <strong>DC1</strong> DC3guid DC=my,DC=domain
...
repadmin /removelingeringobjects <strong>DC1</strong> DCnguid DC=my,DC=domain
```

In case of communication issue (because of firewall restriction, etc.) finish clearing process of chosen DC with the rest of DCs and begin again Step 1 with failured ones:

