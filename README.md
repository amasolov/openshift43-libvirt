# openshift43-libvirt

The goal is to deploy Openshift 4.3 'baremetal' nodes on one KVM machine. This is basically an Ansible version of the official installation guide for Openshift 4.3 on baremetal: https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/installing_on_bare_metal/index

This was tested on a Lenovo server with 128Gb RAM and 32 cores.

## Assumptions
- Compute and storage resources
  - enough RAM to run all masters and workers (3x8+3x16=72Gb RAM)
  - more details here: https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/installing_on_bare_metal/installing-on-bare-metal#installation-requirements-user-infra_installing-bare-metal
- DHCP is configured
  - based on dhcpd (https://www.isc.org/dhcp/)
  - hosted externally and reachable from the nodes and the hypervisor 
  - subnet and omapi key are configured
- DNS is configured 
  - according to the Openshift documentation (https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/installing_on_bare_metal/installing-on-bare-metal#installation-infrastructure-user-infra_installing-bare-metal)
- Ansible omapi_host module and Python3 is fixed
  - omapi_host module is not python3 compliant so it won't work out of the box. Waiting for this PR (https://github.com/ansible/ansible/pull/67560) to be merged. Feel free to patch your system manually.
  - if using omapi_host is not desirable then create dhcp leases manually.
- Red Hat Enterprise Linux 8 Installation DVD is present
  - this was tested on Red Hat Enterprise Linux 8.1 and it's designed around having the installation dvd file in /data/rhel-8.1-x86_64-dvd.iso on your hypervisor. It can be altered through the playbook variables.

## Configuration
1. Edit inventory and add your KVM server to the hypervisor group
```
# cat inventory
[hypervisor]
10.1.10.10 ansible_user=root ansible_ssh_common_args='-o StrictHostKeyChecking=no'
```
2. Review the playbook variables in `group_vars/all.yaml` and make sure it suits your environment. You need to generate MACs for your VMs, add your ssh key, your openshift cluster secret, etc.

## Deployment
Please be aware that the playbook will *DELETE* all VMs on the hypervisor. 

To start deployment: `ansible-playbook -i inventory site.yaml`

You might need to adjust delays based on your compute power. Please check virt-install commands in the playbooks if you have a different NIC name on your hypervisor. 

## User management
Configuring an HTPasswd identity provider is described here (https://access.redhat.com/documentation/en-us/openshift_container_platform/4.3/html/authentication/configuring-identity-providers#configuring-htpasswd-identity-provider) and can be performed after getting your cluster up and running.

## Accessing the console
The management console should be available upon succesfull installation: 

http://console-openshift-console.apps.CLUSTER_NAME.CLUSTER_DOMAIN
