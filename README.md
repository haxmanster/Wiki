# DPDK ON UBUNTU LTS 18.04

### BINDING DEVICE
```bash
modprobe igb_uio
dpdk-devbind -u 00:08.0
dpdk-devbind --bind=igb_uio 00:08.0
```

### INIT DPDK TO OPENVSWITCH
```
ovs-vsctl set Open_vSwitch . other_config:dpdk-init=true other_config:dpdk-socket-mem=128
```
### CRATE DPDK BRIDGE
```bash
ovs-vsctl add-br br0 -- set bridge br0 datapath_type=netdev

ovs-vsctl add-port br0 dpdk-p0 -- set Interface dpdk-p0 type=dpdk options:dpdk-devargs=0000:00:08.0
```
### ADD IP TO BRIDGE
```bash
ip addr add 192.168.1.1/24 dev br0
ip links set br0 up
```
### OPTIONAL: ADDING FLOW CONTROLER 
```bash
ovs-vsctl set-controller br0 tcp:10.28.6.24:6653
```
# ADD NAME SPACE 
```bash
ip netns add namespace4
ip link add vns4-br0 type veth peer name vpeerns4
ip link set vpeerns4 netns namespace4
ip netns exec namespace4 ip link
ip link set vns4-br0 up
ip netns exec namespace4 ip addr add 10.10.10.50/24 dev vpeerns4
ip netns exec namespace4 ip link set mtu 1450 dev vpeerns4
ip netns exec namespace4 ip link set vpeerns4 up
ip netns exec namespace4 ip link set dev lo up
ovs-vsctl add-port br0 vns4-br0
ovs-vsctl set port vns4-br0 tag=200

ip netns add namespace3
ip link add vns3 type veth peer name vpeerns3
ip link set vpeerns3 netns namespace3
ip netns exec namespace3 ip link
ip link set vns3 up
ip netns exec namespace3 ip addr add 10.10.10.60/24 dev vpeerns3
ip netns exec namespace3 ip link set mtu 1450 dev vpeerns3
ip netns exec namespace3 ip link set vpeerns3 up
ip netns exec namespace3 ip link set dev lo up
ovs-vsctl add-port br0 vns3
ovs-vsctl set port vns3 tag=200
```

