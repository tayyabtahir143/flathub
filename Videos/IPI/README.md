# OpenShift IPI Cluster Setup on vSphere

This guide walks you through the complete process of deploying an OpenShift Container Platform (OCP) IPI (Installer-Provisioned Infrastructure) cluster on **VMware vSphere**.

---

## 🧰 Prerequisites


### vSphere Requirements
- vCenter 6.7 or newer
- ESXi hosts with shared storage and properly configured networking
- DNS (forward and reverse lookup) entries for:
  - API, & ingress
- DHCP service (for IPI to dynamically assign IPs)
- NTP properly configured and synchronized
- vSphere credentials with privileges to create/modify VMs, disks, and networks


IP Addressing and Temp DNS setup for dns server installation.
```bash
ip -br -c a




🌐 DNS & Load Balancer Configuration
Ensure these records exist (replace with your domain/cluster name):

Record Type	Name	Points to / Targets
A	api.ocp.example.com	IP for API
A	api-int.ocp.example.com	 IP for API
A	*.apps.ocp.example.com IP for ingress

We will be using dnsmasq as DNS and DHCP.
```bash
dnf install dnsmasq -y
> /etc/dnsmasq.conf
vim /etc/dnsmasq.conf
domain-needed
bogus-priv
server=8.8.8.8
server=8.8.4.4
listen-address=192.168.12.10
no-poll
addn-hosts=/etc/dnshost
domain=ipicluster.tayyabtahir.com
resolv-file=/etc/resolv.conf
address=/.apps.ipicluster.tayyabtahir.com/192.168.12.28    #Wildcard DNS Record




##DHCP Configuration
dhcp-range=ens33,192.168.12.2,192.168.12.30,255.255.255.0,4m
dhcp-option=option:router,192.168.12.1
interface=ens33
#dhcp-host=00:50:56:b3:a2:26,192.168.12.27

#dhcp-host=00:50:56:b3:e0:58,192.168.12.21
#dhcp-host=00:50:56:b3:81:04,192.168.12.22
#dhcp-host=00:50:56:b3:6a:d0,192.168.12.23

#dhcp-host=00:50:56:b3:f0:eb,192.168.12.24
#dhcp-host=00:50:56:b3:0e:f1,192.168.12.25


vim /etc/dnshost
192.168.12.29 api.ipicluster.tayyabtahir.com api
192.168.12.29 api-int.ipicluster.tayyabtahir.com api-int

systemctl restart dnsmasq
```

Once DNS has been installed and configured, change bastian host's dns ip to its own IP.
```bash
nmcli connection modify ens33 ipv4.dns 192.168.12.10
nmcli connection down ens33 && nmcli connection up ens33

cat /etc/resolv.conf
```




### Client Tools

⚙️ Step 1: Extract and Configure the Installer

- OpenShift Installer: [Download](https://console.redhat.com/openshift/install/vsphere/installer-provisioned)
- `oc` CLI
- Pull secret from [Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/vsphere/installer-provisioned)

```bash
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz
wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz

tar -zxvf openshift-install-linux.tar.gz
tar -zxvf openshift-client-linux.tar.gz
cp oc kubectl /usr/local/sbin/
cp oc kubectl /usr/local/bin/
cp openshift-install /usr/local/sbin/
cp openshift-install /usr/local/bin/

openshift-install version
oc version
```

---

## 📁 Directory Structure

Create a working directory for the installer:

```bash
mkdir -p ~/ipi/config
cd ~/ipi


🗂️ Step 2: Prepare Install-config.yaml file
```bash
openshift-install create install-config --dir=config
```
You will be prompted for:

Platform: vSphere

vCenter hostname or IP

vCenter credentials

Datacenter

Default datastore

Cluster and network

Base domain (e.g., example.com)

Cluster name (e.g., ocp)

Pull secret

This creates an install-config.yaml file in ocp-cluster/.


Open the install-config file and replace machineNetwork with your base network.

-from

machineNetwork:
  - cidr: 10.0.0.0/16

-to

machineNetwork:
  - cidr: 192.168.12.0/24


#Number5

🏗️ Step 3: Create the Cluster
```bash
openshift-install create cluster --dir=config
```
This process will:

Deploy the bootstrap VM

Create control plane and worker VMs

Wait for the bootstrap process to complete

Automatically approve CSRs and finish the install

It can take 30–60 minutes depending on resources.
#Number6

I will create a folder in vSphere cluster and start deploying the master and worker nodes there.
#Number7

Once cluster creation command completed, configure kubeconfig file from config directory, and verify the cluster.

#Number8

Once
✅ Step 4: Verify the Cluster
After completion:
```bash
export KUBECONFIG=config/auth/kubeconfig
oc get nodes
oc get co  # Check cluster operators
```
#Number9

🔐 Access the Web Console
Once installed:

Console: console-openshift-console.apps.ipicluster.tayyabtahir.com

Login using the credentials from:
```bash
cat ocp-cluster/auth/kubeadmin-password
```




🧹 Cleanup
To destroy the cluster:
```bash
openshift-install destroy cluster --dir=config
```