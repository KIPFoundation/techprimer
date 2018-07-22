### Big Data & Sharding

![KFS Big Data Persistence](../images/tech-primer/KIP-Big-Data-Persistence.png)

<p align="center"> <b>Fig 19:</b> KFS - Big Data Persistence <sup><a href="#references"></a></sup> </p>

$$
\Delta
$$

KFS is designed in an effective manner to support the persistence & operation of large volumes of Big Data.

![KFS Sharding](../images/tech-primer/KFS-Sharding.png)

<p align="center"> <b>Fig 20:</b> KFS - Sharding Approach <sup><a href="#references"></a></sup> </p>

This is made possible by integrating our file system with HDFS<sup><a href="#references">[20]</a></sup> at a fundamental, a highly scalable filesystem for copious amount of Big Data. The native interface is established to create, read, update & delete the information between shared between internal big data file systems and respective KFS network.  