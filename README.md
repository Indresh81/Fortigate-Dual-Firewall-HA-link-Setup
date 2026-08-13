# FortiGate 300D Active-Passive HA Firewall Lab

A hands-on lab building a redundant firewall edge using two FortiGate 300D units in an Active-Passive HA cluster, fronted by a distribution switch and serving two VLANs through an access switch. Includes failover testing with continuous ping and traceroute.

## Topology

![Topology](images/firewall_topology_single_page.png)

```
ISP → Router (gateway) → 3560-TOP (distribution switch)
                              ├── Gi0/2 → Port1 → FortiGate FW1-HA
                              └── Gi0/3 → Port1 → FortiGate FW2-HA
FW1-HA Port2 → Gi0/1 ┐
FW2-HA Port2 → Gi0/2 ┴→ 3560-BOTTOM (access switch)
                              ├── Gi0/3 → PC (VLAN 10)
                              └── Gi0/4 → PC (VLAN 20)
```

## Lab Components

| Component        | Role                                   |
|-------------------|-----------------------------------------|
| Router            | Gateway to ISP                         |
| Cisco 3560 (TOP)  | Distribution switch, uplinks to both firewalls |
| FortiGate 300D x2 | Active-Passive HA cluster (`FW-HA-CLUSTER`) |
| Cisco 3560 (BOTTOM)| Access switch, VLAN 10 / VLAN 20 hosts |

## HA Configuration

- **Mode:** Active-Passive
- **Group name:** `FW-HA-CLUSTER`
- **FW1-HA:** device priority 200 (Primary)
- **FW2-HA:** device priority 100 (Secondary)
- **Monitor interfaces:** port1, port2
- **Heartbeat interface:** port8

## Interface / VLAN Configuration

| Interface | IP / Netmask         | Purpose                  |
|-----------|-----------------------|---------------------------|
| mgmt1     | 192.168.1.99/24        | Management                |
| port1     | 192.168.55.2/24        | WAN uplink (to 3560-TOP)  |
| VLAN10 (on port2) | 192.168.10.1/24 | LAN, DHCP 192.168.10.20–200 |
| VLAN20 (on port2) | 192.168.20.1/24 | LAN, DHCP 192.168.20.20–200 |

**DNS servers:** 8.8.8.8 (primary), 1.1.1.1 (secondary)

## Firewall Policies

| Name           | In → Out       | Source/Dest | NAT | Action |
|-----------------|-----------------|-------------|-----|--------|
| VLAN10-to-WAN   | VLAN10 → port1  | all/all     | Enabled | Accept |
| VLAN20-to-WAN   | VLAN20 → port1  | all/all     | Enabled | Accept |

Logging: Security Events, UTM inspection profile applied.

## Failover Testing

- Continuous `ping -t` to the VLAN20 gateway (192.168.20.1) and to 8.8.8.8 during a manual FW1 power-off/power-on, to observe HA failover convergence time.
- `tracert` to 8.8.8.8 and to an internal test host (172.18.2.1) to confirm routing through the cluster.
- HA LED indicators (front panel) checked pre/post failover — HA status LED changes from green (synced) to red during a role transition.
- HA monitor page (`System > HA`) confirmed both units **Synchronized**, with FW1-HA as Primary and FW2-HA as Secondary.

## Screenshots

| File | Description |
|------|--------------|
| `HA_1.png` | HA cluster monitor — both units synchronized |
| `ha_policy.png`, `ha_policy_1.png`, `ha_policy_2.png` | Firewall policy configuration and policy list |
| `ha_vlan10.png`, `ha_vlan20.png` | VLAN10/VLAN20 interface configuration |
| `ha_dns.png` | DNS server settings |
| `ha_int.png` | Physical/VLAN interface list |
| `fw1_2.png` | System settings (hostname, admin access) |
| `fw1_4.png` | HA cluster settings — FW1-HA (priority 200) |
| `fw2_2.png` | HA cluster settings — FW2-HA (priority 100) |
| `IMG_..._160925.jpg`, `IMG_..._164854.jpg` | Front-panel HA/status LEDs on the FortiGate units |
| `ping_before_-_fw1_power_off.png` | Ping to gateway before failover |
| `ping_after_-_fw1_power_on.png` | Ping to gateway after FW1 recovery |
| `ping_2.png` | Ping to 8.8.8.8 during failover |
| `tracert.png` | Traceroute to 8.8.8.8 and 172.18.2.1 |
| `test_after.png` | ipconfig on VLAN20 test PC |
| `vlan20_dhcp.png` | DHCP lease on VLAN20 client |
| `firewall_topology_single_page.png` | Full topology diagram |

## Key Takeaways

- Configured FortiGate Active-Passive HA with priority-based failover and dedicated heartbeat/monitor interfaces.
- Set up VLAN segmentation (VLAN10/VLAN20) with per-VLAN DHCP and outbound NAT policies to a shared WAN.
- Validated failover behavior with live ping/traceroute during a simulated primary-unit outage.
