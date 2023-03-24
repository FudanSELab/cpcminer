# Replication Package For Paper: Revealing Code Change Propagation Channels by Evolution History Mining

**CPCminer** is a graph-based approach to mine the frequent change propagation channels (FCPCs) from the software evloution history. The approach consists of following three main steps: 

+ **Step 1: Data Preprocessing**. It extracts the changed code entities and construct the software relationship graphs (SRGs) for every commit to provide complete code changes and SRGs dataset. 
+ **Step 2: Spatial-Temporal Window Processing**. For every commit that satisfies certain criteria (i.e., the core commit), it first establishes a spatial-temporal window (ST-Window), i.e., a limited time and space window, and then constructs the Spatial-Temporal Change Relationship Graph (ST-CRG) by collecting the changed code entities and corresponding relationships within the ST-window.
+ **Step 3: Graph Transformation and Mining**. After gaining all ST-CRGs, it transforms every ST-CRG into a Change Propagation Channel Graph (CPC-G) by reverting the vertices and edges, and then mines frequent CPCs with a frequent subgraph mining algorithm.

Users are suggested to refer to the research paper for more detials.

This project is still in an early stage of development, and we are improving the implementation and documentation. 

Please feel free to contact Daihong Zhou(dhzhou17 at fudan dot edu dot cn) for the replication package.



This repo provides  the replication package for the paper (Baidu Netdisk: the [links](https://pan.baidu.com/s/1kxYhFxe3wgp2-YwAcMFWQw) , and the password：c3v0). It includes the following folders:

* **code**:  the tool CPCMiner.  
* **project**: the git repository used in this study. 
* **neo4j_dataset**: the neo4j dataset contains entity-level software relationship graphs and change information from over 5,032 versions of five well-maintained projects. 
* **mapping**: the data for code entity mapping.
* **graph**: the generated ST-CDGs, CPC-Gs, FCPCs.  
* **result**:  the basic information about the studied projects, the mined frequent change propagation channels (FCPCs), e.g., the top 40 FCPCs, all 243FCPCs.



## Environment

- java 11+
- neo4j 4.2 (https://neo4j.com/download-center/#community)
- python 3.8+
- gspan_mining  (pip install gspan-mining)
- memory 64G+ 

## Quick start
To run CPCMiner, follow the next four steps.  Note that only one project can be analyzed at a time.

**Step 1: Setup the database**

Please copy the related database to the folder `data` (in the root path of Neo4j database), and use Neo4j 4.2 to open the database.  

**Step 2:  Configure the application-dev.yml**
Configure the application-dev.yml under the folder `code` by setting your own configuration. (<u>If you're using the existing database, just configure the database address, username, and password.</u>)

**Step 3:  Run CPCMiner**

1. Genertate database : `java -jar CPCMiner.jar -i -config /home/zdh/cpcminer/application-dev.yml`   (<u>If you're using the existing database, please skip this step.</u>)

- -i: use to genertate database, i.e., SRGs, change info
- -config: the path of `application-dev.yml`

2. Genertate ST-CDG: `java -jar CPCMiner.jar -st -config /home/zdh/cpcminer/application-dev.yml -rd /home/zdh/cpcminer/mapping -pj jmeter -fi jmeter1000_file(Map).txt -mi jmeter_functionEndLineToId(Map).txt -jd method_diff -tw 6 -mc 15 -od /home/zdh/cpcminer/graph`

- -st : use to genertate ST-CDG
- -config: the path of `application-dev.yml`
- -rd: the root path of the mapping results, e.g., `/home/zdh/cpcminer/mapping`
- -pj: the project name, e.g., `jmeter`
- -fi: the mapping results (file), e.g., `jmeter1000_file(Map)_0409.txt` (e.g., the absolute path is `/home/zdh/cpcminer/mapping/jmeter/jmeter1000_file(Map)_0409.txt`)
- -mi: the mapping results (method), e.g., `jmeter_functionEndLineToId(Map).txt`
- -jd: the path of method_diff, eg, `method_diff`
- -tw: the time window (half)
- -mc: the max commit size in the time window
- -od: the output root path (the genertated graph, e.g., ST-CDGs, CPC-Gs) (e.g., the absolute path is `/home/zdh/cpcminer/graph/jmeter`)

3. Genertate CPC-G: `java -jar CPCMiner.jar -cpc -config /home/zdh/cpcminer/application-dev.yml -rd /home/zdh/cpcminer/graph -pj jmeter -fi jmeter1000_file(Map)_0409.txt -stcdg jmeter_m15_io10_k2_tw6_cpcg_nd.txt`

- -cpc: use to genertate CPC-G
- -config: the path of `application-dev.yml`
- -rd: the root path of input (ST-CDGs) / output (the genertated graph, e.g., CPC-Gs)
- -pj: the project name, e.g., `jmeter`
- -fi: the mapping results (file), e.g., `jmeter1000_file(Map)_0409.txt`
- -stcdg: the file name of ST-CDG, e.g., `jmeter_m15_io10_k2_tw6_cpcg_nd.txt`

4. Generate FCPC: `python -m gspan_mining -s 3 -u 2 -w True jmeter_m15_io10_k2_tw6_cpcg_nd.txt > jmeter_m15_io10_k2_tw6_cpcg_nd_3.txt` 

- -s: int, the min support value
- -u:  int, the upper bound of number of vertices of output subgraph
- -w: bool, the output where one frequent subgraph appears in database

