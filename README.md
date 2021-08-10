Based on solution developed for [Active Directory Topology Visualization (part 1)]() purpose I’ve made very similar script to have nice picture of defined site links in AD.

I think it’s quite good to know if gap in replication is not caused by lack of site link, etc.

Details:

Therre is queried via _vbs_ script the following DN:

```txt
CN=IP,CN=Inter-Site Transports,CN=Sites,CN=Configuration,DC=my,DC=domain
```

and result is presented in _dot_ syntax formatted file.

(_vbs_ script can be downloaded from [here](/files/getSiteLinks.zip)

Usage:

```cmd
cscript /nologo getSiteLinks.vbs
```

Gallery:

Result of above vbs script can look like as follow:

```dot
GRAPH siteLinks {

    node [fontname=helvetica, image="site.png", labelloc=b, color=white];
 
    Site1 -- HQ;
    Site2 -- Site3;
    Site2 -- HQ;
    Site3 -- HQ;
    Test -- HQ;
    Site2 -- HQ;
    Site4 -- HQ;
    Site5 -- Site6;
    Site5 -- HQ;
    Site6 -- HQ;
    Site6 -- HQ;
    Site7 -- Site3;
    Site7 -- HQ;
    Site3 -- HQ;
    Site8 -- Site4;
    Site8 -- Site9;
    Site8 -- HQ;
    Site4 -- Site9;
    Site4 -- HQ;
    Site9 -- HQ;
    Backup -- HQ;
    Site7 -- Site10;
    Site7 -- HQ;
    Site10 -- HQ;
    Test -- HQ;
 
}
```

and based on it GraphViz can generate:

– dot diagram layout (command: dot *.dot -Tjpg -odot.jpg):



