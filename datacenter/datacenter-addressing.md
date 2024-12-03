---
title: Addressing
parent: Datacenters
nav_order: 4
layout: page-with-toc
---

# Datacenter Addressing

## Why are Datacenters Different?

In the previous section, we saw that we can modify distance-vector and link-state routing protocols to compute all paths through the datacenter network.

However, these protocols might scale poorly in datacenters. In distance-vector protocols, we have to make an announcement for every destination, which means that 100,000+ destinations have to be advertised. In link-state protocols, we have to flood advertisements along every link, which scales poorly in Clos networks with a huge number of links. Also, recall that datacenter topologies often use cheap commodity switches, which have limited memory and CPU resources (e.g. the forwarding table can't be too large).

In general-purpose networks, we solved these scaling problems by introducing hierarchical IP addressing. Higher-level organizations (e.g. country-level) could allocate ranges of addresses to smaller organizations (e.g. universities). Datacenters don't have geographic and organizational hierarchies that we can use to organize addresses.

However, in datacenters, we can exploit the fact that the operator controls the physical topology of the network, and assign addresses to servers based on where they're located in the building. We can also exploit the fact that the topology has some regular structure (e.g. we're probably organizing servers in rows, instead of randomly stuffing them in the building).


## Topology-Aware Addressing

<img width="900px" src="/assets/datacenter/6-42-dc-addressing.png">

In this particular topology, the racks are physically organized into separate pods in the building. One natural approach would be to allocate a range of addresses to each pod. Then, each pod can allocate sub-ranges to each rack in the pod. Finally, each rack can allocate an individual IP address to each server.

The operator knows how many servers are in each rack, and how many racks are in each pod, so we can use that information to allocate ranges of the appropriate size. For example, a rack could receive a /24 range, which gives that rack 256 addresses for its servers.

This allocation approach lets aggregate routes and store fewer entries in our forwarding table. For example, consider one of the spine routers at the top of the diagram. This router doesn't need to remember a path for every single server. Instead, the forwarding table only needs four entries, one for each pod. When a packet arrives, the router checks the first 16 bits to forward the packet to the appropriate pod.

Route aggregation also results in more stability. If a host is added or removed inside a specific rack, the spine router doesn't need to know. As long as we maintain the same addressing scheme, the existing forwarding table is still correct without any changes. As a result, routing updates usually occur when links and switches fail, but not when hosts fail.

Assigning addresses based on datacenter topology is good for scaling, but there are some limitations. In particular, if we move a server to a different location, we'd have to change its address.