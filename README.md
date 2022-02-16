# dcinabox
This repository provides an Ansible playbook to automate the deployment of the Datacenter in a Box solution described in the [NSX-T Easy Adoption Design Guide Document](https://communities.vmware.com/t5/VMware-NSX-Documents/NSX-T-Easy-Adoption-Design-Guide-NSX-T-version-3-1/ta-p/2893937). Running the playbook requires a machine with Ansible, the [VMware supported NSX-T Ansible modules](https://github.com/vmware/ansible-for-nsxt) installed via an Ansible Galaxy, the python package pyVmomi, a text editor, and the git CLI tool. 

To make satisfying those requirements easier, we prepared a docker image that incorporates all the required components. The only requirement on the end user machine is to have docker installed and active.  Please follow the instructions specific to your operating system to install docker.

___

## DC in a Box deployment steps
1)	Deploy the NSX Manager OVA via the vSphere Client
2)	Check that the NSX Manager GUI is accessible and install the license
3)	Customize the user_defined_vars.yml file based on your environment
4)	Run the Ansible Playbook

Steps 1 and 2 should be straightforward. If required, you can consult the documentation here:

[NSX Manager Installation Guide](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/3.1/installation/GUID-FA0ABBBD-34D8-4DA9-882D-085E7E0D269E.html)

[Add an NSX License Key](https://docs.vmware.com/en/VMware-NSX-T-Data-Center/3.1/administration/GUID-8E665EAC-A44D-4FB3-B661-E00C467B2ED5.html#GUID-8E665EAC-A44D-4FB3-B661-E00C467B2ED5)

If you are looking to automate the deployment of NSX Manager and the license installation, check the nsx-manager-ova branch in this repository

### Step 3 - Customize the user_defined_vars.yml file

Run the container with the required software components

```
docker run -it lcamarda/ansible-for-nsxt-v3.1:v1.0
```
Once in the container shell, clone the repository with the playbook and variable files
```
root@45b44791e585:/# git clone https://github.com/vmware-nsx/dcinabox
```
Navigate to repository directory
```
root@45b44791e585:/# cd dcinabox/
```
Open the file with the environment specific variables and customize it based on the environment characteristics. (Use “i” to edit the file after it is open)
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
  cluster: Cluster-02a
  vds: SiteA-vDS-02

#The NSX Manager appliace must be deployed from OVA already before running the playbook, you need to provide the credentials set during the deplyment process
nsx:
  username: admin
  password: VMware1!VMware1!

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

### Step 4 - Run the Playbook
Run the following command, in around 20-30 minutes the DC in a Box will be ready for consumption
```
root@45b44791e585:/dcinabox# ansible-playbook dc_in_abox.yml
```
