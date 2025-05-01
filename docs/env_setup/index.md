# Initial Setup

While a home network wouldn't typically be considered priority or critical enough to warrant a need to take extra precautions for a project like this, I didn't want to anger the rest of the household by killing the Internet connection. This is especially the case as I tend to swap between projects and completion timelines aren't entirely set in stone.

With that in mind, I decided to develop this automation by first replicating it into a virtualized environment in GNS3. As I had originally setup a GNS3 server for use for my CCNA studies, I thought it best to utilize it now to stand up a testing environment for building my Ansible playbook(s).

Since we are integrating with tools in a virtual environment, a particular setup had to be devised in order to ease development while also ensuring that we could get the 
environment as close to production as humanely possible.

In the end, we settled with the following for the server and networking setup:

``` mermaid
flowchart TD

    %%%%%%%%%%%
    %% NODES %%
    %%%%%%%%%%%

    %% DEFINE TITLE NODES %%

    PHN_Block_Title@{ shape: doc, label: "PRODUCTION HOME NETWORK" }
    GNS_Block_Title@{ shape: doc, label: "GNS3 SERVER<br/>(Virtual Network Lab)" }
    Proxmox_Block_Title@{ shape: doc, label: "PROXMOX VE SERVER<br/>(VM/CTs)" }

    %% Define Production Home Network Nodes

    PRD_OPNSense["OPNSense Firewall/Router"]
    PRD_Switching["LAN(s)"]
    DEV_Workstation["Dev Workstation<br/>(VSCode)"]

    %% Define Proxmox Server Nodes

    PVE_LXC["Ubuntu 24.04 LXC Container<br/>(Ansible Control Node)"]

    %% Define GNS3 Server Nodes

    VIRT_OPNSense["OPNSense Virtual Appliance<br/>(Ansible Managed Node)"]
    VIRT_Switching["LAN(s)"]

    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    %% DEFINE SUBGRAPH STRUCTURES %%
    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

    %% Define Internet Subgraph
    
    subgraph Internet
    end
    
    %% Define Production Home Network Subgraph (and any nested/formatting-type subgraphs)
    
    %%{ init: { 'flowchart': { 'curve': 'stepBefore' } } }%%
    subgraph Production_Home_Network [" "]
            
        PHN_Block_Title

        PRD_OPNSense
        PRD_Switching
        DEV_Workstation
        
        %% Define Proxmox VE Server Block
        subgraph Proxmox_VE_Server [" "]
            direction TB
            PVE_LXC
            Proxmox_Block_Title
        end

        %% Define GNS3 Server Block
        subgraph GNS3_Server [" "]
            direction TB
            VIRT_OPNSense
            VIRT_Switching
            GNS_Block_Title
        end
    end

    %%%%%%%%%%%%%%%%%%%%%%
    %% Link Definitions %%
    %%%%%%%%%%%%%%%%%%%%%%

    %% Link ID: 0 (INTERNET TO OPNSENSE PROD FIREWALL)
    Internet === PRD_OPNSense

    %% Link ID: 1 (OPSENSE PROD FIREWALL TO PROD LAN)
    PRD_OPNSense === PRD_Switching

    %% Link ID: 2 and 3 (PROD LAN TO VIRTUAL OPNSENSE APPLIANCE)
    PRD_Switching === VIRT_OPNSense
    PRD_Switching === VIRT_OPNSense

    %% Link ID: 4 (VIRTUAL OPNSENSE APPLIANCE TO VIRTUAL LAN)
    VIRT_OPNSense === VIRT_Switching

    %% Link ID: 5 (PROD LAN TO DEVELOPMENT WORKSTATION)
    PRD_Switching === DEV_Workstation

    %% Link ID: 6 (PROD LAN TO PROXMOX LXC)
    PRD_Switching === PVE_LXC

    %% Link ID: 7, 8, 9 (TITLES)
    VIRT_Switching ~~~ GNS_Block_Title
    PVE_LXC ~~~ Proxmox_Block_Title

    %%%%%%%%%%%%
    %% Styles %%
    %%%%%%%%%%%%

    %% Hidden Nodes Style
    classDef hidden fill:#0000,stroke:#0000

    %% VLAN Trunk Link Style
    linkStyle 1,4 stroke-width:7px,stroke:blue

    %% OPNSense Virtual Appliance MGMT Link Style
    linkStyle 2,6 stroke-width:7px,stroke:red

    %% OPNSense Virtual Appliance WAN Link Style
    linkStyle 3 stroke-width:7px,stroke:green

    %% 'Other' Link Style
    linkStyle 0,5 stroke-width:7px

    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    %% Class-Based Style Applications %%
    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

    Production_Home_Network:::PROD
```

