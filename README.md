## Overview

If you have a look closer into [Active Directory Topology Visualization (part 1)](https://github.com/Grad1ent/ActiveDirectoryAndAround/tree/Active-Directory-Topology-Visualization-part-1) solution developed some time ago you will find that _vbs_ script queries one domain controller to find replication topology. It is quick approach to have overview of AD replication ASAP. However it represents viewpoint only of this domain controller and sometimes it doesn’t have to be objective true.

If domain controllers replicate each other without any issues and there isn’t any modification in numbers of them (adding, removing, etc.) topology looks the same on every DC and above solution is enough. But to have proper recognition of condition of AD environment during its modification there is needed something more comprehensive.

Here is my trial to find full overview of AD physical topology and condition of replication as a side effect of quering every domain controller in our environment. Below _vbs_ script queries all DCs found in AD, formats information about sites, servers and connection objects into _dot_ syntax and controls pictures of nodes (domain controllers) and labels of edges (connection objects) to report issues in topology like orphan or not accessible DCs or connection objects just generated and not seen by any other DC.

## Practice

_vbs_ script can be found [here](/files/getReplicationTopology.zip).

Usage:
```cmd
cscript /nologo getReplicationTopology.vbs
```

Example of _dot_ code generated by above _vbs_ script:
```dot
DIGRAPH replicationTopology {
 
	label = "Reference DC: POZ-DC1";
	labelloc = t;
	fontname = helvetica;
	node [fontname = helvetica, image = "server.png", labelloc = b, color = white];
	edge [style = dotted fontname = helvetica fontsize = 8.0];
 
	SUBGRAPH cluster_WRO {
	label = "Site: WRO\lSubnets:\l10.10.10.0/24\l"
 
	WRO_DC1 [label = "WRO-DC1.qa.local" image = "noaccess.png"];
 
	}
 
	SUBGRAPH cluster_POZ {
	label = "Site: POZ\lSubnets:\l10.20.20.0/24\l"
 
	POZ_DC1 [label = "POZ-DC1.qa.local"];
 
	}
 
	SUBGRAPH cluster_WAW {
	label = "Site: WAW\lSubnets:\l10.30.30.0/24\l"
 
	}
 
	POZ_DC1 -> WRO_DC1;
	WRO_DC1 -> POZ_DC1;
}
```

and diagram:
<p align="center">
   <img src="/pics/fdp4-261x300.jpg"/>
</p>

Pictures of nodes used in diagrams:

<p align="left">
   <img src="/pics/server.png"/>
</p>
DC queried by _vbs_ script


<p align="left">
   <img src="/pics/noaccess.png"/>
</p>
DC not queried by _vbs_ script because of communication issue


<p align="left">
   <img src="/pics/orphan.png"/>
</p>
DC not fully removed from AD during [decommission](http://technet.microsoft.com/en-us/library/cc816798%28v=ws.10%29.aspx)

## Gallery
<p align="center">
   <img src="/pics/dot5-276x300.jpg"/>
</p>

<p align="center">
   <img src="/pics/fdp6-211x300.jpg"/>
</p>

<p align="center">
   <img src="/pics/fdp7-300x296.jpg"/>
</p>

## Theory

[How Active Directory Replication Topology Works](http://technet.microsoft.com/en-us/library/cc755994%28v=ws.10%29.aspx)

[KCC and Topology Generation](http://technet.microsoft.com/en-us/library/cc961781.aspx)

[Active Directory Topology Visualization (part 1)](https://github.com/Grad1ent/ActiveDirectoryAndAround/tree/Active-Directory-Topology-Visualization-part-1)
