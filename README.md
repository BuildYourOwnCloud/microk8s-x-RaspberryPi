# Microk8s x Raspberry Pi 4 Cluster Build Guide 

This document includes a full hardware and software breakdown of the Raspberry Pi Kubernetes clusters present at the Canonical booth during the Kubecon Valencia conference. The goal of this document is to provide enough information to allow anyone to assemble their own cluster to the same specification. 

![Multiple Microk8s Raspberry Pi Clusters](/images/1-MultipleFinishedClusters.jpg)

These clusters are useful for development and test purposes but could also be used in production for smaller workloads or as an edge applicance.  

## Hardware Configuration

As of 2022 the world-wide supply chain still has many issues and some items found in this list may be unobtainable. Therefore, we have tried to suggest adequate alternatives where possible. The logic for choosing each component is also explained. The total build price as of 2022 is around $500-650 USD. 

### Cluster Parts List

The following parts were used as part of the cluster: 

 - 4 x Raspberry Pi Model 4B 8GB (Link): 

Using models with less memory is possible, but less ram means that potentially less containers can run on-top of the nodes. Raspberry Pi Model 3 can also be used. Remember the OS and Kubernetes itself will utilise some of the memory footprint on the system so keep this in mind during capacity planning. As of this writing the global supply of RPI4 are very limited due to supply chain issues and high demand. Therefore, we have highlighted some possible alternative single-board computers in the optional parts section. 

 - 4 x Raspberry Pi POE HAT v2.0 (Link): 

POE is actually optional, but makes the cluster design much cleaner. If your Router/Switch does not support POE, then you will need to buy power supplies for each machine in the cluster. Its important to remember that a good power supply is required as usually a phone charger will not be enough and will give low-power errors. 

 - 4 x Raspberry Pi SSD Adapter (Link): 

The SSD adapters are used to connect the RPI through the USB3 port. This is essentially a SATA to USB3 cable. Other adapters can be used but if the cable is very long it can cause the cluster to be very messy. The Canonical clusters were built using official adapters from the Pi Hut. 

 - 4 x SD Card for the Operating System (Link): 

The SD cards used for the Kubecon clusters varied from 128GB to 256GB depending on the availability from suppliers. The SD card size can be pretty smaller, 32GB should work, but we would recommend 64GB-256GB if possible. It is important to remember that Microk8s or Kubernetes in general usually runs a database such as ETCD or dqlite which can be very harsh to the SD card. We utilise the SSD to mitigate this. 

 - 4 x 2.5" SSD for Microk8s and Container volumes (Link): 

The SSD(s) used as part of the cluster build were 256GB, 2.5" drives. Smaller drives can be used, 32GB as a minimum but we tried to pick respected, well known brands like Kingston. It may be possible to use small 2.5" magnetic drives, but they will be noiser and may require more power draw. 

 - 1 x Raspberry Pi Cluster Case (Link): 

The cluster case used at Kubecon is the UTRONICS Raspberry Pi Cluster Complete Enclosure (Upgraded). This case was chosen for multiple reasons: its well built, sturdy and has enough room for all of the machines with the extra SSDs, POE hats and the switch/router at the bottom. The machines can also be easily removed as the enclosure provides rack style machine mounting. The case also includes two optional 80mm fans for improved cooling. The case is available on Amazon (Link) or at suppliers on the UK (Link) or US (Link). There are many alternative cases available, such as the Pico Cluster (Link), the c4labs case (Link) or the Cloudlet case (Link). 

 - 1 x 5 Port POE Switch or Router (Link): 

With the POE hats, a POE Switch or Router can provide Power to the machines as well as network connectivity which greatly simplifies the number of cables required in the cluster. During Kubecon the switch utilised was the TP-Link 5-Port TL-SG105PE switch which is actually a managed switch and includes a full web-gui for switch management. This can be used to change things like the switch hostname, as well as traffic priority (QoS) and power management of the POE ports for example. Other switches will also work, for example the NETGEAR GS305EPP is another good option which has POE and is managed. These options provide 65W and 100W output, but a 40W switch should also suffice. 

The main downside of using a switch is that it requires a router infront and static DHCP leases will need to be given to the cluster machines OR static IP(s). The main reason for this is the clustering feature of microk8s relies on the ability for machines in the cluster to reliablely talk to each other. 

One way around this problem is obviously to use a router instead of switch. There are two recommended models: The mikrotik hEX PoE (RB960PGS) if you can find one, or the Ubiquiti ER-X-SFP EdgeRouter. These will provide even more additional features to the cluster but the price is generally more expensive and these are hard devices to find right now. 

 - 5 x Cat 5/6/7 Cable (Link): 

As we have 4 machines in the cluster, each one of them requires a network connection to the switch. We do not utilise WiFi in this solution. For Kubecon 2022 we purchased a back of 5 cables in a set, all pre-made. The minimum size we could find was 0.5m but 0.25M or smaller is much better. If you're able to crimp/produce your own cables then this will save cost and provide a more optimal cable configuration. Finally, you will need a longer cable to connect your cluster back to your home router. 

 - Sissors, Philips Screwdrivers (assorted), SD Card Reader, Patience. 

