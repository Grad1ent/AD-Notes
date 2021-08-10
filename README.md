## Overview

Except of well known ports such 389 TCP/UDP, 636 TCP, 3268 TCP, etc. (full overview is here) Active Directory uses a few ones from dynamic pool for replicaton purposes. If we don’t have strict security policy where is allowed only explicity defined traffic in firewalls we can leave default configuration of communication between domain contollers:

1024  – 65535 TCP Dynamic RPC W2K/W2K3

49152 – 65535 TCP Dynamic RPC W2K8+

However if we would like to have more control over AD and SYSVOL replication we can limit above scope to our needs.

1. Restricting Active Directory replication traffic to exemplary 5000 TCP port (0x1388):

```txt
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NTDS\Parameters]
"TCP/IP Port"=dword:00001388
```

#2a. Restricting SYSVOL FRS traffic to exemplary 5050 TCP port (0x13ba):

```txt
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NtFrs\Parameters]
"RPC TCP/IP Port Assignment"=dword:000013ba
```

#2b. Restricting SYSVOL DFS-R traffic to exemplary 5050 TCP port:

```cmd
Dfsrdiag StaticRPC /port:5050 /Member:<myDC>
```

#3. RPC dynamic port allocation to exemplary 6000 – 6050 TCP port pool:

```txt
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Rpc\Internet]
"Ports"=hex(7):36,00,30,00,30,00,30,00,2d,00,36,00,30,00,35,00,30,00,00,00,00,00
@=""
"PortsInternetAvailable"="Y"
"UseInternetPorts"="Y"
```

## Practice

Common operation issues in complex environments, like Event 1722: “The RPC server is unavailable”, are sometimes caused by FW restrictions where are allowed traffics on fixed ports only instead of full RPC dynamic pool. For example just promoted domain controller with default configuration is trying to replicate with remote partner located behind FW via dynamically assigned 49157 TCP port where design of replication topology strictly defines 5000 TCP for example and it is implemented on all DCs and FWs.

To check what ports are used for replication purposes simply query 135 TCP enpoint mapper of each domain contoller. It looks like as follow using portqry.exe:

```cmd
portqry.exe -n <myDC> -e 135 -p TCP
```

and in the result try to find section: MS NT Directory DRS Interface to check AD replication ports:

```txt
...
UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface
ncacn_np:127.0.0.1[\\pipe\\lsass]

UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface
ncacn_np:127.0.0.1[\\PIPE\\protected_storage]

UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface
ncacn_ip_tcp:127.0.0.1[6003]

UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface
ncacn_ip_tcp:127.0.0.1[6004]

UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface
ncacn_ip_tcp:127.0.0.1[5000]

UUID: e3514235-4b06-11d1-ab04-00c04fc2dcd2 MS NT Directory DRS Interface
ncacn_http:127.0.0.1[6005]

...
```
 
and _Frs2_ _Service_ to check DFS-R port:

```txt
...
UUID: 897e2e5f-93f3-4376-9c9c-fd2277495c27 Frs2 Service
ncacn_ip_tcp:127.0.0.1[5050]

...
```

To find out if above RPC ports are fixed or not simply query registry settings of this domain controller:

Restricted Active Directory replication traffic:

```cmd
reg query \\<myDC>\HKLM\SYSTEM\CurrentControlSet\Services\NTDS\Parameters /v "TCP/IP Port"
```

RPC dynamic port allocation:

```cmd
reg query \\<myDC>\HKLM\SYSTEM\CurrentControlSet\Services\NTFRS\Parameters /v "RPC TCP/IP Port Assignment"
``` 

## References

[How to configure a firewall for domains and trusts](https://support.microsoft.com/en-us/kb/179442)

[Active Directory Replication Over Firewalls](http://social.technet.microsoft.com/wiki/contents/articles/584.active-directory-replication-over-firewalls.aspx)

[Restricting AD Replication Traffic between DCs to only a few ports](http://blogs.technet.com/b/luistog/archive/2012/05/08/restricting-ad-replication-traffic-between-dcs-to-only-a-few-ports.aspx)

[Service overview and network port requirements for Windows](http://support.microsoft.com/kb/832017#method1)

[Using PORTQRY for troubleshooting](http://blogs.technet.com/b/askds/archive/2009/01/22/using-portqry-for-troubleshooting.aspx)
