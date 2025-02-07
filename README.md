NSX-T Topology Documentation

1. Overview

This document provides a detailed overview of the NSX-T Data Center topology, focusing on its architecture, workflows, and security features. The NSX-T platform is designed to:

Simplify network virtualization.

Enable multi-tenant support through logical segmentation.

Facilitate secure east-west and north-south traffic flows.

Support scalability and high availability in multi-site deployments.

Provide centralized control and monitoring with granular security policies.

2. Architecture Components

2.1 Management and Control Planes

NSX Manager:

Centralized management for configuration, monitoring, and policy enforcement.

Deployed at both Site 1 and Site 2 to ensure high availability.

Enforces:

NSX Manager Rules: Administrative controls for operations.

NSX Policy Rules: Security and traffic controls for tenants and workloads.

NSX Controller Cluster (Control Plane Communication - CCP):

Maintains logical network state (e.g., switches, routers).

Distributes forwarding tables and routes to transport nodes.

Ensures synchronization and high availability.

2.2 Physical Layer

Physical Network:

L2/L3 devices connecting transport nodes and enabling inter-site communication.

Supports Layer 2 extensions (e.g., L2 VPN) for workload mobility.

Transport Nodes:

Hypervisors or edge devices hosting transport endpoints (TEPs) for encapsulating and decapsulating Geneve traffic.

2.3 Data Plane

Transport Zones:

VLAN Transport Zone (VLAN TZ): For workloads requiring direct access to physical networks.

Overlay Transport Zone (Overlay TZ): For virtualized, scalable tenant networks using Geneve encapsulation.

Logical Segments:

Represent virtual L2 networks for tenants (e.g., Segment A, Segment B for Tenant A).

Logical Gateways:

Tier-0 Gateway:

Connects overlay networks to external networks via uplinks.

Manages north-south traffic, NAT, and routing.

Tier-1 Gateway:

Routes traffic between logical segments and aggregates traffic to Tier-0.

Supports distributed routing for efficient east-west traffic.

2.4 Security Mechanisms

Traditional Segmentation:

Uses VLANs and ACLs for isolation.

Rigid and less scalable compared to virtualized approaches.

Microsegmentation:

Enables granular security policies at the vNIC level.

Enforced through Distributed Firewalls (DFW) for dynamic isolation.

Implements a zero-trust model by defaulting to "deny all" and allowing only permitted traffic.

3. Traffic Flows

3.1 East-West Traffic (Within Overlay)

Intra-Segment Traffic:

Example: VM1 communicates with VM2 within Segment A.

Traffic stays local to the transport node and is switched at Layer 2.

Inter-Segment Traffic:

Example: VM1 (Segment A) communicates with VM3 (Segment B).

Workflow:

Traffic from VM1 is encapsulated using Geneve by the source TEP.

The packet traverses the overlay transport zone.

At the destination TEP, the Geneve header is removed, and traffic is delivered to VM3.

3.2 North-South Traffic (External Communication)

Example: VM4 in Tenant A communicates with an external server.

Traffic flow:

VM4 sends traffic to the Tier-1 Gateway.

The Tier-1 Gateway forwards traffic to the Tier-0 Gateway.

The Tier-0 Gateway performs SNAT or routing and sends the packet to the physical network.

3.3 Inter-Site Traffic

Example: VM in Site 1 communicates with a VM in Site 2.

Workflow:

Source TEP encapsulates traffic with Geneve.

Packet traverses the physical network connecting the sites.

Destination TEP decapsulates the packet and forwards it to the target VM.

3.4 VPN Traffic

L2 VPN Connectivity:

Extends Layer 2 segments across sites for seamless workload mobility.

Example: Workload in Segment A (Site 1) communicates with Segment B (Site 2).

4. Operational Workflows

4.1 Deployment

Install NSX Manager instances at both sites.

Set up the NSX Controller Cluster for centralized control.

Configure transport zones and transport nodes.

4.2 Tenant Onboarding

Create logical segments (e.g., Segment A, Segment B).

Attach tenant workloads (VMs) to the appropriate segments.

Configure Tier-1 Gateways for routing between segments and to Tier-0 Gateways.

4.3 Policy Configuration

Define security groups and microsegmentation rules for tenants.

Apply distributed firewall (DFW) policies to enforce isolation and traffic control.

Use NSX Manager to monitor traffic and ensure compliance with policies.

4.4 Monitoring and Maintenance

Use NSX Manager for real-time monitoring of traffic flows, segment health, and security logs.

Regularly update NSX components and back up configurations.

Scale by adding transport nodes or expanding transport zones as needed.

5. Summary of SNAT and DNAT

SNAT (Source NAT):

Used for outbound traffic from workloads to external networks.

Translates the source IP of a VM (e.g., 172.16.10.4) to the Tier-0 Gateway’s uplink IP (e.g., 203.0.113.1).

DNAT (Destination NAT):

Used for inbound traffic to workloads from external networks.

Translates a public IP (e.g., 203.0.113.5) to the private IP of a VM (e.g., 172.16.10.4).

6. Role of CCP and LCP

6.1 CCP (Central Control Plane)

Managed by the NSX Controller Cluster.

Responsibilities:

Computes and distributes logical routing and switching state to transport nodes.

Ensures consistent configurations across all transport nodes.

Example: CCP updates forwarding tables for a new logical segment and sends them to the transport nodes.

6.2 LCP (Local Control Plane)

Resides on each transport node.

Responsibilities:

Receives and applies instructions from the CCP.

Ensures local data plane functionality, even if the CCP is unavailable.

Example: The LCP processes forwarding and DFW rules for east-west traffic within a transport node.

7. Conclusion

The NSX-T Data Center topology delivers scalable, secure, and flexible network virtualization. Key highlights include:

Granular microsegmentation for security.

Centralized management and distributed control via CCP and LCP.

Seamless traffic flows, including NAT for north-south traffic and Geneve encapsulation for east-west traffic.

Multi-site support for high availability and workload mobility.
