# OPNSense Wireguard ProtonVPN Selective Routing Setup and Configuration

This guide works as of 13th November 2023 with OPNsense 23.7.8

This guide is mainly being used a reminder for myself when i inevitably decide to change something with my router config and also hopefully a helping hand for other users.

## Step 1: Install Updates, Plugins and Enable SSH

### Update OPNSense and Install Wireguard
1. Before we start anything we need to update our OPNSense.
2. Navigate to **`System > Firmware > Status > Updates`**
3. If updates are needed. Wait for them to download, install and possibly restart the system. Once that is finished we are good to continue
4. Navigate to **`System > Firmware > Plugins`**
5. Search for **`os-wireguard`** (choose os-wireguard not os-wireguard-go)
6. Press `+` to install

### Enable SSH
1. Navigate to **`System > Settings > Administration`**
2. Check **`Enable Secure Shell`** if it's not already checked. (You should disable Secure Shell once this guide has been completed)
3. SSH is already installed for Linux and Mac users.
4. If you are a windows user which I am not. Use either PuTTY or Windows PowerShell.

## Step 2: Proton Configuration
1. Access https://account.protonvpn.com/downloads
2. Go about half way down the page until you find **`WireGuard Configuration`**
3. Enter a name for the config file to be generated. (For this i decided to keep it simple and name it **`OPNSense`** but you can name it whatever you would like)
4. Select **`Router`** as your platform
5. Now the next part of selecting the VPN options is entirely your choice but I personally have the 'Block malware, ads, & trackers' option aswell as VPN Accelerator enabled as its just nice to have.
6. Hit create when you are happy with your config

The config file I will be referencing in this guide
```
[Interface]
# Key for Tutorial
# Bouncing = 4
# NetShield = 2
# Moderate NAT = off
# NAT-PMP (Port Forwarding) = off
# VPN Accelerator = on
PrivateKey = yMIf4bUinsvT7XMuDrA7EePIxvfeV0bH6BSnV2Zdg0A=
Address = 10.2.0.2/32
DNS = 10.2.0.1

[Peer]
# US-CA#323
PublicKey = hYKR1itwhCnthIPSMnQ3p38X3Fx54wkimogq/UJdzCY=
AllowedIPs = 0.0.0.0/0
Endpoint = 146.70.195.98:51820
```

We are nearly all set except we are missing the interface public key which we have to get ourselves from SSH

1. SSH into your OPNsense server with `ssh root@192.168.1.1`
2. Choose Option 8
3. Enter the following command: `echo <InterfacePrivateKey> | wg pubkey`

For example:
`echo yMIf4bUinsvT7XMuDrA7EePIxvfeV0bH6BSnV2Zdg0A= | wg pubkey`

4. The interface public key will then be displayed.

For example:
`Interface Public Key: oTI8o1lHl2ZwViGOJpCM+uRNdruCojcCHAAJEbZadV0=`

## Step 3. OPNsense Configuration

### Peer Configuration (Endpoint)

1. Navigate to **`VPN > WireGuard > Peers`**
2. Select the `+` to add a new endpoint
3. Enter your configuration like belows example:
```
Enabled:             *Checked*
Name:                <Something easy to recognise> eg. Proton_US_CA-233
Public Key:          <Peer Public Key> eg. hYKR1itwhCnthIPSMnQ3p38X3Fx54wkimogq/UJdzCY=
Shared Secret:       <Leave empty>
Allowed IPs:         0.0.0.0/0
Endpoint Address:    <Peer Endpoint IP> eg. 146.70.195.98
Endpoint Port:       51820
Instances:           <Leave empty for now>
Keepalive Interval:  25
```

#### Visual Guide:
<img width="600" alt="Peer Config Tutorial" src="https://github.com/MacvoidTech/OPNSense_WG/assets/144901250/46d84ec4-5052-4ab7-acbd-8b4017a371a0">

4. Click save
5. Click apply

### Instance Configuration (Local)

1. Navigate to **`VPN > WireGuard > Instance`**
2. Select the `+` to add a new instance connection
3. Check **`advanced mode`** in the top left
4. Enter your configuration like belows example:
```
Name:                <Something easy to recognise> eg. ProtonVPN
Public Key:          <Click the cog icon to generate a public and private keypair>
Private Key:         <Will be generated with the public key>
Listen Port:         <Choose a unique port> eg. 51820
MTU:                 <1412 or 1420> I chose 1412
DNS Server:          <Leave empty otherwise you will override your DNS config>
Tunnel Address:      <10.2.0.2/28 as setting it as /32 will cause some issues later>
Depend on (CARP)     <None>
Peers:               <Select the peer you just create> eg. Proton_US_CA-233
Disable Routes:      Checked
Gateway:             <Choose a unique IP address that is close to the tunnel> eg. 10.2.0.4
```
#### Visual Guide:
<img width="600" alt="Screenshot 2023-11-13 at 1 42 50 pm" src="https://github.com/MacvoidTech/OPNSense_WG/assets/144901250/50128db0-40c7-42db-8005-b98a5e201b66">

