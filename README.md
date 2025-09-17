# 🖥️ Linux Network Namespace Simulation Assignment

## 📚 Background
Network namespaces in Linux allow for isolated network environments within a single host.  
In this lab, we simulate two separate networks connected via a router using Linux network namespaces and bridges.

---

## 🌐 Network Topology Diagram
<img width="1069" height="632" alt="Untitled" src="https://github.com/user-attachments/assets/775e0591-a87e-476a-8473-53fe618d8aaf" />


# ⚙️ Implementation Steps

## 1️⃣ Create Bridges
```bash
ip link add br0 type bridge
ip link add br1 type bridge
ip link set br0 up
ip link set br1 up
```
 ## 2️⃣ Create Namespaces 
 ```bash
ip netns add ns1
ip netns add ns2
ip netns add router-ns
```
## 3️⃣ Create Veth Pairs and Connect to Bridges/Namespaces
### ns1 <-> br0
```bash
ip link add veth-ns1-host type veth peer name veth-ns1
ip link set veth-ns1-host master br0
ip link set veth-ns1-host up
ip link set veth-ns1 netns ns1
```
### ns2 <-> br1
```bash
ip link add veth-ns2-host type veth peer name veth-ns2
ip link set veth-ns2-host master br1
ip link set veth-ns2-host up
ip link set veth-ns2 netns ns2
```
### router <-> br0 

```bash
ip link add veth-rtr0-host type veth peer name veth-rtr0
ip link set veth-rtr0-host master br0
ip link set veth-rtr0-host up
ip link set veth-rtr0 netns router-ns
 ```
### router <-> br1
```bash
ip link add veth-rtr1-host type veth peer name veth-rtr1
ip link set veth-rtr1-host master br1
ip link set veth-rtr1-host up
ip link set veth-rtr1 netns router-ns
 ```
4️⃣ Assign IP Addresses and Bring Up Interfaces
### ns1 
```bash
ip netns exec ns1 ip link set dev lo up
ip netns exec ns1 ip link set dev veth-ns1 up
ip netns exec ns1 ip addr add 10.0.1.2/24 dev veth-ns1
 ```
### ns2 

```bash
ip netns exec ns2 ip link set dev lo up
ip netns exec ns2 ip link set dev veth-ns2 up
ip netns exec ns2 ip addr add 10.0.2.2/24 dev veth-ns2
 ```

### router-ns 
```bash
ip netns exec router-ns ip link set dev lo up
ip netns exec router-ns ip link set dev veth-rtr0 up
ip netns exec router-ns ip addr add 10.0.1.1/24 dev veth-rtr0
ip netns exec router-ns ip link set dev veth-rtr1 up
ip netns exec router-ns ip addr add 10.0.2.1/24 dev veth-rtr1
 ```

## 5️⃣ Configure Default Routes
```bash 
ip netns exec ns1 ip route add default via 10.0.1.1
ip netns exec ns2 ip route add default via 10.0.2.1
 ```
## 6️⃣ Enable IP Forwarding in Router
 
```bash 
ip netns exec router-ns sysctl -w net.ipv4.ip_forward=1
 ```
## ⚠️ Troubleshooting: Bridge Netfilter (✅ Recommended)
If ping between ns1 and ns2 fails:

```bash
sysctl -w net.bridge.bridge-nf-call-iptables=0
sysctl -w net.bridge.bridge-nf-call-ip6tables=0
```
## 📝 IP Addressing Scheme

| Namespace   | Interface   | IP Address     | Subnet         | Default Gateway |
|-------------|-------------|----------------|----------------|------------------|
| ns1         | veth-ns1    | 10.0.1.2/24     | 10.0.1.0/24     | 10.0.1.1         |
| router-ns   | veth-rtr0   | 10.0.1.1/24     | 10.0.1.0/24     | —                |
| ns2         | veth-ns2    | 10.0.2.2/24     | 10.0.2.0/24     | 10.0.2.1         |
| router-ns   | veth-rtr1   | 10.0.2.1/24     | 10.0.2.0/24     | —                |

## 🌐 Routing Explanation
ns1 and ns2 are in different subnets (10.0.1.0/24 and 10.0.2.0/24).

Router namespace (router-ns) connects both subnets and forwards traffic.

IP forwarding is enabled in router-ns.

ns1 sends traffic to 10.0.2.2 via 10.0.1.1, router forwards to ns2, and reply returns via 10.0.2.1.

# ✅ Verification & Testing
## Check Bridges & Ports
```bash
ip link show type bridge
bridge link show
```
## Result
<img width="910" height="216" alt="image" src="https://github.com/user-attachments/assets/fa30c81a-bbb0-4754-8740-0d9b49274c8b" />
<img width="908" height="170" alt="image" src="https://github.com/user-attachments/assets/0f81fa67-aa64-4c71-93a6-83f4b9477f97" />

## Check Namespaces & IPs

 ```bash
ip netns list
ip netns exec ns1 ip -br a
ip netns exec ns2 ip -br a
ip netns exec router-ns ip -br a
```
## Result
<img width="662" height="268" alt="image" src="https://github.com/user-attachments/assets/a1307f54-41bc-4f2a-b903-7a265dfdfae7" />

## Routing Tables
```bash
ip netns exec ns1 ip r
ip netns exec ns2 ip r
ip netns exec router-ns ip r
```
## Result
<img width="574" height="175" alt="image" src="https://github.com/user-attachments/assets/eeb25b38-2951-4284-8c7a-ddd383cf66e4" />

## Forwarding Status
```bash
ip netns exec router-ns sysctl net.ipv4.ip_forward
```
## Result
<img width="681" height="38" alt="image" src="https://github.com/user-attachments/assets/f5e2976b-8bc6-452e-87cf-8f9d05b49c99" />

Connectivity Tests
## Gateway reachability
 ```bash
ip netns exec ns1 ping -c 2 10.0.1.1
ip netns exec ns2 ping -c 2 10.0.2.1
```
## Result
<img width="582" height="302" alt="image" src="https://github.com/user-attachments/assets/45f8e01f-0b6f-4d9d-b468-b4bbc6f66717" />


## End-to-end connectivity
 ```bash
ip netns exec ns1 ping -c 3 10.0.2.2
ip netns exec ns2 ping -c 3 10.0.1.2
```
## Result
<img width="571" height="344" alt="image" src="https://github.com/user-attachments/assets/5a956baf-eae3-4d02-b2a7-a8a601dc4795" />

## Expected Results:

Ping to gateways: 0% packet loss

Ping ns1↔ns2: 0–1ms, 0% loss

Traceroute shows 2 hops: 10.0.1.1 → 10.0.2.2

# 🧹 Cleanup

## Disable IP forwarding (optional)
```bash
ip netns exec router-ns sysctl -w net.ipv4.ip_forward=0 2>/dev/null || true
```
## Delete namespaces
```bash
ip netns del ns1 2>/dev/null || true
ip netns del ns2 2>/dev/null || true
ip netns del router-ns 2>/dev/null || true
```

## Delete host-side veth ends
```bash
ip link del veth-ns1-host 2>/dev/null || true
ip link del veth-ns2-host 2>/dev/null || true
ip link del veth-rtr0-host 2>/dev/null || true
ip link del veth-rtr1-host 2>/dev/null || true
```

## Bring down & delete bridges
```bash
ip link set br0 down 2>/dev/null || true
ip link del br0 2>/dev/null || true
ip link set br1 down 2>/dev/null || true
ip link del br1 2>/dev/null || true
```
