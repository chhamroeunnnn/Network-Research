# Method 1: Python + Netmiko Script
This Python script configures PAT (NAT Overload) on a target router by defining the inside/outside interfaces, setting up an ACL for allowed internal subnets, and enabling translation.
```Py
from netmiko import ConnectHandler

# 1. Router connection parameters
router = {
    "device_type": "cisco_ios",
    "host": "192.168.1.1",
    "username": "admin",
    "password": "SecurePassword123!",
    "secret": "EnablePassword123!",  # Privilege mode password
}

# 2. Commands to deploy PAT (NAT Overload)
nat_commands = [
    "! Define NAT Inside Interface",
    "interface GigabitEthernet0/0",
    "ip nat inside",
    "exit",
    "! Define NAT Outside Interface",
    "interface GigabitEthernet0/1",
    "ip nat outside",
    "exit",
    "! Define ACL for local LAN subnet",
    "ip access-list standard NAT_LAN",
    "permit 192.168.1.0 0.0.0.255",
    "exit",
    "! Bind ACL to Outside Interface with Overload",
    "ip nat inside source list NAT_LAN interface GigabitEthernet0/1 overload",
]


def deploy_nat():
    print(f"Connecting to router {router['host']}...")
    with ConnectHandler(**router) as net_connect:
        net_connect.enable()

        print("Sending NAT configuration commands...")
        output = net_connect.send_config_set(nat_commands)
        print("\n--- Execution Output ---")
        print(output)

        # Save configuration
        save_output = net_connect.save_config()
        print("\nConfiguration saved to startup-config.")


if __name__ == "__main__":
    deploy_nat()
```
---

# Method 2: Ansible Playbook

``` YAML
---
- name: Configure PAT (NAT Overload) on Cisco Routers
  hosts: routers
  gather_facts: no
  connection: network_cli

  tasks:
    - name: Configure NAT Interfaces and Rules
      cisco.ios.ios_config:
        lines:
          - ip access-list standard NAT_LAN
          - permit 192.168.1.0 0.0.0.255
          - ip nat inside source list NAT_LAN interface GigabitEthernet0/1 overload
        parents: []

    - name: Set GigabitEthernet0/0 as NAT Inside
      cisco.ios.ios_config:
        lines:
          - ip nat inside
        parents: interface GigabitEthernet0/0

    - name: Set GigabitEthernet0/1 as NAT Outside
      cisco.ios.ios_config:
        lines:
          - ip nat outside
        parents: interface GigabitEthernet0/1
```
---

### Key Automation Considerations for NAT
Order of Execution: NAT commands must be configured in a logical sequence. Interfaces must be flagged as inside or outside before active sessions begin routing.

ACL Dependencies: The ACL referenced by ip nat inside source list <ACL_NAME> must exist before or during the same configuration block, or traffic will fail to translate.

Pre-Checks & Verification: Always include automated verification steps (such as executing show ip nat translations or show ip nat statistics post-deployment) to verify that translations active counter increases.