Finally you will need some tools to assemble the case: some sissors to open packaging, different types of Philips screwdrivers and some patience. Flashing the SD card for every machine and configuring each machine individually can take some time. The SD Card reader will need to be able to read/write MicroSD cards and you will need another machine to write the Ubuntu image to the SD cards. 

### Optional Parts

 - 2 x USB to 2 PIN Fan Connector (Link): 

The UTRONICS case has two optional 80MM case fans which come with it. These fans came with 2-pin connectors and there is seemingly no where to easily connect them as the GPIO pins are attached to POE hats. One simple way to solve this problem is to use a USB to 2-Pin fan connector cable or create your own. These can be found on AliExpress, eBay and Amazon. 

 - 2 x Noctua Silent 80mm Fan (Link): 

Two fans are included with the UTRONICS case but they are pretty basic. One optional recommendation is to replace these with high quality Noctua silent fans or high performance fans such as the Noctua NF-A9x14. 

 - Rock Pi X (Link): 

For those who want multi-architecture support in their cluster, the Rock Pi X is a great addition to the cluster and it includes the required pins in order to utilise the POE hat. The Rock Pi X includes an Intel CPU, a Cherry Trail Quad Core Z8350 64Bit CPU and therefore, can be used to bring x86_64 support to the cluster if required. This is quite useful if you plan to use the cluster to perform software builds. A different Ubuntu image is required to boot this machine. 

![RockPiX Intel x86_64 Machine](/images/1-RockPiX.jpeg)

- Rock Pi 4C (Link)

The Rock Pi 4C is functionally similar to the Raspberry Pi 4 with a similar form factor and breakout pins. It also supports the POE Hat. It has a few differences, for example it can only drive a single 4K screen and is only supplied with a maximum 4GB of Memory as of this writing, but does have some benefits. The main two beneficial differences for this cluster are storage options, at the Rock Pi 4 supports both M.2 SSD and eMMC connections which are both better than the SD card slot on the RPI4. A different Ubuntu image is required to boot this machine. 
 - Heatsinks, cable ties, double sided tape: 

It is recommended to utilise some cable ties to clean up the network cables, fan cables and potentially the SSD cables depending on their length. It is also possible to buy OEM heatsinks which can be used on the various chipsets on the board. The cable ties can be purchased from a variety of websites such as Amazon or AliExpress. During the event we utilised double sized tape to stick the switch to the bottom of the case, but it may be possible to screw it to the bottom of the case if you have the correct adapter. 

### Assembly Guide

The assembly guide for the UTRONICS case is included in this repo or can be found using the following link. It is pretty straight forward to follow with two main caveats: the screws can be easily confused and the fan placement may be misleading in the diagrams. The image below provides a list of all the screws and the rubber feet provided with the case: 

![UTRONICS Case Screws List](/images/ClusterCaseScrews.jpeg)

The smaller black screws denoted (1) are used to assmeble the majority of the case. The slightly larger screws denoted (2) are used to mount the SSD to the removable RPI trays. The larger screws denoted (3) are used to mount the fans to the chassis and these are regular PC fan screws. The screws marked (4) are used to mount the Raspberry Pi machines to the removable machine trays. The screws marked (5) are used to mount the removable machines to the cluster case. Finally the rubber feet marked (6) are attached to the bottom of the case when it is built. 

Once the case has been assembled, mount the SSD(s) and the raspberry pi(s) to each of the removable rack mounts. The switch should then be mounted in the bottom of the case. One of the main problems of the design is that the POE Hats need some clearance from the Raspberry Pi but the rack mounts do not provide this properly. It is possible to better mount this using some very long M2 screws. See this picture to see how it was mounted during the event. 

The next stage is to flash the SD Cards with Ubuntu and to boot the machines one by one using the POE. 

## Software Configuration

This section details the software configuration for the entire cluster, including the OS installation, Microk8s installation and additional software configuration. 

### Flashing the SD cards with Ubuntu

WIP 

## Microk8s 22.04 Demo Features

As part of the 22.04 Microk8s release, the following two new features were demonstrated during the event: 

- Private Addons Repo: this allows third-party applications to be easily bundled with Microk8s (https://ubuntu.com/blog/the-microk8s-addons-framework-now-open-to-everyone) as a turn-key solution. 

- OpenEBS/Mayastor Clustered Storage: Microk8s added a new container storage solution for Microk8s which can be used for PV(s). With the correct setup, this will utilise the SSD(s) and will provide a resilient storage solution for workloads. The full guide can be found here: https://microk8s.io/docs/addon-mayastor. 

## Summary, Getting Help and Future Improvements

Hopefully this guide has provided enough information to allow you to build your own Microk8s cluster using Raspberry Pi. For questions on this setup, please reach out to Calvin Hartwell (calvin.hartwell@canonical.com) or reach out on the Microk8s slack channel on the official Kubernetes slack (https://microk8s.io/docs/get-in-touch). Microk8s bugs can also be raised on Github here: https://github.com/ubuntu/microk8s/issues. 

The next feature which could be added to the cluster is the use of MAAS with customer power drivers. This would allow the cluster to be network booted and dynamically provisioned or provisioned on-demand. Another nice addition may be some Google Coral devices for added AI/ML capability. 
