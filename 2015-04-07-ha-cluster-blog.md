##Introduction

One of the most anticipated new features in Stardog 3.0 is the support for High Availability (HA) Clustering. Any service that runs as a single instance on a single machine is at risk of failure. While the service is down, any applications that depend upon it are likely to be down as well. Clustered versions of critical services resolve the issue by running multiple copies of the service. If one instance is down, another one will pick up the slack. This is a crucial property for the reliability of an org's core infrastructure.

Addressing this scenario is easy in principle, but difficult in practice. Coordinating consistent data states across distributed instances of a service is difficult to do efficiently and correctly. The rise of Big Data and NoSQL systems has allowed database administrators to make tradeoffs to favor efficiency of reads vs writes or whether to maintain absolute consistency across a clustered system. The choices they make depend upon the system properties they require. The shape of their data will also constrain which options make sense. Stardog excels at richly interlinked graphs, so that influenced how we designed its clustered architecture.

##Goals

We chose to favor high availability and improved read throughput and reasoning. This approach gives a consistent view of the data across the cluster. The more nodes in the cluster you have available, the less susceptible you are to hardware problems and the more read queries you can run per second. Any node can respond to a query and reasoning results will be the same everywhere.

The need to maintain consistency across the cluster has implications on the speed of writes to the backend. A single server node cannot be responsible for accepting a write from a client. Were it to face a hardware failure, no other node would see the change and the cluster would be in an inconsistent state. To solve this problem, the Stardog cluster uses a two-phase commit protocol to make sure that every node has received a copy of the write.

##Architecture

Building this kind of infrastructure from scratch is hard. So we based our implementation, in part, on the [Apache Zookeeper](<http://zookeeper.apache.org>) project. This is a widely-used and battle-tested distributed coordination framework. It provides many of the features we needed to let the individual nodes communicate robustly. Zookeeper is currently used in production at companies like Rackspace, Yahoo!, eBay and Pinterest to enable a variety of distributed systems and policies.

A Stardog cluster involves a Coordinator to handle writes and Participant instances to respond to read-only queries. To maintain its consistency guarantees, a Stardog HA Cluster requires at least three nodes. While it is possible to run a cluster on a single machine for development or testing purposes, that would ultimately defeat the goal of resisting hardware failure. 

Each Stardog instance is configured to participate in the cluster and runs a proxy for the service that communicates with the Zookeeper infrastructure. (Stardog can also use an existing Zookeeper service.) Reads are spread out across all of the Participant nodes to distribute the load. With more than one server instance available to respond to queries, the cluster can answer more queries than a single instance would be able to in the same timeframe.

##DNS Configuration for the Client
The benefits of a transparently failsafe server environment would be undermined if all of the clients had to keep track of the individual server nodes and talk to them directly. As new instances were started to respond to increased load, we would have to have a way of alerting all of the clients about their presence. As nodes were shutdown, we would have to let the clients know not to rely on them any more. To avoid this extra complexity, we rely on a Domain Name Service (DNS) configuration that allows a single server instance to be associated with multiple instances of the server. As instances come online or go away, the DNS records are kept up to date. The clients only depend upon the stable server name and port configuration and are able to pick an instance from the DNS response records.

With an HA Cluster in place, customers can communicate with Stardog as if it were a single instance while getting the benefit of a clustered environment. They get the ease of use and simplicity they are familiar with and the added confidence of knowing they are protected against failure for business-critical systems. Customers also benefit from being able to scale up their Stardog infrastructure incrementally in response to increased load demands.

Learn more about setting up your own clustered instances of Stardog [here](<http://docs.stardog.com/#_high_availability_cluster>). If you haven't done so already, go ahead and download Stardog [here](<http://stardog.com/#download>).
