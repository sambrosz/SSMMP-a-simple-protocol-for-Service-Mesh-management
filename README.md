# SSMMP: Simple Service Mesh Management Protocol 
  <img src="/ssmmp-pictures.pptx.svg" />

# A short introduction to SSMMP
A simple protocol for Service Mesh management is proposed. 
The protocol specification consists of the formats of messages, and the actions taken by senders and recipients.
<br>
The idea is that microservices of Cloud-Native Application (CNApp) should be also involved in configurations of their communication sessions. It does not interfere with the business logic of the microservices and requires only minor and generic modifications of the microservices codebase, limited only to network connections. 
<br>
The proposed protocol has been implemented for simple social media CNApps as a proof of concept.
The implementations clearly show that SSMMP should be viewed (by developers) as an integral part of CNApps.
<br>
The main actors of the protocol are: Manager, agents, and running instances of services (API Gateways, regular microservices, and BaaS services). 
<br>
There may be two (or more) running instances of the same service. 
Manager communicates only with the agents. 
Agent, on a node, communicates with all service instances running on that node. 
<br>
Within the framework of SSMMP, any service instance (running on a node) can only communicate with its agent on that node. 
<br>
Agent has a service repository at its disposal. It consists of bytecodes of services that can be executed (as service instances) by the agent on this node. 
<br>
The agent (as an application) should have operating system privileges to execute applications, and to kill application processes.
<br>
Agent acts as an intermediary in performing the tasks assigned by the Manager.
<br>
All service instance executions as well as shutting down running instances are controlled by the Manager through its agents.
Each agent must register with Manager so that the network address of its node and its service repository are known to Manager.
At a request of Manager, agent can execute instances of services whose bytecodes are available in its repository or shut down these instances. 
<br>
Once a service instance is executed, it initiates the SSMMP communication session with its agent.
The network address of the agent is, of course, localhost for the all service instances running on the same node (host). The port number of the agent (to communicate with its service instances) is fixed for SSMMP, and is the same for all the agents.
<br>
The agent can monitor the functioning of service instances running on its node (in particular, their communication sessions), and report their status to Manager.
<br>
Manager can also shut down (via its agent) a running instance that is not being used, is malfunctioning, or is being moved to another node.
Manager controls the execution of CNApps in accordance with the policy defined by the Cloud provider. 
<br>
Current state of the control as well as its history are stored in a dedicated database DB of Manager. Manager knows the service repositories of all its agents. 
<br>
The Knowledge-base of Manager consists of abstract graphs of CNApps, i.e. the CNApps that can be deployed on the cluster comprising all the nodes. 
<br>
The current state of any running instance of service is stored in Manager's database, and consists of:
<br>
•	open communication sessions and their load metrics; 
<br>
•	observable (healthy, performance and security) metrics, logs and traces.
<br>
The key element of SSMMP is the concept of communication session understood jointly as establishing a connection and then starting a protocol session on this connection.
The process of establishing and closing such sessions is controlled by the Manager through its agents. 
<br>
Current states of the running service instances are the basis for decisions made by Manager:  
•	execution and shutdown of service instances, 
<br>
•	load balancing by multiple instance executions of stateless services, and closing some of them, 
<br>
•	establishing or closing communication sessions, 
<br>
•	and reconfiguration of running instances; this includes moving some instances to other nodes. 
<br>
These decisions (mutually interrelated) are forwarded to appropriate agents as tasks to be accomplished.  

