# Example Script

## 1. Security & Access Control (Port Security & ACLs)

### Access Control Lists (ACLs):
 Push firewall or traffic-filtering rules across switches (e.g., blocking a compromised IP address network-wide in seconds).

### Port Security:
 Automatically enable MAC address sticky binding or standard dot1x (802.1X) authentication settings on all user-facing access ports.

### 802.1X Dynamic Assignment: 
Configure switch ports to automatically assign incoming laptops to specific VLANs based on their logged-in user credentials.

# Prerequisites & Lab Setup
Before running the script, make sure:
```
Python and Netmiko are installed (pip install netmiko).

Your computer has network access to the switch (e.g., in GNS3, EVE-NG, PNET Lab, or a physical rack).

The target switch has SSH enabled and an admin account configured.
```
### Step 1: 
Write the Python Script (security_config.py)
Create a script named security_config.py that pushes a standard security ACL (blocking an unauthorized IP range) and configures port security on an access port.

```Python
from netmiko import ConnectHandler

# 1. Target Switch Details
switch_device = {
    'device_type': 'cisco_ios',
    'host': '192.168.1.50',
    'username': 'admin',
    'password': 'Cisco123',
}

# 2. Security Commands:
# - Define an ACL blocking access from 10.0.0.0/24 while allowing all other traffic
# - Enable Port Security on interface GigabitEthernet0/1 to restrict MAC addresses
security_commands = [
    # --- Define Access Control List (ACL) ---
    'ip access-list standard BLOCK_GUEST_SUBNET',
    'deny 10.0.0.0 0.0.0.255',
    'permit any',
    'exit',
    
    # --- Configure Port Security ---
    'interface GigabitEthernet0/1',
    'description Secured User Access Port',
    'switchport mode access',
    'switchport port-security',
    'switchport port-security maximum 2',  # Allow up to 2 MAC addresses on this port
    'switchport port-security violation restrict', # Drop unauthorized traffic & log alert
    'switchport port-security mac-address sticky',  # Lock in connected MAC addresses automatically
    'ip access-group BLOCK_GUEST_SUBNET in',       # Apply the ACL to inbound traffic
    'no shutdown'
]

print("Connecting to the switch...")

try:
    # 3. Establish SSH connection
    net_connect = ConnectHandler(**switch_device)
    print("Connection established. Applying security baseline...")

    # 4. Push configuration commands
    output = net_connect.send_config_set(security_commands)
    
    print("\n--- Device Output ---")
    print(output)

    # 5. Save the configuration
    net_connect.save_config()
    print("\nSecurity baseline applied and configuration saved successfully!")

    # 6. Disconnect
    net_connect.disconnect()

except Exception as e:
    print(f"Failed to apply security configuration: {e}")
```
# Step 2: 
## How to Run the Script:

Open your terminal (Command Prompt/PowerShell on Windows, Terminal on Mac/Linux, or VS Code Terminal).

Change directory to where your script is saved:
```Bash
cd path/to/your/script
```
Then Run Python file 
```Bash
python security_config.py
```
# Step 3: 
## Terminal Output
When executed successfully against the target switch, the output looks like this:
```Plaintext
Connecting to the switch...
Connection established. Applying security baseline...

--- Device Output ---
config term
Enter configuration commands, one per line.  End with CNTL/Z.
Switch(config)#ip access-list standard BLOCK_GUEST_SUBNET
Switch(config-std-nacl)#deny 10.0.0.0 0.0.0.255
Switch(config-std-nacl)#permit any
Switch(config-std-nacl)#exit
Switch(config)#interface GigabitEthernet0/1
Switch(config-if)#description Secured User Access Port
Switch(config-if)#switchport mode access
Switch(config-if)#switchport port-security
Switch(config-if)#switchport port-security maximum 2
Switch(config-if)#switchport port-security violation restrict
Switch(config-if)#switchport port-security mac-address sticky
Switch(config-if)#ip access-group BLOCK_GUEST_SUBNET in
Switch(config-if)#no shutdown
Switch(config-if)#end
Switch#

Security baseline applied and configuration saved successfully!
```
# Step 4: 
## Verification (Checking the Result)
You can verify the security configuration on the switch CLI by running:

>"Verify the ACL: show ip access-lists BLOCK_GUEST_SUBNET"

>"Verify Port Security: show port-security interface GigabitEthernet0/1"

# Note
If you want to bloack more IP just look at this example:
```Py
security_commands = [
    # --- Define Access Control List (ACL) ---
    'ip access-list standard BLOCK_GUEST_SUBNET',
    
    # 1. Block original Guest subnet (10.0.0.0/24)
    'deny 10.0.0.0 0.0.0.255',
    
    # 2. Block IoT / Smart Devices subnet (172.16.50.0/24)
    'deny 172.16.50.0 0.0.0.255',
    
    # 3. Block Contractor / Vendor subnet (192.168.99.0/24)
    'deny 192.168.99.0 0.0.0.255',
    
    # 4. Allow all remaining internal and authorized traffic
    'permit any',
    'exit',
    
    # --- Configure Port Security ---
    'interface GigabitEthernet0/1',
    'description Secured User Access Port',
    'switchport mode access',
    'switchport port-security',
    'switchport port-security maximum 2',
    'switchport port-security violation restrict',
    'switchport port-security mac-address sticky',
    'ip access-group BLOCK_GUEST_SUBNET in',
    'no shutdown'
]
```
---
# 2. DHCP Configuartion Script

network switches/routers, you use Python to push the DHCP Pool configuration. This tells the switch which range of IP addresses to hand out to end devices automatically when they plug in:

```Python
from netmiko import ConnectHandler

switch_device = {
    'device_type': 'cisco_ios',
    'host': '192.168.1.50',
    'username': 'admin',
    'password': 'Cisco123',
}

# Configuration commands to set up a DHCP server on the switch
dhcp_commands = [
    # Exclude management/gateway IPs from being handed out
    'ip dhcp excluded-address 10.0.0.1 10.0.0.10',
    
    # Create the DHCP Pool for Guest devices
    'ip dhcp pool GUEST_POOL',
    'network 10.0.0.0 255.255.255.0',       # Subnet range (10.0.0.1 to 10.0.0.254)
    'default-router 10.0.0.1',               # Default Gateway
    'dns-server 8.8.8.8 8.8.4.4',            # DNS Servers
    'lease 1 0 0',                           # Lease duration: 1 day
    'exit'
]

net_connect = ConnectHandler(**switch_device)
net_connect.send_config_set(dhcp_commands)
net_connect.save_config()
net_connect.disconnect()
```
## Note For Black by DHCP IP With ACL
We can black end device by MAC Address.
### Example:
```Py
# Block a specific rogue MAC address on a port
mac_block_commands = [
    'mac access-list extended BLOCK_ROGUE_DEVICE',
    'deny host a4b1.c102.f310 any',   # Drops traffic from this specific hardware MAC
    'permit any any',
    'exit',
    'interface GigabitEthernet0/1',
    'mac access-group BLOCK_ROGUE_DEVICE in'
]
```