# Initial Setup

While a home network wouldn't typically be considered priority or critical enough to warrant a need to take extra precautions for a project like this, I didn't want to anger the rest of the household by killing. This is especially the case as I tend to swap between projects and completion timelines aren't entirely set in stone.

With that in mind, I decided to develop this automation by first replicating it into a virtualized environment in GNS3. As I had originally setup a GNS3 server for use in CCNA studies, I thought it best to utilize it now to stand up a testing environment that would take the brunt of most of the damage I could potentially do.

Since we are integrating with tools in a virtual environment, a particular setup had to be devised in order to ease development while also ensuring that we could get the 
environment as close to production as humanely possible.

In the end, we settled with the following for the server and networking setup:

``` mermaid
flowchart LR

    %%%%%%%%%%%
    %% NODES %%
    %%%%%%%%%%%

    %% Define Production Home Network Nodes

    PRD_OPNSense(["OPNSense Firewall/Router"])
    PRD_Switching(["L2 Network (w VLAN Switching)"])
    DEV_Workstation(["Dev Workstation<br/>(VSCode + SSH Plugin)"])

    %% Define Proxmox Server Nodes

    PVE_LXC(["Ubuntu 24.04 LXC Container<br/>(Ansible Control Node w/Git Repos)"])

    %% Define GNS3 Server Nodes

    VIRT_OPNSense(["OPNSense Virtual Appliance<br/>(Ansible Managed Node)"])
    VIRT_Switching(["Virtual L2 Network (w/ VLAN Switching)"])

    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
    %% DEFINE SUBGRAPH STRUCTURES %%
    %%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

    %% Define Internet Subgraph
    
    subgraph Internet
    end
    
    %% Define Production Home Network Subgraph (and any nested/formatting-type subgraphs)
    subgraph Production_Home_Network [PRODUCTION HOME NETWORK]
        subgraph PHN_Container[" "]
            direction LR
            
            PRD_OPNSense
            PRD_Switching
            DEV_Workstation
            
            %% Define Proxmox VE Server Block
            subgraph Proxmox_VE_Server ["PROXMOX VE SERVER<br/>(VM/CTs)"]
                subgraph Proxmox_VE_Server_Padding[" "]
                    PVE_LXC
                end
                Proxmox_VE_Server_Padding:::hidden
            end

            %% Define GNS3 Server Block
            subgraph GNS3_Server ["GNS3 SERVER<br/>(Virtual Network Lab)"]
                subgraph GNS3_Server_Padding[" "]
                    VIRT_OPNSense
                    VIRT_Switching
                end

                GNS3_Server_Padding:::hidden
            end
        end

        subgraph PHN_Padding[" "]
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

    %% Link ID: 7 (HIDDEN LINK FOR Production Home Network block formatting)
    PHN_Container:::hidden ~~~ PHN_Padding:::hidden

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

| COLOR | VLAN Name | VLAN ID |
| ------ | ------------ | --- |
| BLUE | VLAN Trunks (RoAS or Router-On-A-Stick) | N/A |
| GREEN | OPNSense Virtual Appliance WAN | 200 |
| RED | OPNSense Virtual Appliance MGMT | 210 |
| BLACK | OTHER | N/A |