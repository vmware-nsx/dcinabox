# dcinabox
This repository provides an Ansible playbook to automate the deployment of the Datacenter in a Box solution described in the NSX-T Simplified Reference Architecture Document. Running the playbook requires a machine with Ansible, the [VMware supported NSX-T Ansible modules](https://github.com/vmware/ansible-for-nsxt) installed via an Ansible Galaxy, the python package pyVmomi, the VMware ovftool, a text editor, and the git CLI tool. 

To make satisfying those requirements easier, we prepared a docker image that incorporates all the required components. The only requirement on the end user machine is to have docker installed and active.  Please follow the instructions specific to your operating system to install docker.

___

## DC in a Box deployment steps
1)	Run the container packaging all the required software and mount the NSX OVA Image locally
2)	Customize the user_defined_vars.yml file based on your environment
3)	Run the Ansible Playbook

### Step 1 - Run the docker container
You need to download the NSX OVA file to the system where you run docker. Take a note of the absolute path of the directory where the OVA file is located.

Run the container via this command:
```
docker run -v /home/luca/Downloads/:/ova -it  lcamarda/ansible-for-nsxt-v3.1:v1.1
```
You need to enter the path to the OVA file folder, in my case: /home/luca/Downloads/. That folder and its content will be presented to the container in /ova

### Step 2 - Customize the user_defined_vars.yml file

Once in the container shell, clone the repository with the playbook and variable files
```
root@45b44791e585:/# git clone https://github.com/vmware-nsx/dcinabox
```
Navigate to repository directory
```
root@45b44791e585:/# cd dcinabox/
```
Switch to the nsx-manager-ova branch
```
root@45b44791e585:/dcinabox# git checkout nsx-manager-ova
```
Open the file with the environment specific variables and customize it based on the environment characteristics. (Use “i” to edit the file after it opens)
```
root@45b44791e585:/dcinabox# vi user_defined_vars.yml
```
The file will look like this:
```
#This section cover the existing vCenter deployment.
vcenter:
  fqdn: vcsa-01a.corp.local
  ip: 192.168.110.22
  username: administrator@vsphere.local
  password: VMware1!
  datacenter: Site-A
  cluster: Cluster-02a
  vds: SiteA-vDS-02

nsx:
  username: admin
  password: VMware1!VMware1!
  datacenter: Site-A
  datastore: esx-03a-local
  dns_server: 192.168.110.10
  dns_domain: corp.local
  ntp_server: 192.168.110.10
  dns_domain: corp.local
  ova_file: nsx-unified-appliance-3.1.3.3.0.18844962.ova
  license: XXXXXXXXXXXXXXX


#In this section provide the VLAN ID for the 3 networks.
#The uplink VLAN represent the transit network between the physical switches and the NSX Gateway
#The overlay network represents the network where the TEP interfaces reside
#The management network host the management appliances. i.e., vCenter, NSX Manager, Edge Nodes Management Interfaces....
uplink_vlan_id: '254'
overlay_vlan_id: '130'
management_vlan_id: '100'

#In this section provide the details about the uplink network. The uplink network must be a /28 as a minimum.
uplink_network:
  snat_ip: 192.168.254.6
  vip_ip: 192.168.254.5
  edge01_ip: 192.168.254.1
  edge02_ip: 192.168.254.2
  gateway_ip: 192.168.254.3
  prefix_length: 24

#In this section provid the details about the mananagement network.
management_network:
  netmask: 255.255.255.0
  prefix_length: 24
  gateway_ip: 192.168.110.1
  nsx_manager_ip: 192.168.110.15
  edge01_ip: 192.168.110.19
  edge02_ip: 192.168.110.20
  portgroup_name: SiteA-vDS-02-Mgmt
  nsxgateway_serviceinterface_present: False #When connecting the DC in a Box to an Untrusted external network (i.e., the Internet set this to True)
  nsxgateway_serviceinterface_ip: 192.168.100.1 #An IP for the service interface on the management network is only required if "nsxgateway_serviceinterface_present" is set to "True"

#provide IP Range for the allocation of the TEP interfaces IP
tep_range:
  start: 192.168.130.51
  end: 192.168.130.91
  cidr: 192.168.130.0/24

#Provide Edge Node VMs deployment information
edge_nodes:
  edge1_datastore: nfs-02
  edge1_host: esx-03a.corp.local
  edge2_datastore: nfs-02
  edge2_host: esx-04a.corp.local
  password: VMware1!VMware1!
```
Save and close the file (:wq!)

### Step 3 Run the Playbook
Run the following command, in around 30-40 minutes the DC in a Box will be ready for consumption
```
root@45b44791e585:/dcinabox# ansible-playbook dc_in_abox.yml
```
