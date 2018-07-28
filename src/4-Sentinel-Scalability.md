## Proposed System

- We propose a composite system with extended abilities to support better network attributes by approaching existing limitations with sentinel scalability, modulated trust, and actor-based throughput.
- The proposed system extends the value measuring capabilities in a blockchain-alike transactional environment with Total Digital Utility(TDU) scores.
- Also, the economical uncertainty in performing transactions on the blockchain is addressed by the method of stable gas costing mechanism.

We discuss each of approaches here in detail:

### Sentinel Scalability

KIP leverages TARA(Ternary Augmented Raft Architecture)<sup><a href="#references">[3]</a></sup>, an enhanced version of RAFT<sup><a href="#references">[4]</a></sup> algorithm, for high-frequency consensus, simplified service discovery & dynamic node management.

TARA provides multi-faceted abilities to manage a network of distributed systems by managing nodes in a graph format. Analogous to vertices connected acyclically by drawing edges in a graph, nodes are connected by an RPC(Remote Procedure Call)<sup><a href="#references">[5]</a></sup> connection between them. The RPC connections establish a network in a hybrid manner.

Each node in the KIP system is defined in the following structure:

``` go

type Node struct {
	state    NodeState		// Whether an Orphan, Follower, Candidate OR a Leader
	role     NodeRole		// Whether is under service, star or master subnet

	eventmux *event.TypeMux // Event multiplexer used between the services of a stack
	config   *Config
	accman   *accounts.Manager

	ephemeralKeystore string         // if non-empty, the key directory that will be removed by Stop
	instanceDirLock   flock.Releaser // prevents concurrent use of instance directory

	serverConfig p2p.Config
	server       *p2p.Server // Currently running P2P networking layer

	serviceFuncs []ServiceConstructor     // Service constructors (in dependency order)
	services     map[reflect.Type]Service // Currently running services

	rpcAPIs       []rpc.API   // List of APIs currently provided by the node
	inprocHandler *rpc.Server // In-process RPC request handler to process the API requests

	ipcEndpoint string       // IPC endpoint to listen at (empty = IPC disabled)
	ipcListener net.Listener // IPC RPC listener socket to serve API requests
	ipcHandler  *rpc.Server  // IPC RPC request handler to process the API requests

	httpEndpoint  string       // HTTP endpoint (interface + port) to listen at (empty = HTTP disabled)
	httpWhitelist []string     // HTTP RPC modules to allow through this endpoint
	httpListener  net.Listener // HTTP RPC listener socket to server API requests
	httpHandler   *rpc.Server  // HTTP RPC request handler to process the API requests

	wsEndpoint string       // Websocket endpoint (interface + port) to listen at (empty = websocket disabled)
	wsListener net.Listener // Websocket RPC listener socket to server API requests
	wsHandler  *rpc.Server  // Websocket RPC request handler to process the API requests

	stop chan struct{} // Channel to wait for termination notifications
	lock sync.RWMutex
}

```

wherein the state and mode attributes are enumerated as follows:

``` go

type NodeState int32

const (
	ORPHAN    NodeState = 0
	FOLLOWER  NodeState = 1
	CANDIDATE NodeState = 2
	LEADER    NodeState = 3
)
var NodeStateMap map[common.Hash]uint


type NodeRole int32

const (
	IDLE_ROLE    NodeMode = 0
	SERVICE_ROLE NodeMode = 1
	STAR_ROLE    NodeMode = 2
	GRAND_ROLE   NodeMode = 3
)
var NodeRoleMap map[common.Hash]uint

```

TARA, based on PBFT hardened RAFT facilitates the network with addition and removal of nodes in a dynamic manner. A formal specifications for the same here:

``` go

func ParseNode(string enodeURL) {
    // Parse node info by its enode:/// URL
}

func addPeer(Node node) {
    // Add the peer into a specific network
}

func (n *Node) setPeerRole() error {
    // Update the status of the node under its type definition
}

func (n *Node) getPeerRole() error {
    // Get the current role of the specified node by its type definition
}

func removePeer(Node node) {
    //  Remove the node from the specified network & revoke all access
}

```

