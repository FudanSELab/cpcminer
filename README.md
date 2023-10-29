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
* **manual_study**: the manual validation data. 
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

## Examples (ST-CDG)
```
t # <graph-id>
v <node-id> <node-name-id> <node-name> 
e <node-name-id # node-name-id # Relationship> 
m <commit-id>  // This is the commit id for the core commit
c <commit-id> <changed code entities> // Only changed code entities relate to the core entity
F <commit-id> <changed files> // Only changed files relate to the core file
```
Here is an example in Tomcat
```
t # 9
v 0 F1429 java/org/apache/tomcat/util/net/AbstractJsseEndpoint.java
v 1 M19282 java/org/apache/tomcat/util/net/SecureNioChannel.java#FSecureNioChannel#Thandshake(boolean,boolean)#M
v 2 M21749 java/org/apache/tomcat/util/compat/Jre9Compat.java#FJre9Compat#TsetApplicationProtocols(SSLParameters,String[])#M
v 3 M19244 java/org/apache/tomcat/util/net/SecureNio2Channel.java#FSecureNio2Channel#ThandshakeInternal(boolean)#M
v 4 M21752 java/org/apache/tomcat/util/compat/JreCompat.java#FJreCompat#TgetApplicationProtocol(SSLEngine)#M
v 5 F1299 java/org/apache/tomcat/util/compat/Jre9Compat.java
v 6 M18737 java/org/apache/tomcat/util/net/AbstractJsseEndpoint.java#FAbstractJsseEndpoint#TcreateSSLEngine(String,List<Cipher>,List<String>)#M
v 7 F1300 java/org/apache/tomcat/util/compat/JreCompat.java
v 8 M21750 java/org/apache/tomcat/util/compat/JreCompat.java#FJreCompat#TsetApplicationProtocols(SSLParameters,String[])#M
v 9 V0 org.apache.tomcat.util.compat.Jre9Compat.setApplicationProtocolsMethod
v 10 V1 org.apache.tomcat.util.compat.Jre9Compat.m2
v 11 V2 org.apache.tomcat.util.compat.JreCompat.sm
v 12 F1448 java/org/apache/tomcat/util/net/SecureNio2Channel.java
v 13 F1449 java/org/apache/tomcat/util/net/SecureNioChannel.java
e M19244#M21752#CALL
e F1300#V2#CONTAIN
e F1449#M19282#CONTAIN
e F1299#V1#CONTAIN
e F1448#M19244#CONTAIN
e M19282#M21752#CALL
e M18737#M21750#CALL
e F1299#F1300#EXTENDS
e F1299#M21749#CONTAIN
e F1300#M21750#CONTAIN
e F1299#V0#CONTAIN
e F1300#M21752#CONTAIN
e F1429#M18737#CONTAIN
m tomcat(d4fb4f8d1fa021d0d0b290a7ef034eae33973f94)
c tomcat(d4fb4f8d1fa021d0d0b290a7ef034eae33973f94) #M21750#V0#V1#M21749#V2
c tomcat(9c25b79b0240d6362e8051996af3e3a8456cd9b6) 
c tomcat(2a0a6d0bb1e0c6cfcbd3c61054dc2db53de2e2b3) #M21752
c tomcat(c8dcfee6333a0d5d677403cc80665b262dbc73de) 
c tomcat(04a5e59a6b03a4aa1d411eabf1f1d8f5a1f6d1e3) 
c tomcat(a77346f90db4895d4a80db146ebf0d75ff074762) 
c tomcat(5ac9eaf7144ce1f8d21955f0fb4e4f5e6d84c91d) #M19282#M18737#M19244
c tomcat(c35cd4bf5e3affd1454e3708d02b783ce91660ed) 
f tomcat(d4fb4f8d1fa021d0d0b290a7ef034eae33973f94) #F1299#F1300
f tomcat(9c25b79b0240d6362e8051996af3e3a8456cd9b6) 
f tomcat(2a0a6d0bb1e0c6cfcbd3c61054dc2db53de2e2b3) #F1299#F1300
f tomcat(c8dcfee6333a0d5d677403cc80665b262dbc73de) 
f tomcat(04a5e59a6b03a4aa1d411eabf1f1d8f5a1f6d1e3) 
f tomcat(a77346f90db4895d4a80db146ebf0d75ff074762) 
f tomcat(5ac9eaf7144ce1f8d21955f0fb4e4f5e6d84c91d) #F1429#F1448#F1449
f tomcat(c35cd4bf5e3affd1454e3708d02b783ce91660ed) 
```
## Examples (CPC-G)
```
t # <graph-id>
v <node-id> <Relationship> 
e <node-id> <node-id> <lable_L1@L2> <file_0> <file_1> <file_2> <time_diff>
i <the original id of ST-CDG>  
```
Here is an example in Tomcat
```
t # 8
v 0 CALL
v 1 CALL
v 2 CALL
v 3 EXTENDS
e 0 1 A@CF 1300#java/org/apache/tomcat/util/compat/JreCompat.java 1448#java/org/apache/tomcat/util/net/SecureNio2Channel.java 1429#java/org/apache/tomcat/util/net/AbstractJsseEndpoint.java 22607.33 d4fb4f8d1fa021d0d0b290a7ef034eae33973f94#2a0a6d0bb1e0c6cfcbd3c61054dc2db53de2e2b3#5ac9eaf7144ce1f8d21955f0fb4e4f5e6d84c91d
e 1 3 A@CF 1300#java/org/apache/tomcat/util/compat/JreCompat.java 1429#java/org/apache/tomcat/util/net/AbstractJsseEndpoint.java 1299#java/org/apache/tomcat/util/compat/Jre9Compat.java 658.67 d4fb4f8d1fa021d0d0b290a7ef034eae33973f94#2a0a6d0bb1e0c6cfcbd3c61054dc2db53de2e2b3#5ac9eaf7144ce1f8d21955f0fb4e4f5e6d84c91d
e 0 2 A@CF 1300#java/org/apache/tomcat/util/compat/JreCompat.java 1448#java/org/apache/tomcat/util/net/SecureNio2Channel.java 1449#java/org/apache/tomcat/util/net/SecureNioChannel.java 658.67 2a0a6d0bb1e0c6cfcbd3c61054dc2db53de2e2b3#5ac9eaf7144ce1f8d21955f0fb4e4f5e6d84c91d
e 1 2 A@CF 1300#java/org/apache/tomcat/util/compat/JreCompat.java 1429#java/org/apache/tomcat/util/net/AbstractJsseEndpoint.java 1449#java/org/apache/tomcat/util/net/SecureNioChannel.java 22607.33 d4fb4f8d1fa021d0d0b290a7ef034eae33973f94#2a0a6d0bb1e0c6cfcbd3c61054dc2db53de2e2b3#5ac9eaf7144ce1f8d21955f0fb4e4f5e6d84c91d
e 2 3 A@CF 1300#java/org/apache/tomcat/util/compat/JreCompat.java 1449#java/org/apache/tomcat/util/net/SecureNioChannel.java 1299#java/org/apache/tomcat/util/compat/Jre9Compat.java 658.67 d4fb4f8d1fa021d0d0b290a7ef034eae33973f94#2a0a6d0bb1e0c6cfcbd3c61054dc2db53de2e2b3#5ac9eaf7144ce1f8d21955f0fb4e4f5e6d84c91d
e 0 3 A@CF 1300#java/org/apache/tomcat/util/compat/JreCompat.java 1448#java/org/apache/tomcat/util/net/SecureNio2Channel.java 1299#java/org/apache/tomcat/util/compat/Jre9Compat.java 658.67 d4fb4f8d1fa021d0d0b290a7ef034eae33973f94#2a0a6d0bb1e0c6cfcbd3c61054dc2db53de2e2b3#5ac9eaf7144ce1f8d21955f0fb4e4f5e6d84c91d
i 9
```