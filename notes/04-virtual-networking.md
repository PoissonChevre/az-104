# 4. Virtual Networking

**!** = exam trap

## VNets and subnets

- A VNet belongs to **one region** and **one subscription**. It has one or more address spaces (use RFC 1918 ranges).
- **!** Plan address spaces with **no overlap** with on-prem and other VNets. You need this later for peering and VPN.
- **!** Azure **reserves 5 IPs** in each subnet: `.0` (network), `.1` (gateway), `.2` and `.3` (Azure DNS), and the last one (broadcast).
  A /29 gives only **3** usable IPs, and the first VM gets `.4`.

## IP addresses

- **Private IP**: dynamic (default) or static, set **on the NIC**. **!** Never set a fixed IP inside the OS.
- **Public IP**: **!** **Standard SKU only** (Basic is retired) → always **static**, *secure by default* (closed until an NSG allows traffic), zone-redundant possible.
- A public IP goes on a **NIC**, a **Load Balancer**, a **NAT Gateway**, **Bastion**, an **App Gateway**… **!** **never on a subnet**.
- **Public IP prefix**: a block of contiguous public IPs.

## NAT Gateway

- Linked to a **subnet**. **Outbound only**.
- **!** For outbound traffic, it **wins** over the public IP of a VM.
- New VNets have **private subnets by default**: no implicit outbound access. You need a NAT Gateway, a public IP or a load balancer to reach the Internet.

## Network Security Groups (NSG)

- Rules use 5 values: source, source port, destination, destination port, protocol → **Allow** or **Deny**.
- **Priority 100–4096**: the lowest number is checked first, and the **first match wins**.
- **Stateful**: the return traffic of an allowed connection is allowed automatically.
- Default rules (can't be deleted, but you can override them with a lower number):

| Inbound | Outbound |
|---|---|
| 65000 `AllowVnetInBound` | 65000 `AllowVnetOutBound` |
| 65001 `AllowAzureLoadBalancerInBound` | 65001 `AllowInternetOutBound` |
| 65500 `DenyAllInBound` | 65500 `DenyAllOutBound` |

- Attach an NSG to a **subnet** and/or a **NIC**. One NSG per subnet or NIC, but one NSG can be reused. Same region as the resource.
- **!** NSG on **both** subnet and NIC → **both must allow** the traffic.
  - Inbound: **subnet** NSG first, then **NIC** NSG.
  - Outbound: **NIC** NSG first, then **subnet** NSG.
- Troubleshoot: *Effective security rules* on the NIC, and **IP flow verify** (Network Watcher).

## Augmented rules, service tags, ASGs

- One rule can have **several IPs or ranges** and **several ports**.
- **Service tags**: names for Azure IP ranges (`Internet`, `VirtualNetwork`, `AzureLoadBalancer`, `Storage`, `Sql`…). No IP list to maintain.
- **ASG** (Application Security Group): group NICs by role (`asg-web`, `asg-db`) and use the ASG as source or destination in rules.
  **!** All NICs in an ASG must be in the **same VNet**.

## Commands

```bash
# VNet and subnet
az network vnet create -g labo-rg1 -n vnet1 --address-prefix 10.10.0.0/16 \
  --subnet-name subnet1 --subnet-prefix 10.10.10.0/24

# Static private IP (on the NIC)
az network nic ip-config update -g labo-rg1 --nic-name vm0-nic -n ipconfig1 --private-ip-address 10.10.10.10

# Public IP on a NIC (for RDP)
az network public-ip create -g labo-rg1 -n vm0-ip --sku Standard
az network nic ip-config update -g labo-rg1 --nic-name vm0-nic -n ipconfig1 --public-ip-address vm0-ip
az network nic ip-config update -g labo-rg1 --nic-name vm0-nic -n ipconfig1 --remove publicIpAddress

# NAT Gateway on a subnet
az network public-ip create -g labo-rg1 -n nat-pip --sku Standard
az network nat gateway create -g labo-rg1 -n natgw1 --public-ip-addresses nat-pip
az network vnet subnet update -g labo-rg1 --vnet-name vnet1 -n subnet1 --nat-gateway natgw1

# NSG rule: RDP from my IP only
az network nsg rule create -g labo-rg1 --nsg-name nsg1 -n allow-rdp-me --priority 100 \
  --source-address-prefixes <my-ip> --destination-port-ranges 3389 --access Allow --protocol Tcp

# ASG: web traffic to the web servers
az network asg create -g labo-rg1 -n asg-web
az network nic ip-config update -g labo-rg1 --nic-name vm0-nic -n ipconfig1 --application-security-groups asg-web
az network nsg rule create -g labo-rg1 --nsg-name nsg1 -n allow-web --priority 200 \
  --source-address-prefixes Internet --destination-asgs asg-web --destination-port-ranges 80 443 --access Allow --protocol Tcp

# Check the rules that really apply to a NIC
az network nic list-effective-nsg -g labo-rg1 -n vm0-nic
```
