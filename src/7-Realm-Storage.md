## Storage Realm

As a hybrid mesh, KIP offers the user to store information into permissioned datastores in a decentralized manner with comprehensive abilities to share the stored information to a limited number of audience in an encrypted medium.

### Permissioned Storage

KIP offers a native decentralized file storage system called Krama File System(KFS). The file system offers a few groundbreaking measures to reduce the unneeded cost of storing the data on blockchain.  

KFS stores the information and persists changes to the stored information in cryptographic signatures with new checksum-like signatures representing the change in state. KFS is an extension to the IPFS(InterPlanetary File System)<sup><a href="#references">[18]</a></sup>.

![KFS Permissioned Storage](images/tech-primer/KIP-KFS-Permissioned-Storage.png)

<p align="center"> <b>Fig 18:</b> KFS - Permissioned Storage & Proxy channels for inter-channel data sharing <sup><a href="#references"></a></sup> </p>

KIP initiates permissioned KFS networks by deploying a genesis node with a custom swarm key<sup><a href="#references">[19]</a></sup>. The swarm key shall be shared with other nodes that are willing to participate in the permissioned networks. Evidently, the new nodes joining the network with reference to swarm key have access to the files stored in the genesis node & the other predecessor nodes within the *swarm*.

KIP shall also support the ability to request data via RESTful API from other swarms by bridging two or more disparate swarms, keeping the partition between swarms intact.

> **Distributed Storage & Access - Formal Notation:**  
>
> **STEP 1:** User initiates the transaction by signing with the KIP account private key
>
> **STEP 2:** The transaction is wrapped up with nonce, platform fee, data, value, tier & targeted cluster to reside by diAPP
>
> **STEP 3:** The bundled transaction is transferred to Karma Service Interface orchestration network
>
> **STEP 4:** The KSI extracts the data from the transaction and pushes it to relevant KFS cluster
>
> **STEP 5:** The data is transported to the storage realm (KFS) along with the cluster ID. Rest of the transaction elements are transported to the blockchain
>
> **STEP 6:** The leader node of the cluster reads the incoming payload and stores the content in the Merkle DAG, with the file content uniquely referred with a  multihash checksum.
>
> **STEP 7:** Participating nodes of the specified cluster verifies the content by checking checksum versus content across the Merkle DAG. If data is tempered with or corrupted, KFS network alerts the file owner and suitable measures can be enforced.
>
> **STEP 8:** Nodes support `CREATE` operation by storing the file contents and distributing the update heartbeat within the swarm.
>
> **STEP 9:** Nodes support `READ` operation by accessing the content of the requested file stored in the swarm by its KFS file identifier & transport the content back to the client through data streams.
>
> **STEP 10:** Nodes support `UPDATE` operation by updating the content of the file with a new entry to the Merkle DAG with the new supplied content queried by its KFS file identifier
> 
> **STEP 10:** Nodes support `DELETE` operation by deleting the content of the file and removing the leaf of the Merkle DAG and updating the path.

**Description 7.1:**

$$
\Delta
$$