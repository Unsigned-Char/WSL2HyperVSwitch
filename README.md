# Recipe: Persistent Network Configuration in WSL 2 using Hyper-V Virtual Switch

## Problem Description
Connecting to services running in WSL 2 from external sources can be challenging due to the instances being on a different network. This guide offers a solution to replace the internal virtual switch of WSL 2 with an external version in Windows 20H2 (WSL 2.0) and configure it for better networking control.

## Solution Overview
This recipe uses a Hyper-V virtual switch to bridge the WSL 2 network, providing improved control and visibility of Windows' network adapters within Ubuntu. The configuration supports both dynamic and static IP addressing, eliminating the need for port forwarding and simplifying network setup.

### Steps
1. **Enable Hyper-V and Management PowerShell Features:**
   - Open PowerShell as administrator and run:
     ```powershell
     Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V
     Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V-Management-PowerShell
     ```

3. **Create an External Virtual Switch in Hyper-V:**
   - Open PowerShell as administrator and run:
     ```powershell
     New-VMSwitch -Name "External Switch" -NetAdapterName eth0
     ```

4. **Modify the WSL Configuration:**
   - Create or modify the `.wslconfig` file in your user profile directory (`$env:USERPROFILE/.wslconfig`) with the following content:
     ```plaintext
     [wsl2]
     networkingMode=bridged
     vmSwitch="External Switch"
     dhcp=true
     ipv6=true
     ```

5. **Enable systemd in the WSL Distribution:**
   - Edit the `/etc/wsl.conf` file in your WSL distribution and add the following lines:
     ```plaintext
     [boot]
     systemd=true
     [network]
     hostname = HOSTAGE
     generateResolvConf = false
     ```

6. **Configure Network Addressing:**
   - For dynamic address configuration, ensure the following is present in `/etc/systemd/network/10-eth0.network`:
     ```plaintext
     [Match]
     Name=eth0
     [Network]
     DHCP=yes
     ```
   - For static address configuration, use:
     ```plaintext
     [Match]
     Name=eth0
     [Network]
     Address=192.168.x.xx/24
     Gateway=192.168.x.x
     DNS=192.168.x.x
     ```

7. **Link systemd Resolv.conf:**
   - Create a symbolic link to link resolv.conf from systemd:
     ```bash
     ln -sfv /run/systemd/resolve/resolv.conf /etc/resolv.conf
     ```

8. **Verification:**
   - Restart the WSL 2 instance and verify the network configuration with:
     ```bash
     ip addr show eth0
     ```

