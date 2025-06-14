# Networking Complete Guide notes

## 1. Core Concepts

### 1.1 OSI Model
1. Physical Layer (Layer 1)
   - Hardware transmission
   - Cables and connectors
   - Network interface cards
   - Repeaters and hubs

2. Data Link Layer (Layer 2)
   - MAC addressing
   - Error detection
   - Ethernet frames
   - Switches and bridges

3. Network Layer (Layer 3)
   - IP addressing
   - Routing
   - Packet forwarding
   - Routers

4. Transport Layer (Layer 4)
   - TCP/UDP protocols
   - Port numbers
   - Connection management
   - Flow control

5. Session Layer (Layer 5)
   - Session management
   - Authentication
   - Synchronization
   - Dialog control

6. Presentation Layer (Layer 6)
   - Data encryption
   - Data compression
   - Character encoding
   - Format conversion

7. Application Layer (Layer 7)
   - End-user protocols
   - HTTP, FTP, SMTP
   - DNS, DHCP
   - Application services

## 2. IP Addressing

### 2.1 IPv4 Addressing
```
# Address Classes
Class A: 0.0.0.0 to 127.255.255.255    # /8
Class B: 128.0.0.0 to 191.255.255.255  # /16
Class C: 192.0.0.0 to 223.255.255.255  # /24
Class D: 224.0.0.0 to 239.255.255.255  # Multicast
Class E: 240.0.0.0 to 255.255.255.255  # Reserved

# Private Address Ranges
10.0.0.0/8        # Class A private
172.16.0.0/12     # Class B private
192.168.0.0/16    # Class C private
```

### 2.2 IPv6 Addressing
```
# Address Format
2001:0db8:85a3:0000:0000:8a2e:0370:7334

# Address Shortening Rules
- Remove leading zeros
- Replace consecutive zeros with ::
Example: 2001:db8:85a3::8a2e:370:7334

# Special Addresses
::/128           # Unspecified
::1/128          # Loopback
fe80::/10        # Link-local
ff00::/8         # Multicast
```

### 2.3 Subnetting
```
# Subnet Mask Examples
/24 = 255.255.255.0
/16 = 255.255.0.0
/8  = 255.0.0.0

# VLSM Calculation
Network: 192.168.1.0/24
Subnet 1: 192.168.1.0/25   # 126 hosts
Subnet 2: 192.168.1.128/26 # 62 hosts
Subnet 3: 192.168.1.192/26 # 62 hosts
```

## 3. Protocols

### 3.1 TCP/IP Protocol Suite
```
# Application Layer Protocols
HTTP    80    # Web traffic
HTTPS   443   # Secure web
FTP     21    # File transfer
SSH     22    # Secure shell
SMTP    25    # Email sending
POP3    110   # Email receiving
IMAP    143   # Email management
DNS     53    # Name resolution
DHCP    67/68 # IP assignment

# Transport Layer
TCP # Connection-oriented
- Three-way handshake
- Reliable delivery
- Flow control
- Error recovery

UDP # Connectionless
- Fast delivery
- No guarantee
- No flow control
- Broadcast support
```

### 3.2 Routing Protocols
```
# Interior Gateway Protocols (IGP)
RIP
- Distance vector
- Hop count metric
- Max 15 hops
- Updates every 30s

OSPF
- Link state
- Cost-based metric
- Area hierarchy
- Fast convergence

EIGRP
- Advanced distance vector
- Complex metric
- Fast convergence
- Cisco proprietary

# Exterior Gateway Protocols (EGP)
BGP
- Path vector
- Policy-based routing
- Internet backbone
- AS path selection
```

## 4. Network Security

### 4.1 Firewall Types
```
# Packet Filtering
- Layer 3/4 filtering
- Stateless inspection
- Basic access control
Example:
iptables -A INPUT -p tcp --dport 80 -j ACCEPT

# Stateful Inspection
- Connection tracking
- Dynamic rule updates
- Enhanced security
Example:
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Application Layer
- Deep packet inspection
- Protocol awareness
- Content filtering
Example:
squid.conf:
acl ALLOWED_SITES dstdomain .example.com
http_access allow ALLOWED_SITES
```

### 4.2 VPN Technologies
```
# Site-to-Site VPN
ipsec.conf:
conn site-to-site
    type=tunnel
    left=192.168.1.1
    leftsubnet=10.0.1.0/24
    right=192.168.2.1
    rightsubnet=10.0.2.0/24
    auto=start

# Remote Access VPN
openvpn.conf:
port 1194
proto udp
dev tun
ca ca.crt
cert server.crt
key server.key
dh dh2048.pem
server 10.8.0.0 255.255.255.0
```

## 5. Network Services

