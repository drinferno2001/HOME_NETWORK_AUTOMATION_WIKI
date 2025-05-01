# Ansible Managed Node Setup (OPNSense)

To connect the Ansible control node (LXC) to the GNS3 OPNSense virtual appliance, the following steps were taken:

## Control Node 

### Generate SSH Keys 

SSH Keys were generated through the CLI locally using the following command: 

`ssh-keygen -t ed25519`

**Note:** *\*The default key name, file location, and passphrase defaults were left as is.\**

A public/private key pair (**id_ed25519/id_ed25519.pub**) was created under the **.ssh** directory for the logged-in user.

## OPNSense

### Upload SSH Public Key

Once logged onto the OPNSense target node, we navigated to **System > Access > Users > Edit** (for **root** user)

Added public key (copy content from **id_ed25519.pub** file) from control node to **Authorized_Keys** field.

### Enable SSH / Root Login

Afterward, navigated to **System > Settings > Administration > Secure Shell** and checked the following options:
  
- Secure Shell Server (Enable Secure Shell)
- Root Login (Permit root user login)

### Configure Management Interface

In GNS3, the [LAN] and [WAN] interfaces should've been auto-configured to use the physical vtnet0 and vtnet1 interfaces. The next physical interface [vtnet2]
was used for management access and was configured using the following steps:

- Added as a management interface by navigating to **Interfaces > Assignments > Assign a new interface**
    - Added using the name **ANSIBLE_MGMT**.
- Under **Interfaces > [ANSIBLE_MGMT]**, the following options were checked:
    - Enable (Enable Interface)
    - Lock (Prevent Interface Removal)
    - Block bogon networks
- Under the same menu, a static IPV4 configuration was selected and provided with the following: **172.99.210.201/24**
- Under **Firewall > Rules > ANSIBLE_MGMT**, a firewall rule was created with the following additional criteria:
    - Source - **ANSIBLE_MGMT net**
    - Destination - **ANSIBLE_MGMT address**
    - Description - **ALLOW ALL TRAFFIC**