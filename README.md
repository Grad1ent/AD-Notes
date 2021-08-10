Topology of DFS-R can be easily visualized by using [GraphViz](http://graphviz.org) package.

Based on [Active Directory Topology Visualization (part 1)]() solution it's next script to have clear picture how DFS replication looks like. Design of solution is very the same: _vbs_ script queries AD regarding to DFS replication groups, folders, servers and connections and formats result into _dot_ syntax file. Then _dot_ file is used as input for _GraphViz_ package to generate picture of DFS-R topology.

_vbs_ script can be downloaded [here]()

Usage:

```cmd
cscript /nologo getDFSRTopology.vbs
```

Result:

```dot
DIGRAPH DFSRTopology {

fontname=helvetica;
node [fontname=helvetica, image="server.png", labelloc=b,color=white];

SUBGRAPH cluster_Bold_and_Beautiful {
label = "Group: Bold_and_Beautiful\nFolder: B&B";

FS01_0 [label=FS01];
FS02_0 [label=FS02];

}

SUBGRAPH cluster_OnlySN_PR {
label = "Group: OnlySN_PR\lFolder: PR-SN"

FS03_1 [label=FS03];
FS02_1 [label=FS02];

}

SUBGRAPH cluster_REPL_Maximo_PROD {
label = "Group: REPL_Maximo_PROD\lFolder: PROD_CfR_Archive\lFolder: PROD_CfR_Current"

FS03_2 [label=FS03];
FS02_2 [label=FS02];
FS01_2 [label=FS01];

}

SUBGRAPH cluster_RG_CORPDATA_DATA {
label = "Group: RG_CORPDATA_DATA\lFolder: DATA"

FS02_3 [label=FS02];
FS01_3 [label=FS01];

}

SUBGRAPH cluster_RG_CORPDATA_USERS {
label = "Group: RG_CORPDATA_USERS\lFolder: USERS"

FS01_4 [label=FS01];
FS02_4 [label=FS02];

}

FS02_0 -> FS01_0;
FS01_0 -> FS02_0;
FS02_1 -> FS03_1;
FS03_1 -> FS02_1;
FS01_2 -> FS03_2;
FS02_2 -> FS03_2;
FS01_2 -> FS02_2;
FS03_2 -> FS02_2;
FS03_2 -> FS01_2;
FS02_2 -> FS01_2;
FS01_3 -> FS02_3;
FS02_3 -> FS01_3;
FS02_4 -> FS01_4;
FS01_4 -> FS02_4;

}
```

and based on it here is the picture of DFR replication topology as result of the following command:
```cmd
fdp *.dot -Tjpg -O)
```

<p align="center">
   <img src="/pics/fdp4-300x278.jpg"/>
</p>