### 5.1 DNS Configuration
```
# BIND Configuration
named.conf:
zone "example.com" {
    type master;
    file "/etc/bind/zones/example.com.db";
};

# Zone File
example.com.db:
$TTL 86400
@   IN  SOA  ns1.example.com. admin.example.com. (
        2021010101  ; Serial
        3600        ; Refresh
        1800        ; Retry
        604800      ; Expire
        86400       ; Minimum TTL
)
@       IN  NS   ns1.example.com.
@       IN  A    192.168.1.10
www     IN  A    192.168.1.20
mail    IN  MX   10 mail.example.com.
```

### 5.2 DHCP Setup
```
# ISC DHCP Configuration
dhcpd.conf:
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 8.8.8.8, 8.8.4.4;
    option domain-name "example.com";
    default-lease-time 86400;
    max-lease-time 172800;
}
```

## 6. Load Balancing

### 6.1 Layer 4 Load Balancing
```
# HAProxy Configuration
haproxy.cfg:
frontend http_front
    bind *:80
    default_backend http_back

backend http_back
    balance roundrobin
    server web1 192.168.1.10:80 check
    server web2 192.168.1.11:80 check
    server web3 192.168.1.12:80 check
```

### 6.2 Layer 7 Load Balancing
```
# Nginx Configuration
nginx.conf:
upstream backend {
    least_conn;
    server 192.168.1.10:8080;
    server 192.168.1.11:8080;
    server 192.168.1.12:8080;
}

server {
    listen 80;
    server_name example.com;

    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

## 7. Network Troubleshooting

### 7.1 Common Commands
```bash
# Connectivity Testing
ping -c 4 8.8.8.8                  # Basic connectivity
traceroute google.com              # Path tracing
mtr google.com                     # Continuous trace
nc -zv 192.168.1.10 80            # Port testing

# DNS Troubleshooting
nslookup example.com              # Basic lookup
dig example.com                   # Detailed DNS info
dig @8.8.8.8 example.com         # Specific server
host -t MX example.com           # Mail records

# Network Statistics
netstat -tuln                    # Open ports
ss -s                           # Socket statistics
iftop                          # Bandwidth usage
tcpdump -i eth0 port 80        # Packet capture
```

### 7.2 Performance Analysis
```bash
# Bandwidth Testing
iperf3 -s                      # Server mode
iperf3 -c server_ip           # Client mode

# Network Load
top                          # System overview
htop                        # Enhanced top
nload eth0                 # Interface load
nethogs                   # Per-process usage
```

## 8. Cloud Networking

### 8.1 AWS VPC Configuration
```
# VPC Setup
aws ec2 create-vpc --cidr-block 10.0.0.0/16

# Subnet Creation
aws ec2 create-subnet \
    --vpc-id vpc-123456 \
    --cidr-block 10.0.1.0/24 \
    --availability-zone us-east-1a

# Route Table
aws ec2 create-route-table --vpc-id vpc-123456
aws ec2 create-route \
    --route-table-id rtb-123456 \
    --destination-cidr-block 0.0.0.0/0 \
    --gateway-id igw-123456
```

### 8.2 Azure Virtual Network
```
# Network Creation
az network vnet create \
    --name myVNet \
    --resource-group myRG \
    --location eastus \
    --address-prefix 10.0.0.0/16

# Subnet Configuration
az network vnet subnet create \
    --name mySubnet \
    --vnet-name myVNet \
    --resource-group myRG \
    --address-prefix 10.0.1.0/24
```

## 9. Container Networking

### 9.1 Docker Networking
```
# Network Creation
docker network create \
    --driver bridge \
    --subnet 172.18.0.0/16 \
    --gateway 172.18.0.1 \
    mynetwork

# Container Connection
docker run -d \
    --name web \
    --network mynetwork \
    nginx

# Network Inspection
docker network inspect mynetwork
```

### 9.2 Kubernetes Networking
```yaml
# Pod Networking
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80

# Service Configuration
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
```

## 10. Network Automation

### 10.1 Ansible Network Automation
```yaml
# Router Configuration
- name: Configure Router
  hosts: routers
  tasks:
    - name: Set Interface IP
      ios_config:
        lines:
          - ip address 192.168.1.1 255.255.255.0
        parents: interface GigabitEthernet0/1

# Switch VLAN Setup
    - name: Configure VLANs
      ios_vlan:
        vlan_id: 100
        name: Users
        state: present
```

### 10.2 Network Monitoring
```yaml
# Prometheus Configuration
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'network'
    static_configs:
      - targets: ['192.168.1.10:9100']

# Grafana Dashboard
{
  "dashboard": {
    "panels": [
      {
        "title": "Network Traffic",
        "type": "graph",
        "datasource": "Prometheus",
        "targets": [
          {
            "expr": "rate(node_network_receive_bytes_total[5m])"
          }
        ]
      }
    ]
  }
} 
