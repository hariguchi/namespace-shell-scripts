# namespace-shell-funcs -- Shell Functions for Linux Namespace

A set of shell functions working with Linux Namespace

* **ns_add**:          Add namespaces
* **ns_del**:          Delete namespaces
* **ns_add_if**:       Add an interface to namespace
* **vif_add**:         Create a pair of veth interfaces
* **vif_add_pair**:    Create a pair of veth interfaces from interface names
* **vif_del**:         Delete a (pair of) veth interface(s)
* **vif_peer_index**:  Return peer vif's ifindex
* **ns_add_ifaddr**:   Attach an IPv4/IPv6 address to the specified interface and namespace
* **ns_flush_ifaddr**: Delete all IPv4/IPv6 addresses from the interface in the specified namespace
* **ns_exec**:         Execute a command in the specified namespace
* **br_add**:          Add a kernel bridge
* **br_del**:          Delete a kernel bridge
* **br_addif**:        Add an interface to a bridge
* **br_delif**:        Delete an interface from a bridge
* **pci2if**:          Convert pci address to interface name


## Functions

### **ns_add** -- Add namespaces
```
ns_add ns1 [ns2 ...]
```

### **ns_del** -- Delete namespaces
```
ns_del [ns1 ...]
```

All the existing namespaces are deleted if no arguments are
specified.

### **ns_add_if** -- Add an interface to namespace
```
ns_add_if namespace interface
```
Example:
```
# . ./namespace-shell-funcs
# ns_add ns1
adding ns1
# ns_add_if ns1 eth1
#
```

### **vif_add** -- Create a pair of veth interfaces
```
vif_add veth_name veth_name
```
Example:
```
# . ./namespace-shell-funcs
# vif_add veth1 veth2
#
```

### **vif_add_pair** -- Create a pair of veth interfaces from interface names
```
vif_add_pair if1 if2
```
Example:
```
# . ./namespace-shell-funcs
# vif_add_pair if1 if2
# ip link | grep if1
7: if2-if1@if1-if2: ...
8: if1-if2@if2-if1: ...
#
```

### **vif_del** -- Delete a (pair of) veth interface(s)
```
vif_del veth_name
```
The peer veth interface is also deleted if the specified veth
interface has the peer.

Example:
```
# . ./namespace-shell-funcs
# vif_del veth1
#
```

### **vif_peer_index** -- Return peer vif's ifindex
```
vif_peer_index interface
```
Example:
```
# . ./namespace-shell-funcs
# vif_add veth1 veth2
# vif_peer_index veth1
6
# vif_peer_index veth2
7
#
```

# **ns_add_if** -- Add an interface to namespace
```
ns_add_if namespace interface
```
Example:
```
# . ./namespace-shell-funcs
# ns_add ns1 ns2      # Add two namespafes: ns1, ns2
adding ns1
adding ns2
# vif_add veth1 veth2 # Create two veths: veth1, veth2
# ns_add_if ns1 veth1 # Add veth1 to ns1
# ns_add_if ns2 veth2 # Add veth2 to ns2
```

# **ns_add_ifaddr** -- Attach an IPv4/IPv6 address to the specified interface and namespace
```
ns_add_ifaddr namespace interface prefix [mtu]
```
Example:

Assume you want to create the following network.
```
  +-----+ veth1                          veth2 +-----+
  | ns1 +--------------------------------------+ ns2 |
  +-----+ 172.16.1.1/24          172.16.1.2/24 +-----+
          2001:0:0:1::1/64    2001:0:0:1::2/64
```
Type the following commands.
```
# . ./namespace-shell-funcs
# ns_add ns1 ns2                             # Add two namespafes: ns1, ns2
adding ns1
adding ns2
# vif_add veth1 veth2                        # Create two veths: veth1, veth2
# ns_add_if ns1 veth1                        # Add veth1 to ns1
# ns_add_if ns2 veth2                        # Add veth2 to ns2
# ns_add_ifaddr ns1 veth1 172.16.1.1/24 1500 # Add 172.16.1.1/24 to veth1
# ns_add_ifaddr ns2 veth2 172.16.1.2/24 1500 # Add 172.16.1.2/24 to veth2
# ns_add_ifaddr ns1 veth1 2001:0:0:1::1/64   # Add 2001:0:0:1::1/64 to veth1
# ns_add_ifaddr ns2 veth2 2001:0:0:1::2/64   # Add 2001:0:0:1::2/64 to veth2
# ip netns exec ns1 ping 172.16.1.2
PING 172.16.1.2 (172.16.1.2) 56(84) bytes of data.
64 bytes from 172.16.1.2: icmp_seq=1 ttl=64 time=0.049 ms
64 bytes from 172.16.1.2: icmp_seq=2 ttl=64 time=0.034 ms
64 bytes from 172.16.1.2: icmp_seq=3 ttl=64 time=0.110 ms
^C
--- 172.16.1.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1027ms
rtt min/avg/max/mdev = 0.034/0.041/0.049/0.009 ms
# ip netns exec ns1 ping6 2001:0:0:1::2
PING 2001:0:0:1::2(2001:0:0:1::2) 56 data bytes
64 bytes from 2001:0:0:1::2: icmp_seq=1 ttl=64 time=0.075 ms
64 bytes from 2001:0:0:1::2: icmp_seq=2 ttl=64 time=0.040 ms
64 bytes from 2001:0:0:1::2: icmp_seq=3 ttl=64 time=0.055 ms
^C
--- 2001:0:0:1::2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2052ms
rtt min/avg/max/mdev = 0.040/0.056/0.075/0.016 ms
#
```