# The abstractions
SSMMP is based on the two abstraction: 
<br>
•	abstract definition of service of CNApp,
<br>
•	and abstract graph of CNApp. 
<br>
CNApp is a network application where microservices communicate with each other by exchanging messages (following CNApp's business logic) using dedicated, specific protocols implemented on top of the network protocol stack.  Usually, it is TCP/UDP/IP.
Each of these protocols is based on the client-server model of communication. 
<br>
The server (as part of a running microservice on a host with a network address) is listening on a fixed port for a client that is a part of another microservice, usually, running on a different host. 
<br>
Since a client initiates a communication session with the server, this client must know the address and port number of the server. 
<br>
A single microservice can implement and participate in many different protocols, acting as a client and/or as a server.
Thus, a microservice can be roughly defined as a collection of servers and clients of the protocols it participates in, and its own internal functionality (business logic). 
<br>
Communication protocols (at application layer) are defined as more or less formal specifications independently of their implementation. 
<br>
Protocol is denoted (P,S) as two closely related parties to the conversation: the server S and the client P which are to be implemented on two microservices. 
<br>
After implementation, they are integral parts (modules) of microservices that communicate using this protocol.
<br>
Abstract inputs of a microservice can be defined as a collection of the servers (of the protocols) it implements: 
 IN := (S1, S2, ... , Sk)
<br>
Abstract outputs of a microservice are defined as a collection of the clients (of the protocols) it implements:  OUT := (P1, P2, ... , Pn) 
<br>
Components of abstract input are called abstract sockets, whereas components of abstract output are called abstract plugs. 
An abstract plug (of one microservice) can be associated to an abstract socket (of another microservice) if they are two complementary parties of the same communication protocol. 

### Simple example of an abstract CNApp
<p align="center">
  <img src="/ssmmp-abstraction.pptx.svg" />
</p>
The above directed acyclic graph represents a workflow of microservices that comprise a simple CNApp. 
The edges of the graph are of the form (abstract plug -> abstract socket). 
They are directed, which means that a client (of a protocol) can initiate a communication session with a server of the same protocol.
<br>
Microservice  is defined as 
A := (IN, F, OUT)
<br>
where IN is the abstract inputs of the microservice, OUT is the abstract outputs, and F  denotes the business logic of the microservice.
<br>
Incoming messages, via abstract sockets of IN or/and via abstract plugs of OUT, invoke (as events) functions that comprise the internal functionality  F  of the microservice. 
This results in outgoing messages sent via IN or/and OUT. 
<br>
<br>
There are 3 kinds of microservices:
<br>
•	API Gateways - entry points of CNApp for users. Usually, IN of API Gateway has only one element. Its functionality comprises in forwarding users requests to appropriate microservices. API Gateway is stateless.
<br>
•	Regular microservices - their IN and OUT are not empty. They are also stateless. Persistent data (states) of these microservices are  stored in backend storage services (BaaS).
<br>
•	Backend storage services (BaaS) where all data and files of CNApp are stored. Their OUT is empty.
<br>
All of them are also called services of CNApp. 
The above Figure illustrates a CNApp composed of one API Gateway, five stateless regular microservices, and two backend storage services (BaaS). 
<br>
The edges denote abstract connections and can also be seen as abstract compositions of services within a workflow. 
<br>
Abstract graph of CNApp is defined as a directed labeled multi-graph 
<br>
G := (V, E)
<br>
V and E denote respectively Vertices and Edges. 
<br>
Vertices V is a collection of names of services of CNApp, i.e. elements denoted in the Figure as: A (the API Gateway); regular microservices: service B, service-1, service-2, service-3, and service-4; and BaaS services: BaaS-1 and BaaS-2. 
<br>
Edges E is a collection of labeled edges of the graph. Each edge is of the form:  
<br>
(C, (P,S), D)
<br>
where C and D belong to V and (P,S) denotes a protocol. 
That is, P belongs to OUT of C, and S belongs to IN of D. The edges correspond to abstract connections between microservices.
<br>
The direction of an edge in the graph represents the client-server order of establishing a concrete connection. 
<br>
An implementation of abstract connection (C, (P,S), D) in a running CNApp results in a concrete plug (in an instance of service C) corresponding to this abstract plug P. The concrete plug is connected to a concrete socket (corresponding to abstract socket S) of an instance of service D. 
This connection is called a communication session. 
<br>
Initial vertices of the abstract graph correspond to API Gateways (entry points for users), whereas the terminal vertices correspond to backend storage services (BaaS) where all data and files of the CNApp are stored. 
<br>
The vertices representing regular microservices are between the API Gateways and the backend storage services (BaaS).
Scaling through replication and reduction (closing replicas) of a service forces it to be stateless. 
API Gateways and regular microservices are stateless and can be replicated, i.e. multiple instances of such a service can run simultaneously. 
<br>
To run CNApp, instances of its services must first be executed, then abstract connections can be configured and established as real connections, and finally protocol sessions (corresponding to these connections) can be started. 
Some services and/or connections may not be used by some executions of CNApp. Temporary protocol sessions can be started for already established connections (and then closed along with their connections) dynamically at runtime. 
Multiple service instances may be running, and some are shutting down.
This requires dynamic configurations of network addresses and port numbers for plugs and sockets of the instances. 
<br>
The novelty of SSMMP lies in the smart use of these configurations. A similar idea has been used by Netflix at the software level, but has not been fully explored.
<br>
The formal specification of SSMMP is <a href="/SSMMPv2_1_specification.pdf"> here </a> (HYPERLINK). 
The complete description of SSMMP is at <a href="https://arxiv.org/abs/2305.16329"> arXive </a>, and as a slide presentation is <a href="/SSMMP_ang.pdf"> here </a>. 

# Summary of the short intro to SSMMP
SSMMP is simple if we consider its description presented above, and especially the complete formal specification. 
<br>
The concept of abstract connection between services (in the abstract graph of CNApp) and its implementation as communication sessions is crucial. 
The abstract definition of service of CNApp is also important here. 
Separation of these abstract notions from deployment is important. 
<br>
The novelty of SSMMP consists in the dynamic establishment and closing of communication sessions at runtime based on the configurations assigned to sockets and plugs by the Manager.
<br>
Although a similar approach has already been used in <a href="https://netflixtechblog.com/zero-configuration-service-mesh-with-on-demand-cluster-discovery-ac6483b52a51?gi=1a42415024ae"> Netflix </a>   (as dedicated software), it can be fully exploited in Netflix by extending the network protocol stack with SSMMP. 
<br>
Since executing, scaling and reconfiguration of CNApp can be done by SSMMP, it seems reasonable to include SSMMP as an integral part of CNApp. 
Then, also CNApp crash recovery can be performed via SSMMP.
<br>
Graph of CNApp and the states of its running instances are stored by Manager in its KB and DB. 
Failures of agents and service instances can be handled if Manager is running properly. 
Replications of cluster nodes, agents and their service repositories are sufficient means for recoveries from such failures. 
<br>
The central Manager is the weakest point here; its failure results in an irreversible failure of the running CNApp. However, if the Manager's current state is kept securely by a supervising manager, the Manager process can be recovered from that state. 
The supervising manager can also serve as a distributed control plane where there are several Managers, each controlling a portion of the CNApp abstract graph. 
<br>
Netflix uses over 1000 microservices now. Uber now has 4000 or more independent microservices. A distributed control plane seems to be necessary for such huge CNApps. 
<br>
SSMMP was designed to be independent of transport and network protocol stack. TCP/IP is the default stack for communication sessions. Named Data Networking can be seen as an alternative. 
# A simple CNApp for testing the SSMMP protocol

<p align="center">
  <img src="/ssmmp-simpleCNApp.pptx.svg" />
</p>

The basic functionality of the application is as follows.
User interface is Command Line Interface (CLI). 
CLI communicates (via TCP) only with  API Gateway.
The API Gateway forwards user requests to appropriate microservices: registration, login, table (retrieving the recent 10 post of the users), storing user posts in a DB, and File transfer to store (upload)  and retrieve (download) files to/from HD.
Responses from microservices are sent back to API Gateway, and then forwarded to the users.
Microservices cannot communicate directly with users; only via the API Gateway. 
API Gateway and microservices are stateless.  
Requests and responses are objects of class String. 

# Implementations of SSMMP tested on the above test CNApp 

The team:  Mateusz Bielicki and Jakub Luka (students of Computer Science at University of Siedlce) 

So far the implementation clearly show that SSMMP should be viewed (by developers) as an integral part of CNApps. 




