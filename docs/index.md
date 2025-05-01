# Project Introduction/Goal

The purpose of this project is to completely automate the setup of my existing home network. This includes automating the setup of my OPNSense firewall/router and for configuring the Mikrotik Cloud Router Switches that are replacing my existing LAN Unifi stack.

For the OPNSense Firewall, the project's goal is to configure the following:

- Host name
- VLANs (themselves and their interface assignments)
- Firewall interfaces (related rules and their aliases)
- DHCP / DNS (for each VLAN)
- TailScale - Potentially migrate to local plugin rather than installing directly in OPNSense CLI 
- Plugins:
    - OS-GIT-BACKUP (for version-control based backups of the firewall)
    - OS-THEME-REBELLION (for dark UI theme)

For the Mikrotik switches, the project's goal is to configure the following:

- Host name
- Management IP address (on management-type VLAN)
- Trunk/access ports for relevant VLANs
- Basic RTSP parameters (so that the switch directly behind the firewall is the root bridge)