### VLANS / Connections

| COLOR | VLAN Name | VLAN ID |
| ----- | ----- | ----- |
| BLUE | VLAN Trunks (RoAS or Router-On-A-Stick) | N/A |
| GREEN | OPNSense Virtual Appliance WAN | 200 |
| RED | OPNSense Virtual Appliance MGMT | 210 |
| BLACK | OTHER | N/A |

### Network Format (VLANS)

| NETWORK | GATEWAY/DNS ADDRESSES | DHCP RANGE |
| ----- | ---- | ----- |
| 172.99.[VLAN ID].200/24 | .254 | .100-.199 |

### Device IP Addresses

| LEASING SCHEME | IP ADDRESS | DEVICE |
| ----- | ----- | ----- |
| STATIC | 172.99.210.200/24 | Proxmox VE Ubuntu LXC Interface |
| STATIC | 172.99.210.201/24 | OPNSense Virtual Appliance MGMT Interface |
| DHCP | DYNAMIC | OPNSense Virtual Appliance WAN Interface

The original production network consisted of an OPNSense firewall in an RoAS configuration with multiple VLAN-trunked Unifi switches providing access to end devices and servers in the household.

In starting this project, the idea was to automate the setup of that production firewall by connecting downstream of its default LAN interface and running the playbook 
against the interface IP address (192.168.1.1/24). However, if I tried to replicate that same configuration in GNS3, I would've had to resort to using a GNS3 appliance to develop my playbooks(s). While I could've looked at some configuration for passing the virtual environment's LAN back to the production network (by providing a physical interface out to the network), I wanted to avoid that looped configuration due to the possibility of the virtual LAN's network segment overlapping with the production's network (as it's the default LAN is in use in production for management of the current Unifi stack).

With that in mind, I used a CLI Debian-based appliance within GNS3 for development. However, I got tired of it pretty quickly due to the headaches involved in CLI navigation and looked to using Visual Studio Code as I was quite comfortable with it from my time working on NGF Records. However, at the same time, I didn't feel that it was good to bloat that appliance with a GUI for development when I was using an old gaming PC as my workstation locally. Instead, I looked to having a separate management interface setup (with minimal configuration) on the OPNSense appliance and having that interface connected to a production VLAN (ID: 210) that my development workstation could access (from it's place on another production VLAN not named here).

That setup looked good so far but I realized that I couldn't use my Windows 11-based workstation as a control node (it was feasible with WSL or Windows Subsystem for Linux but I didn't want to use a setup that Ansible designated as unsupported). At that point, I got around it by setting up an Ubuntu LXC container on my Proxmox hypervisor (configured as a part of my homelab) and connected it to the same management VLAN. I jumped at this setup as I had realized from past experience that Visual Studio Code could use SSH to remote into another environment for development. At the same time, the isolated nature of using a container for development made it an optimal choice for acting as the Ansible control node (and host for both the code and documentation repositories). In the end, I simplified (and secured) the setup even further by making use of SSH agent forwarding to keep SSH keys local to my development workstation.

I also introducted another production VLAN (ID: 200) and address range that I could use for the GNS3 OPNSense appliance WAN. As we were going to be building automations against a virtual firewall whose WAN address was handed down by a production VLAN through DHCP by a bare-metal firewall, I wanted to avoid the possibility of configuring an existing VLAN whose network segment overlapped with the address used by the virtual appliance's WAN connection (my environment used 10.19.50.0/24 originally for as an experimentation VLAN in production and that would've conflicted with automation development if we needed to configure that on one interface while the WAN was also using an address from it.)