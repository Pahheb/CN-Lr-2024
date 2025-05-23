# Routing Lab

By Luis Daniel Casais Mezquida & Javier Moreno Yébenes  
Adapted by Konstantin Rannev  
Computer Networks 22/23 -> 24/25  
Bachelor's Degree in Computer Science and Engineering, grp. 89  
Universidad Carlos III de Madrid


Network configuration using Linksys WRT54GS routers.


## Part 0: Setup

1. Download and run the [Virtual Machine](http://www.it.uc3m.es/fvalera/ro/labuc3m.html)
2. Update the simulator **twice**
```
lightning update && lightning update
```


## Part I: Simple network interconnection

### Start the VM and load the scenario.
```
student@uc3m:~$ lightning start RSC/S16_escenario_1
```

Desired configuration:  

![scenario2](img/scenario1.png)

**A = The last 2 digits of your NIA**  
**B = A + 1**

| Network |       Segment       |
|---------|:-------------------:|
|   `A`   | `172.16.A.0/25`     |
|   `B`   | `172.16.B.0/25`     |
| `RA-RB` | `172.16.0.0/30`     |

It is strongly recommended that you make the routers have the first **non-reserved** IP address within the subnet range

| Device | Interface |          IP         |
|--------|-----------|:-------------------:|
|  `RA`  |  `eth0.0` | `172.16.A.1/25`     |
|        |  `eth0.1` | `172.16.0.1/30`     |
|  `RB`  |  `eth0.0` | `172.16.B.1/25`     |
|        |  `eth0.1` | `172.16.0.2/30`     |
|  `PCA` |   `eth1`  | `172.16.A.2/25`     |
|  `PCB` |   `eth1`  | `172.16.B.2/25`     | 


### Remove the default IP addresses assigned to the `eth0.0` through `eth0.4` and `wlan0` interfaces in the routers
```
RA# show interface eth0.0
```

```
[...]
inet=192.168.0.1/24
[...]
```

```
RA# configure terminal  # enter config mode
RA(config)# interface eth0.0  # select interface
RA(config-if)# no ip address 192.168.0.1/24  # pre-configured IP (one received with show interface)
RA(config-if)# exit
```

Do the same for `eth0.1`, `eth0.2`, `eth0.3`, `eth0.4`, `wlan0`, and for router `RB`:
```
RA(config)# exit
RA# show interface eth0.0  # to check if it was done correctly, inet shouldn't show
```


### Assign an IP address to the Ethernet interface `eth1` of computer `PCA` that is connected to Network `A`
First we see the asigned IPs for `PCA` and `PCB`:
```
student@PCA:~$ ip a
```

```
[...]
3: eth1@[...]
[...]
inet=192.100.100.101/24  # default PCA IP
[...]
```

Remove the default IP:
```
student@PCA:~$ sudo ip addr del 192.100.100.101/24 dev eth1
```

Do the same for PCB.


### Assign an IP address to the interface of `RA` that is connected to the Network `A`
Add IP to `PCA`:
```
student@PCA:~$ sudo ip addr add 172.16.A.2/25 dev eth1
```

Configure `RA` interface w/ `PCA`:
```
RA# configure terminal
RA(config)# interface eth0.0
RA(config-ip)# ip address 172.16.A.1/25
RA(config-ip)# exit 
RA(config)# exit
```

Let's verify that the router and the host can reach each other using the ping command.

Check that you can ping from `PCA` to `RA`:
```
student@PCA:~$ ping 172.16.A.1
```


### Connect `RA` and `RB` through their Ethernet interfaces
Assign IP addresses to the interfaces that connect both routers:
```
RA# configure terminal
RA(config)# interface eth0.1
RA(config-ip)# ip address 172.16.0.1/30
RA(config-ip)# exit 
RA(config)# exit
```

```
RB# configure terminal
RB(config)# interface eth0.1
RB(config-ip)# ip address 172.16.0.2/30
RB(config-ip)# exit
RB(config)# exit
```

Verify that routers can reach each other using the ping command:
```
RA# ping 172.16.0.2/30
```
```
RB# ping 172.16.0.1/30
```


### Assign an IP address to the interface of `RB` connected to Network `B`
Add IP to `PCB`:
```
student@PCB:~$ sudo ip addr add 172.16.B.2/25 dev eth1
```

Configure `RB` interface w/ `PCB`:
```
RB# configure terminal
RB(config)# interface eth0.0
RB(config-ip)# ip address 172.16.B.1/25
RB(config-ip)# exit 
RB(config)# exit
```

Let's verify that the router and the host can reach each other using the ping command.

Check that you can ping from `PCB` to `RB`:
```
student@PCB:~$ ping 172.16.B.1/25
```


### Configure in both routers the routing table entries so that router A can reach network B and vice versa
Forward stuff to Network `B` (`172.16.B.0/25`) through `RB` `eth0.1` (`172.16.0.2`):
```
RA# configure terminal
RA(config)# ip route 172.16.B.0/25 172.16.0.2
```

Forward stuff to Network `A` (`172.16.A.0/25`) through `RA` `eth0.1` (`172.16.0.1`):
```
RB# configure terminal
RB(config)# ip route 172.16.A.0/25 172.16.0.1
```


### Configure the routing tables in `PCA` so it can reach Network `B`
Route stuff to the outside (`default`/`0.0.0.0/0`), and Network `B` (`172.16.B.0/25`) through `RA` `eth0.0` (`172.16.A.1`):
```
student@PCA:~$ sudo ip route add default via 172.16.A.1
student@PCA:~$ sudo ip route add 172.16.B.0/25 via 172.16.A.1
```


### Perform the corresponding settings in PCB
Route stuff to the outside (`default`/`0.0.0.0/0`), and Network `A` (`172.16.A.0/25`) through `RB` `eth0.0` (`172.16.B.1`):
```
student@PCB:~$ sudo ip route add default via 172.16.B.1
student@PCB:~$ sudo ip route add 172.16.A.0/25 via 172.16.B.1
```


### Use the `ping` and `traceroute` command from `PCA` to `PCB`
```
student@PCA:~$ ping 172.16.B.2
student@PCB:~$ ping 172.16.A.2
```
```
student@PCA:~$ traceroute -n 172.16.B.2
student@PCB:~$ traceroute -n 172.16.A.2
```

Close lightning with:
```
student@uc3m:~$ lightning stop
```


## Part II: Network configuration

### Start the VM and load the scenario
```
student@uc3m:~$ lightning start RYSCA/p_encam_a
```

Configuration:

![scenario2](img/scenario2.png)

**A = The last 2 digits of your NIA**

We'll use the `10.0.A.0/24` segment.

| Network |      Segment      |
|---------|-------------------|
| `Ofi1`  | `10.0.A.0/25`     |
| `Ofi2`  | `10.0.A.128/27`   |
|  `Ser`  | `10.0.A.160/28`   |
| `R1-R2` | `10.0.A.176/30`   |
| `R1-R3` | `10.0.A.180/30`   |
| `R2-R3` | `10.0.A.184/30`   |
| `R3-R4` | `10.0.A.188/30`   |
|  `BB`   | `10.0.0.0/24`     |
| `B100`  | `10.0.100.0/24`   |


| Device    | Interface |       IP          |
|-----------|-----------|-------------------|
|   `R1`    | `eth0.1`  | `10.0.A.1/25`     |
|           | `eth0.2`  | `10.0.A.182/30`   |
|           | `eth0.3`  | `10.0.A.177/30`   |
|   `R2`    | `eth0.1`  | `10.0.A.129/27`   |
|           | `eth0.2`  | `10.0.A.178/30`   |
|           | `eth0.3`  | `10.0.A.185/30`   |
|   `R3`    | `eth0.1`  | `10.0.A.161/28`   |
|           | `eth0.2`  | `10.0.A.189/30`   |
|           | `eth0.3`  | `10.0.A.181/30`   |
|           | `eth0.4`  | `10.0.A.186/30`   |
|   `R4`    | `eth0.1`  | `10.0.0.A/24`     |
|           | `eth0.2`  | `10.0.A.190/30`   |
| `hstOfi1` | `eth1`    | `10.0.A.2/25`     |
| `hstOfi2` | `eth1`    | `10.0.A.130/27`   |
|  `R100`   | `eth0.1`  | `10.0.0.100/24`   |
|           | `eth0.3`  | `10.0.100.100/24` |

**DO NOT TOUCH R100.**


### Assign IP addresses to the router interfaces
(for each router `RX`, each interface `eth0.Y`):
```
RX# configure terminal
RX(config)# interface eth0.Y
RX(config-if)# no ip address <old_ip>
RX(config-ip)# ip address <new_ip>  # ip with prefix
RX(config-ip)# exit 
RX(config)# exit
```

Also remember to delete wlan0.


### Assign IP addresses to the hosts
(for each host `hstOfiX`):
```
student@hstOfiX:~$ ip a  # show ip
student@hstOfiX:~$ sudo ip addr del <old_ip> dev eth1
student@hstOfiX:~$ sudo ip addr add <new_ip> dev eth1  # ip with prefix
```


### Check that connectivity exists between PCs `hstOfi1`, `hstOfi2` and the routers `R1`, `R2`
Ping from hstOfiX to RY (for each host hstOfiX, router RY):
```
student@hstOfiX:~$ ping <ip RY eth0.1>  # ip without prefix
RY# ping <ip hstOfiX>
```


### Check connectivity in each of the point-to-point network that interconnects the routers
Ping from `RX` `ethI` to `RY` `ethJ` (for each pair of routers in the same network, using the correct interfaces, the ones "pointing" to the other router):
```
RX# ping <ip RY ethJ>
RY# ping <ip RX ethI>
```


### Configure the required static routes in the routers (routing tables)
Remember to put all routes, and that direct connections don't need to be configured.  

The routing tables are:  

R1
| Network | Destination      |    Next Hop   | Metric |
|---------|------------------|---------------|:------:|
| `Ofi2`  | `10.0.A.128/27`  | `10.0.A.178`  | 1      |
|         | `10.0.A.128/27`  | `10.0.A.181`  | 2      |
| `Ser`   | `10.0.A.160/28`  | `10.0.A.181`  | 1      |
|         | `10.0.A.160/28`  | `10.0.A.178`  | 2      |
| `R2-R3` | `10.0.A.184/30`  | `10.0.A.178`  | 1      |
|         | `10.0.A.184/30`  | `10.0.A.181`  | 2      |
| `R3-R4` | `10.0.A.188/30`  | `10.0.A.178`  | 2      |
|         | `10.0.A.188/30`  | `10.0.A.181`  | 1      |
|  `BB`   | `10.0.0.0/24`    | `10.0.A.181`  | 1      |
|         | `10.0.0.0/24`    | `10.0.A.178`  | 2      |
| `B100`  | `10.0.100.0/24`  | `10.0.A.181`  | 1      |
|         | `10.0.100.0/24`  | `10.0.A.178`  | 2      |


R2
| Network | Destination      |    Next Hop   | Metric |
|---------|------------------|---------------|:------:|
| `Ofi1`  | `10.0.A.0/25`    | `10.0.A.177`  | 1      |
|         | `10.0.A.0/25`    | `10.0.A.186`  | 2      |
| `Ser`   | `10.0.A.160/28`  | `10.0.A.186`  | 1      |
|         | `10.0.A.160/28`  | `10.0.A.177`  | 2      |
| `R1-R3` | `10.0.A.180/30`  | `10.0.A.177`  | 1      |
|         | `10.0.A.180/30`  | `10.0.A.186`  | 2      |
| `R3-R4` | `10.0.A.188/30`  | `10.0.A.186`  | 1      |
|         | `10.0.A.188/30`  | `10.0.A.177`  | 2      |
|  `BB`   | `10.0.0.0/24`    | `10.0.A.186`  | 1      |
|         | `10.0.0.0/24`    | `10.0.A.177`  | 2      |
| `B100`  | `10.0.100.0/24`  | `10.0.A.186`  | 1      |
|         | `10.0.100.0/24`  | `10.0.A.177`  | 2      |


R3
| Network | Destination      |    Next Hop   | Metric |
|---------|------------------|---------------|:------:|
| `Ofi1`  | `10.0.A.0/25`    | `10.0.A.182`  | 1      |
|         | `10.0.A.0/25`    | `10.0.A.185`  | 2      |
| `Ofi2`  | `10.0.A.128/27`  | `10.0.A.185`  | 1      |
|         | `10.0.A.128/27`  | `10.0.A.182`  | 2      |
| `R1-R2` | `10.0.A.176/30`  | `10.0.A.182`  | 1      |
|         | `10.0.A.176/30`  | `10.0.A.185`  | 2      |
|  `BB`   | `10.0.0.0/24`    | `10.0.A.190`  | 1      |
| `B100`  | `10.0.100.0/24`  | `10.0.A.190`  | 1      |


R4
| Network | Destination      |    Next Hop   | Metric |
|---------|------------------|---------------|:------:|
| `A`     | `10.0.A.0/24`    | `10.0.A.189`  | 1      |
| `B100`  | `10.0.100.0/24`  | `10.0.0.100`  | 1      |


R100
| Network | Destination      |    Next Hop   | Metric |
|---------|------------------|---------------|:------:|
| `A`     | `10.0.A.0/24`    | `10.0.0.A`    | 1      |

The metrics are used to set the priority of each route. By default, it's `1` (max priority), and it's recommended to set the secondary routes to the number of hops between the origin and the destination (`2` in these cases) (you can leave the rest empty).


Let's configure the tables (for each router `RX`, each entry in the routing table):
```
RX# configure terminal
RX(config)# ip route <dest> <next hop> <metric>
```

You can check the configuration with:
```
RX# show ip route
```

To delete a configured route:
```
RX(config)# no ip route <dest> <next hop>
```


### Configure the required static routes in the hosts (routing tables)
(for each host `hstOfiX` connected to router `RY`):
```
student@hstOfiX:~$ sudo ip route add default via <ip RY eth0.1>
```

You can check the configuration with:
```
student@hstOfiX:~$ ip route
```

To delete a configured route:
```
student@hstOfiX:~$ sudo ip route del <dest> <next hop>
```


### Check that the network is connected
You can both use `ping` and `traceroute` (`traceroute` is more useful for debugging).  

Let's check if it reaches from `hstOfi1` to `hstOfi2`, and vice versa:
```
student@hstOfi1:~$ ping 10.0.A.130
```
```
student@hstOfi2:~$ ping 10.0.A.2
```

Now let's check if it reaches the outside and R100:
```
student@hstOfi2:~$ ping 10.0.0.A
```
```
student@hstOfi2:~$ ping 10.0.100.100
```


### Force a link failure, use the interface configuration command shutdown to disable the interfaces on both the routers connected to the link
Let's cut link between `R1` and `R2` and try to reach from `hstOfi1` to `hstOfi2`:
```
R1(config)# interface eth0.3
R1(config-if)# shutdown
```

```
R2(config)# interface eth0.2
R2(config-if)# shutdown
```

To restore the link:
```
R1(config-if)# no shutdown
```
```
R2(config-if)# no shutdown
```

And traceroute:
```
student@hstOfi1:~$ traceroute -n 10.0.A.130
```


## Part III: RIP protocol

First remove the static routes previously configured in the routers (`no ip route`), or re-initalize the scenario by running:
```
student@uc3m:~$ lightning stop && lightning start RYSCA/p_encam_a
```
And then re-configure the IPs of every router and PC the same way as in **PART II**. 


### Enable and configure the dynamic routing protocol RIP in each router for each link which contains at least one other router (not PC)
(for each router `RX` in network `NY` through interface `eth0.Z`):
```
Rx# configure terminal
Rx(config)# router rip
Rx(config-router)# network <ip NY>
Rx(config-router)# network eth0.Z
```


### Enable and configure the dynamic routing protocol RIP in passive mode in each router for each link that is connected to a PC (where applicable)
(for each router `RX` in network `NY` through interface `eth0.Z`):
```
Rx# configure terminal
Rx(config)# router rip
Rx(config-router)# network <ip NY>
Rx(config-router)# passive-interface eth0.Z
```

You can check your config with:
```
Rx# show ip rip
Rx# show ip rip status
```

To delete something:
```
Rx(config-router)# no network <ip NY>
Rx(config-router)# no network eth0.Z
```
Note that it takes a while for the RIP algorithm to realize that a link is broken, so initially pings and traceroutes may not work, until it eventually updates.


### Check that it works
Ping from router to router, and from PC to PC:
```
student@hstOfi1:~$ ping 10.0.A.130
```
```
student@hstOfi2:~$ ping 10.0.A.2
```

Now let's check if it reaches the outside and R100:
```
student@hstOfi2:~$ ping 10.0.0.A
```
```
student@hstOfi2:~$ ping 10.0.100.100
```
