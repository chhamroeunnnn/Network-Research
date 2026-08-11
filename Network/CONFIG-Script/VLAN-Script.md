# 1. Script to Create VLAN
Python script using netmiko that creates 5 VLANs on your switch and provisions access ports for 3 end devices per VLAN (15 active ports total).

### Network Layout Managed by the Script:
```
VLAN 10 (HR): Ports Gi0/1, Gi0/2, Gi0/3

VLAN 20 (FINANCE): Ports Gi0/4, Gi0/5, Gi0/6

VLAN 30 (IT): Ports Gi0/7, Gi0/8, Gi0/9

VLAN 40 (GUEST): Ports Gi0/10, Gi0/11, Gi0/12

VLAN 50 (IOT): Ports Gi0/13, Gi0/14, Gi0/15
```
---
```Python
from netmiko import ConnectHandler

# Target Switch Connection Details
switch_device = {
    'device_type': 'cisco_ios',
    'host': '192.168.1.50',
    'username': 'admin',
    'password': 'Cisco123',
}

# Define the 5 VLANs and their assigned interfaces (3 devices/ports each)
vlan_mapping = [
    {
        'vlan_id': 10,
        'name': 'HR',
        'ports': ['GigabitEthernet0/1', 'GigabitEthernet0/2', 'GigabitEthernet0/3']
    },
    {
        'vlan_id': 20,
        'name': 'FINANCE',
        'ports': ['GigabitEthernet0/4', 'GigabitEthernet0/5', 'GigabitEthernet0/6']
    },
    {
        'vlan_id': 30,
        'name': 'IT',
        'ports': ['GigabitEthernet0/7', 'GigabitEthernet0/8', 'GigabitEthernet0/9']
    },
    {
        'vlan_id': 40,
        'name': 'GUEST',
        'ports': ['GigabitEthernet0/10', 'GigabitEthernet0/11', 'GigabitEthernet0/12']
    },
    {
        'vlan_id': 50,
        'name': 'IOT',
        'ports': ['GigabitEthernet0/13', 'GigabitEthernet0/14', 'GigabitEthernet0/15']
    }
]

# Build the configuration command list dynamically
config_commands = []

for item in vlan_mapping:
    v_id = item['vlan_id']
    v_name = item['name']
    
    # 1. Create VLAN and Assign Name
    config_commands.append(f'vlan {v_id}')
    config_commands.append(f'name {v_name}')
    config_commands.append('state active')
    config_commands.append('exit')
    
    # 2. Assign the 3 ports to this VLAN
    for port in item['ports']:
        config_commands.append(f'interface {port}')
        config_commands.append(f'description Device on VLAN {v_id} ({v_name})')
        config_commands.append('switchport mode access')
        config_commands.append(f'switchport access vlan {v_id}')
        config_commands.append('no shutdown')
        config_commands.append('exit')

print("Connecting to switch to push 5 VLANs and 15 interface assignments...")

try:
    net_connect = ConnectHandler(**switch_device)
    
    # Push all generated commands in one execution block
    output = net_connect.send_config_set(config_commands)
    print("\n--- Command Output ---")
    print(output)
    
    # Save configuration to startup-config
    net_connect.save_config()
    print("\nSuccessfully configured 5 VLANs with 3 devices each!")
    
    net_connect.disconnect()

except Exception as e:
    print(f"Failed to push configuration: {e}")

```
---
# Terminal Output Preview
When you execute this script, the switch automatically processes the commands sequentially:

```Plantext
Connecting to switch to push 5 VLANs and 15 interface assignments...

--- Command Output ---
config term
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#vlan 10
Switch(config-vlan)#name HR
Switch(config-vlan)#state active
Switch(config-vlan)#exit
Switch(config)#interface GigabitEthernet0/1
Switch(config-if)#description Device on VLAN 10 (HR)
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 10
Switch(config-if)#no shutdown
Switch(config-if)#exit
... [output omitted for brevity] ...
Switch(config)#interface GigabitEthernet0/15
Switch(config-if)#description Device on VLAN 50 (IOT)
Switch(config-if)#switchport mode access
Switch(config-if)#switchport access vlan 50
Switch(config-if)#no shutdown
Switch(config-if)#exit
Switch(config)#end
Switch#

Successfully configured 5 VLANs with 3 devices each!
```
---
# Verification Command
Log into your switch CLI and type show vlan brief to verify the setup:
```Plantext
Switch# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Gi0/0, Gi0/16, Gi0/17...
10   HR                               active    Gi0/1, Gi0/2, Gi0/3
20   FINANCE                          active    Gi0/4, Gi0/5, Gi0/6
30   IT                               active    Gi0/7, Gi0/8, Gi0/9
40   GUEST                            active    Gi0/10, Gi0/11, Gi0/12
50   IOT                              active    Gi0/13, Gi0/14, Gi0/15
```
---
# To add another port:
| Scenario | Best Approach |
| :--- | :--- |
| **Emergency fix or 1 port change** | **Terminal (CLI)** — Faster and requires no code changes. |
| **Applying changes to 5+ ports/switches** | **Python Script** — Saves time and eliminates typos. |
| **Network Infrastructure as Code (IaC)** | **Update Source Script/Inventory** — Keeps your automation state aligned with reality. |
---