TARA leverages raftpb<sup><a href="#references">[6]</a></sup> - The Protobuf<sup><a href="#references">[7]</a></sup> implementation for RAFT from CoreOS' etcd<sup><a href="#references">[8]</a></sup> package to manage & broadcast network level changes due to addition or removal of peers.

![KIP Sentinel Scalability](images/tech-primer/KIP-Sentinel-Scalability.png)

<p align="center"> <b>Fig 1:</b> KIP - Sentinel Scalability and node management with NTQ <sup><a href="#references"></a></sup> </p>

KIP's approach to achieving sentinel scalability is by observing the interest of the new nodes joining the network, hosted by interested parties called *verifiers*. Each node is vetted for reliability & commercial viability by a benchmarking program that receives the above mentioned interests as input from the owners of the node.

KIP's node benchmark program calculates the reliability based on 2 criteria:

1. **Hardware specifications:**  
  The program collects the host's following hardware information:

	- CPU's architecture,
	- Number of CPU(s),
	- Thread(s) per core,
	- CPU's frequency,
	- MIPS,
	- Total cache size,
	- Total main memory,
	- Available space in the reserved partition,
	- OS type & version, and
	- Kernel build  

	The collected attribute values are aggregated to find the mean score, upon which the node is declared valid under a tier or vice-versa.

**Description 4.1:**

$$
\Delta
$$

1. **Network Specifications:**  
  The benchmark programs then communicates with the node by sending a specific number of arbitrary messages in an encapsulated virtual network to calculate the responsive potential of the node. The measured throughput is compared with the average throughput of all the nodes in the network.


**Description 4.2:**

$$
\Delta
$$

3. **Interest Verification:**  
  The benchmark program also keeps track of the transactions verified by the node for a brief period of time. The monitoring is kept under observation to reserve the interest of the new node in the network & to eliminate bad actor interests from the network. The aggregate of number of transactions dropped over the time observed is compared for shaping the Node Trust Quotient (NTQ) in full.

**Description 4.3:**

$$
\Delta
$$

Upon passing the specification tests, the eligible nodes joining the KIP network is considered "orphan" role with only an ability to become a member of the shadow pool verifying shadow broadcasts, until the KIP's network orchestrator adds the peer into a state channel. Once the node is added into a state channel, the node is assigned the "Follower" role & hence entitled to verify the transactions passed on to its state channels. However, all follower nodes belonging to arbitrary state channels are also responsible for verifying the transactions propagated in the shadow broadcasts shared by the grand leader.

> **Node Lifecycle - Formal Notation:**  
>
> **STEP 1:** Initiate a `Node` instance with `knode://` url  
>
> **STEP 2:** Add the peer into the relevant state channels with `addPeer(node)`  
>
> **STEP 3:** Acknowledge the addition of the node with `chainExtensionMessage`  
>
> **STEP 4:** Node verifies the transactions propagated from both Fast Follow & Shadow Broadcast  
>
> **STEP 5:** Node vests a finite number of KIP tokens & performs its duties for at least minimum duration to contend in the election of star leader  
>
> **STEP 6:** Leadership is transferred to the candidate node with majority votes & shared across the network with `ProposeConfChange()`  
>
> **STEP 7:** The new star leader receives the transactions passed by the classifier from master nodes & maintains leadership by sharing the heartbeat to all following service nodes in the state channels  
>
> **STEP 8:** Star nodes vest an extra number of KIP tokens to contend in the elections for Grand Nodes & changes to the network leadership is again broadcasted across the network  
>
> **STEP 8:** The election occurs again in case of time-out from the leaders at all levels  
>
> **STEP 9:** Nodes are removed due to malicious attempts or cause of faults with `removePeer(node)`  
>
> **STEP 10:** The network changes are broadcasted & process repeats

Every successful modification to the network will be ratified by the `chainExtensionMessage` - a network update heartbeat sent across to all the nodes in the pertaining network by its leader.