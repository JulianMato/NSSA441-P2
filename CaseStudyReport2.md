
# <center>**BGP in the Datacenter**</center> <center> By Julian Mato-Hernandez & Evan Jurdan </center>

# Table of Contents:
- [Introduction](#introduction)
- [Technology Used](#technology-used)           
    - [Clos & Fat Tree](#clos--fat-tree) 
    - [BGP & EBGP](#bgp--ebgp)
    - [Full Mesh & Partial Mesh](#full-mesh--partial-mesh)
    - [ECMP Load Sharing](#ecmp-load-sharing)
- [Experements](#experements)
    - [Topology Description and Benefits](#topology-description-and-benefits)
    - [Link Breakage from Spine to Leaf and from Leaf to ToR](#link-breakage-from-spine-to-leaf-and-from-leaf-to-tor---experement)
    - [Inter & Intra Pod Travel Time](#inter--intra-pod-travel-time---experement)
    - [ECMP Load Balancing Eficiency](#ecmp-load-balancing-eficiency---experemint)
- [Results](#results)
    - [Link Breakage from Spine to Leaf and from Leaf to ToR](#link-breakage-from-spine-to-leaf-and-from-leaf-to-tor---result)
    - [Inter & Intra Pod Travel Time](#inter--intra-pod-travel-time---result)
    - [ECMP Load Balancing Eficiency](#ecmp-load-balancing-eficiency---result)
- [Discussion of results](#discussion-of-results)
- [Critical Summary](#critical-summary)
- [References](#references)
- [Appendix](#appendix)
<br>
<br>
<br>

# Introduction:
**To-Do**
<br>
<br>
<br>

# Technology Used:
## Clos & Fat Tree -
Fat Tree(A subset of Clos topologies) relies heavily on route aggregation. more specifically as you see in figure 1 each level has a "thicker" connection than the last. Clos and therefore Fat Tree traditionally use a 2 or 3 tiered network structure with the "Spine" being at the top followed by "leafs" and in a 3 tiered Clos a final ToR layer. The Spine acts as the main distribution between otherwise isolated pods of systems. The leafs are the aggregation layer, they collect and truncate traffic from the ToR layer to pass it through to the Spine. Leaf layers are also responsible for any IP Sec or other security measures implemented by the system, this is crucial due to the need to keep the Spine as secure as possible. With the specialized nature of each layer relatively, specific hardware considerations have to be made, the Spine needs relatively few connections but needs massive bandwidth to handle the aggregated leaf layer traffic, leaf layers need a large number of ports to accommodate as many hosts or ToR's with a limited number of high-speed links to the Spine, and finally, the ToR layer only needs a large number of low-speed connections. By having all of your traffic aggregate adding new portions to the network is trivial giving Clos tremendous scalability while remaining structured and organized.

![Figure 1](Fat_tree_network.png "Fat Tree")
<br>
<br>

## BGP & EBGP -
BGP "is the routing protocol that glues together the net-works forming the Internet"(Garcia-Martinez and Bagnulo 2432). BGP works on the concept of neighbors, route advertisements, hello timers, and route flooding. To be exact we are using EBGP to simplify our topology, in brief, EBGP advertises and re-advertises routes to all BGP-enabled devices. A neighbor in BGP is simply the next hop or directly adjacent BGP aware device that a BGP device has established trust with, to establish a trust a BGP device must know the device's correct AS or Autonomous System number and the IP address of the connecting link. This is how BGP maintains its isolation from other BGP or malicious networks, without intimate knowledge of the network, not much can be done. Route advertisements are as they sound, when a BGP device wants to share the location of a network directly attached to them they can advertise it to the network thus propagating its location across all BGP routing tables. Hello timers are packets sent from neighbor to neighbor to simply affirm its continued function in the network, normally hello timers are sent at a predefined interval and if ever 3 acknowledgments are missed in a row the network is flooded with a route change. Finally, flooding is the act of propagating information to all systems on the network.
<br>
<br>

## Full Mesh & Partial Mesh -
Full Mesh is a term to refers to the total interconnection of all nodes in a network as shown in figure 2. This allows for maximum points of redundancy in case of a link failure, for this reason, all the pods in a Clos or Fat Tree topology are interconnected. This is best practice in most networks, though it does bring with it some issues namely loops. A large issue with full mesh is the existence of loops in the network leaving the opportunity for packets to get lost in limbo slowing down the network and eventually crashing the machines tunning it. As to avoid this we have loop avoidance protocols but by adding another protocol to your network you are adding latency, and unfortunately the more links the longer convergence could take as the protocol searches for a new route. this can be mitigated by adding link state costs to narrow the decision tree but if done correctly in a 3 tier Clos topology the redundancy already present at the spine a partial mesh can be used while maintaining redundancy. Partial Mesh as it sounds only connects a portion of the systems together leaving a higher chance of failure unless accounted for though it does alleviate network complexity and convergence time at times.

![Figure 2](NetworkTopology-FullyConnected.png "Full Mesh")
<br>
<br>

## ECMP Load Sharing -
**To-Do**
<br>
<br>
<br>

# Experements:

## Topology Description and Benefits -
**To-Do**
<br>
<br>

### *Spine*
| Host Name | Interface 1(E1/1) | Mask | Interface 2(E1/2) | Mask | Interface 3(E1/3) | Mask | AS # |
| :--- | ---: | :--- | ---: | :--- | ---: | :--- | :---: |
| N-1_S-1 | 1.1.1.1 | /30 | 1.1.1.5 | /30 | 1.1.1.9 | /30 | 124 |
| N-1_S-2 | 1.1.1.13 | /30 | 1.1.1.17 | /30 | 1.1.1.21 | /30 | 149 |
| N-1_S-3 | 1.1.1.25 | /30 | 1.1.1.29 | /30 | 1.1.1.33 | /30 | 174 |
| N-1_S-4 | 1.1.1.37 | /30 | 1.1.1.41 | /30 | 1.1.1.45 | /30 | 199 |             
<br>

### *Leaf*
| Host Name | Interface 1(E1/1) | Mask | Interface 2(E1/2) | Mask | Interface 3(E1/3) | Mask | Interface 4(E1/4) | Mask | Interface 5(E1/5) | Mask | AS # |
| :--- | ---: | :--- | ---: | :--- | ---: | :--- | ---: | :--- | ---: | :--- | :---: |
| N-1_P-1_L-1 | 1.1.1.2 | /30 | 1.1.1.26 | /30 | 2.1.1.1 | /30 | 2.1.2.1 | /30 | 2.1.3.1 | /30 | 216 |
| N-1_P-1_L-2 | 1.1.1.14 | /30 | 1.1.1.38 | /30 | 2.2.3.1 | /30 | 2.2.1.1 | /30 | 2.2.2.1 | /30 | 232 |
| N-1_P-2_L-3 | 1.1.1.6 | /30 | 1.1.1.30 | /30 | 2.3.4.1 | /30 | 2.3.6.1 | /30 | 2.3.5.1 | /30 | 248 |
| N-1_P-2_L-4 | 1.1.1.18 | /30 | 1.1.1.42 | /30 | 2.4.6.1 | /30 | 2.4.4.1 | /30 | 2.4.5.1 | /30 | 264 |
| N-1_P-3_L-5 | 1.1.1.10 | /30 | 1.1.1.34 | /30 | 2.5.7.1 | /30 | 2.5.9.1 | /30 | 2.5.8.1 | /30 | 280 |
| N-1_P-3_L-6 | 1.1.1.22 | /30 | 1.1.1.46 | /30 | 2.6.9.1 | /30 | 2.6.7.1 | /30 | 2.6.8.1 | /30 | 296 |             
<br>

### *ToR*
| Host Name | Interface 1(E1/1) | Mask | Interface 2(E1/2) | Mask | Interface 3(E1/3) | Mask | AS # | Advertised Network's |
| :--- | ---: | :--- | ---: | :--- | ---: | :--- | :---: | :--- |
| ToR-1 | 2.1.1.2 | /30 | 2.2.1.2 | /30 | 192.1.1.254 | /24 | 311 | 192.1.1.0/24 |
| ToR-2 | 2.1.2.2 | /30 | 2.2.2.2 | /30 | 192.1.2.254 | /24 | 322 | 192.1.2.0/24 |
| ToR-3 | 2.2.3.2 | /30 | 2.1.3.2 | /30 | 192.1.3.254 | /24 | 333 | 192.1.3.0/24 |
| ToR-4 | 2.3.4.2 | /30 | 2.4.4.2 | /30 | 192.2.4.254 | /24 | 344 | 192.2.4.0/24 |
| ToR-5 | 2.3.5.2 | /30 | 2.4.5.2 | /30 | 192.2.5.254 | /24 | 355 | 192.2.5.0/24 |
| ToR-6 | 2.4.6.2 | /30 | 2.3.6.2 | /30 | 192.2.6.254 | /24 | 366 | 192.2.6.0/24 |
| ToR-7 | 2.5.7.2 | /30 | 2.6.7.2 | /30 | 192.3.7.254 | /24 | 377 | 192.3.7.0/24 |
| ToR-8 | 2.5.8.2 | /30 | 2.6.8.2 | /30 | 192.3.8.254 | /24 | 388 | 192.3.8.0/24 |
| ToR-9 | 2.6.9.2 | /30 | 2.5.9.2 | /30 | 192.3.9.254 | /24 | 399 | 192.3.9.0/24 |             
<br>

![Figure 3](img-top-fing.png "topology")\
<br>
<br>

## Link Breakage from Spine to Leaf and from Leaf to ToR - Experiment -              
One of the main features of a meshed topology and BGP is to be able to re-establish connection after a broken link, so for this experiment, we will be testing that ability. we start by generating large amounts of TCP and UDP traffic from K-Host-1 to K-Host-3 with the help of `nmap -p0-65535 192.2.1.1 -T5` and `nmap -sU -p- 192.2.1.1 -T5`. once the traffic starts to be generated we break a link. To begin with we broke 1.1.1.36/30 5 times followed by breaking link 2.1.1.8/30. K-Host-3 has a Wireshark capture running on it, we log when the traffic stops and when it re-establishes connection and take the difference. Since the hello packet interval is set to 10 seconds and the LSA interval is 32 seconds we should be looking at an average of 31 seconds till convergence.

**To-Re-Work**
<br>
<br>

## Inter & Intra Pod Travel Time - Experiment -          
For this experement, we generated 5 minutes worth of traffic and graphed it by tcp.analysis.ack_rtt on wireshark. We did this once from one pod to the other in other words inter pod traffic, then between two hosts in one pod or Intra pod traffic. This will show if there are any performance losses due to the spine.

**To-Re-Work**
<br>
<br>

## ECMP Load Balancing Eficiency - Experemint -          
**To-Do**
<br>
<br>
<br>

# Results:

## Link Breakage from Spine to Leaf and from Leaf to ToR - Result -              
**To-Do**
<br>            
<br>        

| Location | Time of Disconnect | Time of Re-connect | Convergence Time |
| :--: | :--- | :--- | :---: |
| Convergence 1 Back Bone pod1 → pod2 |  |  |  |
| Convergence 2 Back Bone pod1 → pod2 |  |  |  |
| Convergence 3 Back Bone pod1 → pod2 |  |  |  |
| Convergence 4 Back Bone pod1 → pod2 |  |  |  |
|  |  |  |  |
| Convergence 1 ToR – Leaf pod1 → pod2 |  |  |  |
| Convergence 2 ToR – Leaf pod1 → pod2 |  |  |  |
| Convergence 3 ToR – Leaf pod1 → pod2 |  |  |  |
| Convergence 4 ToR – Leaf pod1 → pod2 |  |  |  |
|  |  |  |  |
| Convergence 1 ToR – Leaf pod1 → pod1 |  |  |  |
| Convergence 2 ToR – Leaf pod1 → pod1 |  |  |  |
| Convergence 3 ToR – Leaf pod1 → pod1 |  |  |  |
| Convergence 4 ToR – Leaf pod1 → pod1 |  |  |  |

<br>
<br>

## Inter & Intra Pod Travel Time - Result -          
<br>
To-Do
<br>
<br>

![Figure 3](5min-capture-intra_pod "topology")\
![Figure 3](5min-capture-inter_pod "topology")\
<br>
<br>

## ECMP Load Balancing Eficiency - Result -          
**To-Do**
<br>
<br>
<br>

# Discussion of Results:           
The results fell in line with what was expected of BGP and Clos. There were some outliers such as one data point in the convergence time experiment that managed to converge in 25 seconds which should not be possible due to the hello timer being 10 seconds, the conclusion was that there must have been a missed hello timer before the link was broken. Another interesting development is the relatively small impact traveling through the spine has on connection speed, it appears nearly negligible.      
<br>
With this data, it is easy to see the stability of this network even under a 98000 packet per minute load. Though some of the network degradations are probably attributable to the form of virtualization used for this topology (GNS3 with qemu). 

**To-Re-Work**

<br>
<br>
<br>

# Critical Summary:            
There are many advantages to BGP but unfortunately, it has one glaring flaw that was out of the scope of this paper, and that would be its security. The risk of AS leaks and the ease of trace routing a network makes BGP very risky. In a perfect world, the AS numbers would be unacceptable but unfortunately, it is not a question of if but of when.             
<br>
On top of the security issues, there is a lot of administration and organization needed to deploy BGP on this topology, obviously, in the context of a data center this is trivial but in a small to medium network, this is simply unreasonable. However when feasible this topology and protocol do compliment each other very well. 

**To-Re-Work**

<br>
<br>
<br>

# References:          
Dutt, Dinesh. BGP in the Data Center. 1st ed., Van Duuren Media, 2017.          
Zhang, Randy, and Micah Bartell. BGP Design and Implementation. Amsterdam University Press, 2004.           
C. Camarero, C. Martínez and R. Beivide, "Random Folded Clos Topologies for Datacenter Networks," 2017 IEEE International Symposium on High Performance Computer Architecture (HPCA), 2017, pp. 193-204, doi: 10.1109/HPCA.2017.26.                  
A. García-Martínez and M. Bagnulo, "Measuring BGP Route Propagation Times," in IEEE Communications Letters, vol. 23, no. 12, pp. 2432-2436, Dec. 2019, doi: 10.1109/LCOMM.2019.2945964.                      
H. K. Dhaliwal and C. -H. Lung, "Load Balancing Using ECMP in Multi-Stage Clos Topology in a Datacenter," 2018 IEEE Conference on Dependable and Secure Computing (DSC), 2018, pp. 1-7, doi: 10.1109/DESEC.2018.8625147.                    
https://www.cisco.com/c/en/us/td/docs/routers/crs/software/crs_r5-3/routing/command/reference/b-routing-cr53xcrs/b-routing-cr53xcrs_chapter_00.html#wp1982813266                
https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/7-x/unicast/configuration/guide/b_Cisco_Nexus_9000_Series_NX-OS_Unicast_Routing_Configuration_Guide_7x/b_Cisco_Nexus_9000_Series_NX-OS_Unicast_Routing_Configuration_Guide_7x_chapter_01011.html#task_98AC2C21385948F5B3726924CDF790E1                   
https://www.cisco.com/c/en/us/td/docs/switches/datacenter/nexus9000/sw/93x/interfaces/configuration/guide/b-cisco-nexus-9000-nx-os-interfaces-configuration-guide-93x/b-cisco-nexus-9000-nx-os-interfaces-configuration-guide-93x_chapter_010000.html#id_78241              



Figure 1 - By Jafet at English Wikipedia - Transferred from en.wikipedia to Commons., Public Domain, https://commons.wikimedia.org/w/index.php?curid=37953156          
Figure 2 - Public Domain, https://commons.wikimedia.org/w/index.php?curid=991408                    
Figure 3 - By Original: FoobazSVG: Rehua - This file was derived from: NetworkTopology-Mesh.png, Public Domain, https://commons.wikimedia.org/w/index.php?curid=31012050

<br>
<br>
<br>

# Appendix:            
<dl>
   <dt>LSA</dt>
       <dd>Link State Advertisement</dd><br>
   <dt>Clos</dt>
       <dd>A type of datacenter topology</dd><br>
   <dt>Fat Tree</dt>
       <dd>A type of Clos topology</dd><br>
   <dt>BGP</dt>
       <dd>Border Gateway Protocol</dd><br>
   <dt>EBGP</dt>
       <dd>External Border Gateway Protocol</dd><br>
   <dt>Full Mesh</dt>
       <dd>When all nodes in a network are interconnected</dd><br>
   <dt>Partial mesh</dt>
       <dd>When only some nodes in a network are interconnected/dd>
   <dt>topology</dt>
       <dd>how to refer to the physical structure of a network</dd><br>
   <dt>spine</dt>
       <dd>The top layer of a Clos topology</dd><br>
   <dt>leaf</dt>
       <dd>The middle or "aggregation" layer of a Clos topology</dd><br>
   <dt>ToR</dt>
       <dd>Top of Rack in a Clos topology</dd><br>
   <dt>pod</dt>
       <dd>Point of delivery</dd><br>
   <dt>link</dt>
       <dd>Connection between two machines</dd><br>
   <dt>Routing protocol</dt>
       <dd>The software that fairies data over a network</dd><br>
   <dt>Inter</dt>
       <dd>Between two things</dd><br>
   <dt>Intra</dt>
       <dd>Within itself</dd><br>
   <dt>route aggregation</dt>
       <dd>The act of making many small individual routes converge in to one route</dd><br>
   <dt>truncate</dt>
       <dd>limit the number of</dd><br>
   <dt>IP Sec</dt>
       <dd>Internet protocol security</dd><br>
   <dt>bandwidth </dt>
       <dd>measure of the data transfer rate or capacity of a given network</dd><br>
   <dt>aggregated</dt>
       <dd>To make one</dd><br>
   <dt>hosts</dt>
       <dd>Computers on the network</dd><br>
   <dt>neighbors</dt>
       <dd>The router or layer 3 switch directly adjacent</dd><br>
   <dt>route advertisements </dt>
       <dd>the way BGP states what networks it is aware of</dd><br>
   <dt>hello timer</dt>
       <dd>Protocol to ensure all nodes in a BGP network are functioning</dd><br>
   <dt>route flooding</dt>
       <dd>The act of receiving saving and repeating a message</dd><br>
   <dt>AS</dt>
       <dd>Autonomous System</dd><br>
   <dt>Autonomous System Number</dt>
       <dd>A unique identifier for each router or layer 3 switch</dd><br>
   <dt>propagating</dt>
       <dd>replicateing</dd><br>
   <dt>route change</dt>
       <dd>when the flow of traffic changes</dd><br>
   <dt>redundancy</dt>
       <dd>Secondary or contingent systems</dd><br>
   <dt>convergence time</dt>
       <dd>The time from a lost link to re-establishing connection</dd><br>
   <dt>octet </dt>
       <dd>one 8 bit quarter of an IPv4 address</dd><br>
   <dt>/30</dt>
       <dd>a subnet with only 4 addresses and 2 usable ones</dd><br>
   <dt>hostname</dt>
       <dd>the name of the machine</dd><br>
   <dt>downlinks</dt>
       <dd>Network connections who are no longer passing traffic</dd><br>
   <dt>hot standby</dt>
       <dd>A machine which is idling for the purpose of taking over if there the primary fails</dd><br>
   <dt>wireshark</dt>
       <dd>A packet capturing software</dd><br>
   <dt>capture</dt>
       <dd>log incoming and outgoing traffic</dd><br>
   <dt>traffic</dt>
       <dd>Data moving in a network</dd><br>
   <dt>GNS3</dt>
       <dd>Network emulation software</dd><br>
   <dt>qemu</dt>
       <dd>virtualization software</dd><br>
<dl>

