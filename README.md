Default Routing Lab: Branch Office Configuration with Multiple ISP Connections
Overview

This repository contains the configuration for setting up Default Routing in a branch office network with multiple ISP connections. The lab demonstrates how to configure a router to use multiple default routes with different metrics to ensure reliable Internet connectivity, and how failover is handled if one ISP connection goes down.
Key Concepts:

    Default Route: A special route used when a router doesn’t have a specific route to a destination. It is used to route traffic to the default gateway.

    Metric: A value used to determine the preferred path when multiple routes to a destination exist. The lower the metric, the more preferred the route.

    Failover: The process by which traffic is rerouted to a backup path if the primary path (ISP) fails.

Network Topology
Devices and IP Addressing:

    Router 1 (Main Router):

        Interface f0/0: 192.168.12.1/24 (connected to ISP-1)

        Interface f0/1: 192.168.13.1/24 (connected to ISP-2)

        Interface e4/0: 192.168.1.1/24 (connected to branch office network)

    Router 2 (ISP-1 Gateway):

        Interface f0/0: 192.168.12.2/24

        Interface f0/1: 192.168.24.2/24

    Router 3 (ISP-2 Gateway):

        Interface f0/0: 192.168.34.3/24

        Interface f0/1: 192.168.13.3/24

    Router 4 (Google Router):

        Interface f0/0: 192.168.34.4/24

        Interface f0/1: 192.168.24.4/24

        Loopback Interface: 8.8.8.8/32 (Google server IP)

Configuration
Router 1 (Main Router)

Router 1 is the central router connecting the branch office to both ISPs. Two default routes are configured here, one for ISP-1 and one for ISP-2, with different metrics.

conf t
ip route 0.0.0.0 0.0.0.0 192.168.12.2 10    # Default route to ISP-1 (metric 10)
ip route 0.0.0.0 0.0.0.0 192.168.13.3 50   # Default route to ISP-2 (metric 50)

Router 2 (ISP-1 Gateway)

conf t
ip route 8.8.8.8 255.255.255.255 192.168.24.4   # Route to Google server
ip route 0.0.0.0 0.0.0.0 192.168.12.1           # Default route towards Main Router

Router 3 (ISP-2 Gateway)

conf t
ip route 0.0.0.0 0.0.0.0 192.168.13.1           # Default route towards Main Router
ip route 8.8.8.8 255.255.255.255 192.168.34.4   # Route to Google server

Router 4 (Google Router)

conf t
ip route 192.168.12.0 255.255.255.0 192.168.24.2  # Route to ISP-1 network
ip route 192.168.1.0 255.255.255.0 192.168.24.2   # Route to branch network
ip route 192.168.13.0 255.255.255.0 192.168.34.3  # Route to ISP-2 network
ip route 192.168.1.0 255.255.255.0 192.168.34.3   # Route to branch network via ISP-2

Testing and Verification
Test 1: Traceroute to Google DNS (8.8.8.8)

After the configuration, we verified the setup by initiating a traceroute from PC1 to the Google DNS server (8.8.8.8). The path should go through ISP-1 as it has the lowest metric.
PC1 Configuration:

    IP Address: 192.168.1.5/24

    Default Gateway: 192.168.1.1

Commands:

show ip
traceroute 8.8.8.8
ping 8.8.8.8

Test 2: Failover to ISP-2

Next, we simulated an ISP failure by shutting down the f0/0 interface on Router 1. The routing table should now prefer the ISP-2 link.

interface f0/0
shutdown

We then confirmed that traffic was rerouted through ISP-2 by performing another traceroute to 8.8.8.8.
Useful Commands for Troubleshooting:

    show ip route: Display the routing table.

    show run | s ip route: Check for static routes.

    show ip int br: Verify interface statuses.

    ping and traceroute: Test connectivity and routing paths.

Conclusion

This lab demonstrates how to configure Default Routing in a branch office network with multiple ISPs. We configured two default routes on the main router to handle traffic from the branch office to external destinations. Additionally, we tested failover functionality to ensure that if one ISP connection went down, the other would take over seamlessly.

This configuration is critical for ensuring high availability and continuous internet access in enterprise networks that rely on multiple ISP connections.