# **ns_flush_ifaddr** -- Delete all IPv4/IPv6 addresses from the interface in the specified namespace
```
ns_flush_ifaddr namespace interface
```
Example:
```
# . ./namespace-shell-funcs
# ns_add ns1 ns2                             # Add two namespafes: ns1, ns2
adding ns1
adding ns2
# vif_add veth1 veth2                        # Create two veths: veth1, veth2
# ns_add_if ns1 veth1                        # Add veth1 to ns1
# ns_add_if ns2 veth2                        # Add veth2 to ns2
# ns_add_ifaddr ns1 veth1 172.16.1.1/24 1500 # Add 172.16.1.1/24 to veth1
# ns_add_ifaddr ns1 veth1 2001:0:0:1::1/64   # Add 2001:0:0:1::1/64 to veth1
# ip netns exec ns1 ifconfig veth1
veth1     Link encap:Ethernet  HWaddr 52:6b:d7:8e:78:7e  
          inet addr:172.16.1.1  Bcast:0.0.0.0  Mask:255.255.255.0
          inet6 addr: 2001:0:0:1::1/64 Scope:Global
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:48 errors:0 dropped:0 overruns:0 frame:0
          TX packets:49 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:4824 (4.8 KB)  TX bytes:5032 (5.0 KB)
#
# ns_flush_ifaddr ns1 veth1
# ip netns exec ns1 ifconfig veth1
veth1     Link encap:Ethernet  HWaddr 52:6b:d7:8e:78:7e  
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:48 errors:0 dropped:0 overruns:0 frame:0
          TX packets:51 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:4824 (4.8 KB)  TX bytes:5212 (5.2 KB)
```

# **ns_exec** -- Execute a command in the specified namespace
```
ns_exec namespace cmd [...]
```
Example:
```
# . ./namespace-shell-funcs
# ns_add ns1 ns2      # Add two namespafes: ns1, ns2
adding ns1
adding ns2
# vif_add veth1 veth2 # Create two veths: veth1, veth2
# ns_add_if ns1 veth1 # Add veth1 to ns1
# ns_exec ns1 ifconfig veth1
veth1     Link encap:Ethernet  HWaddr 5e:56:e9:e8:82:56  
          inet6 addr: fe80::5c56:e9ff:fee8:8256/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:27 errors:0 dropped:0 overruns:0 frame:0
          TX packets:26 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:3195 (3.1 KB)  TX bytes:2882 (2.8 KB)
```

# **br_add** -- Create a kernel bridge
```
br_add br1
```

# **br_del** -- Delete a kernel bridge
```
br_del br1
```

# **br_addif** -- Add an interface to a bridge
```
br_addif bridge interface
```
Example:

Assume you want to create the following network.
```
  +-----+ ns1-br1    br1-ns1 +-----+ br1-ns2    ns2-br1 +-----+
  | ns1 +--------------------+ br1 +--------------------+ ns2 |
  +-----+ 172.16.1.1/24      +-----+      172.16.1.2/24 +-----+
          2001:0:0:1::1/64             2001:0:0:1::2/64
```
Type the following commands.
```
# . ./namespace-shell-funcs
# ns_add ns1 ns2                          # Add two namespafes: ns1, ns2
adding ns1
adding ns2
# vif_add ns1-br1 br1-ns1                 # Create two veths: ns1-br1, br1-ns1
# vif_add ns2-br1 br1-ns2                 # Create two veths: ns2-br1, br1-ns2
# ns_add_if ns1 ns1-br1                   # Add ns1-br1 to ns1
# ns_add_if ns2 ns2-br1                   # Add ns2-br1 to ns2
# ns_add_ifaddr ns1 ns1-br1 172.16.1.1/24 # Add 172.16.1.1/24 to ns1-br1
# ns_add_ifaddr ns2 ns2-br1 172.16.1.2/24 # Add 172.16.1.2/24 to ns2-br1
# br_add br1                              # Create kernel bridge: br1
# br_addif br1 br1-ns1                    # Add br1-ns1 to br1
# br_addif br1 br1-ns2                    # Add br1-ns2 to br1
#
# ns_exec ns1 ping 172.16.1.2
PING 172.16.1.2 (172.16.1.2) 56(84) bytes of data.
64 bytes from 172.16.1.2: icmp_seq=1 ttl=64 time=0.040 ms
64 bytes from 172.16.1.2: icmp_seq=2 ttl=64 time=0.049 ms
64 bytes from 172.16.1.2: icmp_seq=3 ttl=64 time=0.039 ms
^C
--- 172.16.1.2 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2047ms
rtt min/avg/max/mdev = 0.039/0.042/0.049/0.008 ms
```

# **br_delif** -- Delete an interface from a bridge
```
br_delif bridge interface
```

# **pci2if** -- Convert pci address to interface name
```
pci2if pci_address
```
Example:
```
# . ./namespace-shell-funcs
# pci2if 00:08.0
enp0s8
#
```