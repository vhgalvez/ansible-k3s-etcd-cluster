# Ansible K3s ETCD Cluster

This project, `ansible-k3s-etcd-cluster`, provides Ansible playbooks and configurations to deploy a high-availability Kubernetes (K3s) cluster with embedded etcd across multiple nodes. Using this repository, you can set up master and worker nodes, automate node configurations, and simplify management tasks for your K3s cluster.

## Features
- **Automated K3s Deployment**: Quickly deploy K3s across multiple nodes.
- **Embedded ETCD**: Provides a resilient datastore for high availability.
- **Role-Based Node Configuration**: Assign nodes as masters or workers easily.
- **Systemd Service Creation**: Ensures K3s runs on boot and is managed by systemd.
- **Flatcar Linux Compatibility**: Compatible with Flatcar Container Linux and similar immutable OS setups.
- **SSH Key-Based Access**: Secures node access and connections using SSH keys.

## Cluster Overview

The configuration supports a multi-master, multi-worker setup with optional load balancing and additional infrastructure nodes (Redis, PostgreSQL, etc.). Below is an example of a network layout that this repository can help configure:

Public IP (HTTPS) | v Bastion Node (SSH Access) | v Load Balancer (Traefik) | v +----------------+---------------+-----------------+ | | | | Master Nodes Worker Nodes Infra Nodes

markdown
Copiar código

## Inventory Structure

The inventory file `inventory.ini` allows you to configure:
- **Master Nodes**: Hosts the etcd datastore and controls the cluster.
- **Worker Nodes**: Runs applications and workloads.
- **Load Balancer**: (Optional) Balances traffic to the cluster.
- **Database & Cache**: Additional infrastructure nodes as needed.

Example `inventory.ini`:
```ini
[nodos]
10.17.4.21 ansible_user=core ansible_ssh_private_key_file=/path/to/key ansible_port=22
# Add more nodes here...

[masters]
10.17.4.27 ansible_user=core ansible_ssh_private_key_file=/root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift ansible_port=22

[masters]
10.17.4.21 ansible_user=core ansible_ssh_private_key_file=/root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift ansible_port=22
10.17.4.22 ansible_user=core ansible_ssh_private_key_file=/root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift ansible_port=22
10.17.4.23 ansible_user=core ansible_ssh_private_key_file=/root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift ansible_port=22

[workers]
10.17.4.24 ansible_user=core ansible_ssh_private_key_file=/root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift ansible_port=22
10.17.4.25 ansible_user=core ansible_ssh_private_key_file=/root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift ansible_port=22
10.17.4.26 ansible_user=core ansible_ssh_private_key_file=/root/.ssh/cluster_openshift/key_cluster_openshift/id_rsa_key_cluster_openshift ansible_port=22
```

Prerequisites
Ansible installed on your control machine.
SSH access to all nodes with keys configured as per inventory.
Internet access for downloading K3s binaries on each node.
Usage
Clone the repository:

bash
Copiar código
git clone https://github.com/yourusername/ansible-k3s-etcd-cluster.git
cd ansible-k3s-etcd-cluster
Update Inventory: Edit inventory.ini with the IP addresses and access details for your nodes.

Run Playbook: Deploy K3s and configure the cluster with:

bash
Copiar código
ansible-playbook -i inventory.ini install_k3s.yaml
Playbook Structure
The main playbook, install_k3s.yaml, includes tasks for:

Installing K3s on all nodes.
Creating systemd services to manage K3s on master and worker nodes.
Joining worker and additional master nodes to the cluster.
Customizing Templates
Service templates for systemd are located in the templates directory. You can customize:

k3s_master.service.j2 for primary master configuration.
k3s_master_join.service.j2 for joining additional masters.
k3s_agent.service.j2 for worker nodes.
Verifying the Cluster
To check the cluster status:

bash
Copiar código
kubectl get nodes
Ensure each node is in the Ready state.

Troubleshooting
Node Connection Issues: Verify SSH access for each node as configured in inventory.ini.
Permissions Issues with kubectl: Ensure kubeconfig permissions are correctly set.
For any further issues, please create an issue on GitHub.

Contributing
Contributions are welcome! Please submit a pull request or open an issue if you encounter any bugs or have suggestions for improvements.

License
This project is licensed under the MIT License.

# Contacto

Para cualquier duda o problema, por favor abra un issue en el repositorio o contacte al mantenedor del proyecto.

**Mantenedor del Proyecto:** [Victor Galvez](https://github.com/vhgalvez)