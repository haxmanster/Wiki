```
# install dependencies and enable essential network stuff iperf
apt-get update
apt-get install -y bridge-utils openvswitch-switch
modprobe 8021q
echo 1 > /proc/sys/net/ipv4/ip_forward

# openvswitch bridge create and info
ovs-vsctl add-br br0
ovs-vsctl show

# ns1
# add namespace
ip netns add namespace1
# add virtual ethernet "cable" with two ends
ip link add vns1 type veth peer name vpeerns1
# set virutal ethernet on to belong in namespace1
ip link set vpeerns1 netns namespace1
# show link info from the namespace
ip netns exec namespace1 ip link
# start veth port
ip link set vns1 up
# add network config to veth in namespace end
ip netns exec namespace1 ip addr add 10.10.10.10/24 dev vpeerns1
# change MTU to prevent problems with VXLAN tunneling
ip netns exec namespace1 ip link set mtu 1450 dev vpeerns1
# it's alive!
ip netns exec namespace1 ip link set vpeerns1 up
# add loopback just in case
ip netns exec namespace1 ip link set dev lo up
# connect second end of veth to switch
ovs-vsctl add-port br0 vns1
# tag this veth cable to use only vlan100
ovs-vsctl set port vns1 tag=100

# ns2
ip netns add namespace2
ip link add vns2 type veth peer name vpeerns2
ip link set vpeerns2 netns namespace2
ip netns exec namespace2 ip link
ip link set vns2 up
ip netns exec namespace2 ip addr add 10.10.10.20/24 dev vpeerns2
ip netns exec namespace2 ip link set mtu 1450 dev vpeerns2
ip netns exec namespace2 ip link set vpeerns2 up
ip netns exec namespace2 ip link set dev lo up
ovs-vsctl add-port br0 vns2
ovs-vsctl set port vns2 tag=100

# ns3
ip netns add namespace3
ip link add vns3 type veth peer name vpeerns3
ip link set vpeerns3 netns namespace3
ip netns exec namespace3 ip link
ip link set vns3 up
ip netns exec namespace3 ip addr add 10.10.10.10/24 dev vpeerns3
ip netns exec namespace3 ip link set mtu 1450 dev vpeerns3
ip netns exec namespace3 ip link set vpeerns3 up
ip netns exec namespace3 ip link set dev lo up
ovs-vsctl add-port br0 vns3
ovs-vsctl set port vns3 tag=200

# ns4
ip netns add namespace4
ip link add vns4 type veth peer name vpeerns4
ip link set vpeerns4 netns namespace4
ip netns exec namespace4 ip link
ip link set vns4 up
ip netns exec namespace4 ip addr add 10.10.10.20/24 dev vpeerns4
ip netns exec namespace4 ip link set mtu 1450 dev vpeerns4
ip netns exec namespace4 ip link set vpeerns4 up
ip netns exec namespace4 ip link set dev lo up
ovs-vsctl add-port br0 vns4
ovs-vsctl set port vns4 tag=200

# check traffic

#bwm-ng
#iptraf
#nload
#iftop
#tcpdump -nni vns3 icmp
#ip netns exec namespace3 ping 10.10.10.20
#ip netns exec namespace1 iperf -s
#ip netns exec namespace2 iperf -c 10.10.10.10
#ip netns exec namespace3 iperf -s
#ip netns exec namespace4 iperf -c 10.10.10.10

# docker
apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -
add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt-get update
apt-get install -y docker-ce

# floodlight
# https://hub.docker.com/r/glefevre/floodlight/
docker run -d -p 6653:6653 -p 8080:8080 --name=floodlight --restart always glefevre/floodlight 

# enable openflow on switch
ovs-vsctl set bridge br0 protocols=OpenFlow13
# show openflow info
ovs-appctl bridge/dump-flows br0
# connect switch to the floodlight controller, CHANGE IP!
ovs-vsctl set-controller br0 tcp:10.28.6.11:6653
# show openflow info
ovs-appctl bridge/dump-flows br0

# VXLAN tunnel to other VM, remember to CHANGE IP!
ovs-vsctl add-port br0 vxlan1 -- set interface vxlan1 type=vxlan options:remote_ip=192.168.1.2 options:key=flow options:dst_port=8472

# ns5
ip netns add namespace5
ip link add vns5 type veth peer name vpeerns5
ip link set vpeerns5 netns namespace5
ip netns exec namespace5 ip link
ip link set vns5 up
ip netns exec namespace5 ip addr add 10.10.10.10/24 dev vpeerns5
ip netns exec namespace5 ip link set mtu 1450 dev vpeerns5
ip netns exec namespace5 ip link set vpeerns5 up
ip netns exec namespace5 ip link set dev lo up
ovs-vsctl add-port br0 vns5
ovs-vsctl set port vns5 tag=300

# change bridge MAC to match ens3 interface, CHANGE MAC!
#ovs-vsctl set interface br0 mac=\"fa:16:3e:ac:b8:43\"
# connect switch to main port, RISKY!
#ovs-vsctl add-port br0 ens3

# network without namespace
ip link add vns0 type veth peer name vpeerns0
ip link set vns0 up
ip addr add 192.168.0.1/24 dev vpeerns0
ip link set mtu 1450 dev vpeerns0
ip link set vpeerns0 up
ovs-vsctl add-port br0 vns0

# sflow stats
# run on other VM with openflow controller
docker run --restart=always -d -p 8008:8008 -p 6343:6343/udp sflow/flow-trend
# check browser on http://10.28.6.24:8008
# send metrics from switch to collector
ovs-vsctl -- --id=@sflow create sflow agent=br0 target=\"10.28.6.24:6343\" sampling=1000 polling=5 -- set bridge br0 sflow=@sflow
#ovs-vsctl remove bridge br0 sflow <UUID>

# dpdk

# install stuff
apt-get install -y openvswitch-switch-dpdk linux-generic-hwe-16.04 dpdk-dev libdpdk-dev
# check current status, if card is using kernel driver we must configure dpdk correctly
dpdk_nic_bind --status
echo "pci 0000:00:04.0 vfio-pci" > /etc/dpdk/interfaces
# configure hugepages
echo "NR_2M_PAGES=128" > /etc/dpdk/dpdk.conf
#echo "NR_1G_PAGES=1" >> /etc/dpdk/dpdk.conf
# workaround for probably older OS or kernel, use ubuntu 18.04 in the future
echo "options vfio enable_unsafe_noiommu_mode=1" > /etc/modprobe.d/vfio-noiommu.conf

#update-alternatives --set ovs-vswitchd /usr/lib/openvswitch-switch-dpdk/ovs-vswitchd-dpdk
#echo "DPDK_OPTS='--dpdk -c 0x1 -n 4 -m 2048 --vhost-owner libvirt-qemu:kvm --vhost-perm 0664'" | tee -a /etc/default/openvswitch-switch
#ovs-vsctl set bridge br0 datapath_type=netdev
#service openvswitch-switch restart
```
