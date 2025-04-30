# Initial Setup

While a home network wouldn't typically be considered priority or critical enough to warrant a need to take extra precautions for a project like this, I didn't want to anger the rest of the household by killing. This is especially the case as I tend to swap between projects and completion timelines aren't entirely set in stone.

With that in mind, I decided to develop this automation by first replicating it into a virtualized environment in GNS3. As I had originally setup a GNS3 server for use in CCNA studies, I thought it best to utilize it now to stand up a testing environment that would take the brunt of most of the damage I could potentially do.

Since we are integrating with tools in a virtual environment, a particular setup had to be devised in order to ease development while also ensuring that we could get the 
environment as close to production as humanely possible.

In the end, we settled with the following for the server and networking setup:

``` mermaid
flowchart LR

    %% Define Production Home Network Nodes

    PRD_OPNSense(["OPNSense Firewall/Router"])
    PRD_Switching(["L2 Network (w VLAN Switching)"])
    DEV_Workstation(["Dev Workstation<br/>(VSCode + SSH Plugin)"])

    %% Define Proxmox Server Nodes

    PVE_LXC(["Ubuntu 24.04 LXC Container<br/>(Ansible Control Node w/Git Repos)"])

    %% Define GNS3 Server Nodes

    VIRT_OPNSense(["OPNSense Virtual Appliance<br/>(Ansible Managed Node)"])
    VIRT_Switching(["Virtual L2 Network (w/ VLAN Switching)"])

    %% Define Internet Subgraph
    
    subgraph Internet
    end
    
    subgraph Production_Home_Network [PRODUCTION HOME NETWORK]
        subgraph PHN_Container[" "]
            direction LR
            
            PRD_OPNSense === PRD_Switching
            PRD_Switching === DEV_Workstation
            
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
                    VIRT_OPNSense === VIRT_Switching
                end

                GNS3_Server_Padding:::hidden
            end

            PRD_Switching === PVE_LXC
            PRD_Switching === VIRT_OPNSense
            PRD_Switching === VIRT_OPNSense
        end

        subgraph PHN_Padding[" "]
        end

        PHN_Container:::hidden ~~~ PHN_Padding:::hidden
    end

    Production_Home_Network:::PROD

    Internet === PRD_OPNSense

    %% Styles

    classDef hidden fill:#0000,stroke:#0000
```