5. Click save.
6. Click apply.
7. Go to the OPNSense Dashboard and find the Services tab and restart your Wireguard service.
8. Navigate back to **`VPN > WireGuard > Instance`**
9. Click the edit pencil button on your ProtonVPN Instance
10. Replace your Public Key with the one you got from the SSH command **`oTI8o1lHl2ZwViGOJpCM+uRNdruCojcCHAAJEbZadV0=`**
11. Replace your Private Key with the one you got from the Proton Config **`yMIf4bUinsvT7XMuDrA7EePIxvfeV0bH6BSnV2Zdg0A=`**
12. Click save.
13. Click apply.
14. Go back to the OPNSense Dashboard and restart your Wireguard service again.

## Step 4. Interfaces

1. Navigate to **`Interfaces > Assignments`**
2. Create an interface with the Wireguard Tunnel you just created. eg. [ProtonWireguard]
3. Navigate to **`Interfaces > <NewInterfaceName>`**
4. Check **`Enable Interface`**
6. Don't change anything else
7. Save and then hit Apply.

## Step 5. Gateway

1. Navigate to **`System > Gateways > Single`**
2. Add a new Gateway.
3. Enter your configuration like belows example:
```
Name:                <Something distinctive and descriptive> eg. ProtonGateway
Description:         <Add one if you wish to>
Interface:           <Select the interface you created in the assignments tab>
Address Family:      IPv4
IP Address:          <Enter the Endpoint IP> eg.   146.70.195.98
Upstream Gateway:    Unchecked
Far Gateway:         Checked
Disable Gateway Monitoring: Unchecked
Monitor IP:          Blank
*Leave everything else blank*
```
4. Save and then hit Apply.

## Step 6. Aliases

1. Navigate to **`Firewall > Aliases`**
2. Click the `+` button to add a new alias.
3. Enter your configuration like belows example:
```
Enabled:             *Checked*
Name:                <Something distinctive and descriptive> eg. VPN_Access
Type:                <Hosts for Individual IPs or other Aliases. Networks for IP Ranges or VLANs>
Categories:          <Blank>
Content:             <Enter the host IPs, or the network in CIDR format>
Description:         <Add one if you wish to>
```
4. Click save and then hit Apply.
5. Click the `+` button to add another alias.
6. This time enter your configuration as follows:
```
Enabled:             *Checked*
Name:                RFC1918_Networks
Type:                Select Network(s) in the dropdown
Categories:          <Leave Empty>
Content:             192.168.0.0/16, 10.0.0.0/8, 172.16.0.0/12, <You may have to enter these individually as copy and paste may give you an error
Description:         All local (RFC1918) networks
```
7. Click save and then hit Apply.

## Step 7. NAT

1. Navigate to **`Firewall > NAT > Outbound`**
2. Select **`Hybrid Outbound NAT Rule Generation`**
3. Click save
4. Click apply
5. Click the `+` button to add a NAT Outbound Rule
6. Enter your configuration like belows example:
```
(If an option isn't mentioned, leave it untouched)
Interface:                <Select the interface for your WireGuard VPN>
TCP/IP Version:           <IPv4>
Protocol:                 <any>
Source invert:            *Unchecked*
Source Address:           <Select the Alias for the hosts/networks that are intended to use the tunnel> eg. VPN_Access
Source port:              <any>
Destination invert:       *Unchecked*
Destination address:      <any>
Destination port:         <any>
Translation / target:     <Interface Address>
Description:              <Add one if you wish to>
```
7. Click save and then Apply.

## Step 8. Firewall Rules

1. Navigate to **`Firewall > Rules > <Any Interface>`** eg. I have select Guest VLAN and LAN net clients routing through Proton
2. Click the `+` button to add a new firewall rule
3. Enter your configuration like belows example:
```
Action:                <Pass>
Quick:                 <Checked>
Interface:             <Whatever interface you are configuring the rule on>
Direction:             <in>
TCP/IP Version:        <IPv4>
Protocol:              <any>
Source / Invert:       *Unchecked*
Source:                <Select the relevant hosts Alias you created> eg. VPN_Access
Destination / Invert:  *Checked*
Destination:           <RFC1918_Networks>
Destination Port:      <any>
Description:           <Add one if you wish to>
Gateway:               <Select the gateway you created above> eg.ProtonGateway
Click Advanced Options Show/Hide
Set local tag:         NO_WAN_EGRESS
```
#### NO_WAN_EGRESS is a VPN Killswitch which stops traffic intended for the VPN from going out the normal WAN gateway if the tunnel gateway goes down

4. Move this rule above all other firewall rules as OPNSense Firewall Rules work from top to bottom
5. Click save and then Apply.

### NO_WAN_EGRESS Configuration

1. Navigate to **`Firewall > Rules > Floating`**
2. Click the `+` button to add a new firewall rule
3. Enter your configuration like belows example:
```
Action:                <Block>
Quick:                 *Checked*
Interface:             <WAN>
Direction:             <out>
TCP/IP Version:        <IPv4>
Protocol:              <any>
Source/Invert:         *Unchecked*
Source:                <any>
Destination/Invert     *Unchecked*
Destination:           <any>
Destination Port:      <any>
Description:           <Add one if you wish to>

Click Advanced Options Show/Hide

Match local tag:         <NO_WAN_EGRESS>
```
4. Move this rule above all other firewall rules
5. Click save and then Apply.

## References:
- https://docs.opnsense.org/manual/how-tos/wireguard-selective-routing.html
