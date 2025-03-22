# Conclave-Ansible

## Overview

Conclave-Ansible is an Ansible-based automation project designed to configure and manage a cluster environment. This project includes roles for setting up various services such as Docker, Prometheus, Grafana, Slurm, and more.

## Table of Contents

- [Overview](#overview)
- [Table of Contents](#table-of-contents)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Usage](#usage)
  - [Playbooks](#playbooks)
  - [Roles](#roles)
- [Configuration](#configuration)
  - [SSH Key](#ssh-key)
  - [Prometheus](#prometheus)
  - [Docker](#docker)
- [Contributing](#contributing)
- [License](#license)

## Prerequisites

- Ansible 2.9 or higher
- Access to the control node and worker nodes
- SSH access configured for Ansible to connect to the nodes

## Installation

1. Clone the repository:
    ```sh
    git clone https://github.com/yourusername/conclave-ansible.git
    cd conclave-ansible
    ```

2. Install required Ansible roles:
    ```sh
    ansible-galaxy install -r requirements.yml
    ```

## Usage

### Playbooks

- **Configure Control Node**: This playbook configures the control node with various roles.
    ```sh
    ansible-playbook playbook.yml --limit control-node
    ```

- **Configure Worker Nodes**: This playbook configures the worker nodes.
    ```sh
    ansible-playbook playbook.yml --limit workers
    ```

### Roles

- **ssh-key**: Manages SSH keys.
- **common**: Common configurations for all nodes.
- **conclave-backend**: Backend configurations for Conclave. This role performs a `git pull` of the Conclave backend repository, which requires GitHub to have access to the machine's SSH key.
- **munge**: Installs and configures Munge.
- **mariadb**: Installs and configures MariaDB.
- **slurmdbd**: Installs and configures SlurmDBD.
- **slurmctl**: Installs and configures Slurm Controller.
- **slurmd**: Installs and configures Slurm Daemons.
- **slurmrestd**: Installs and configures Slurm REST API.
- **nginx**: Installs and configures Nginx.
- **slurm-exporter**: Installs and configures Slurm Exporter.
- **docker**: Installs and configures Docker, Prometheus, Node Exporter, and Grafana.

## Configuration

### Inventory

The inventory file defines the hosts and groups of hosts that Ansible will manage. In this project, the inventory is defined in the `inventory/nodes` file.

#### Example Inventory

The `inventory/nodes` file contains the following groups and hosts:

```ini
[control-node]
smn01 ansible_host=192.168.4.194 hostname=smn01 cpus=4 realmemory=3195

[workers]
swn04 ansible_host=192.168.4.199 hostname=swn04 cpus=4 realmemory=3695
swn05 ansible_host=192.168.4.135 hostname=swn05 cpus=4 realmemory=3695
swn02 ansible_host=192.168.4.208 hostname=swn02 cpus=4 realmemory=3695
swn03 ansible_host=192.168.4.178 hostname=swn03 cpus=4 realmemory=3695
```

- **control-node**: This group contains the control node, which is responsible for managing the cluster.
- **workers**: This group contains the worker nodes, which perform the tasks assigned by the control node.

Each host entry includes:
- `ansible_host`: The IP address of the host.
- `hostname`: The hostname of the host.
- `cpus`: The number of CPUs available on the host.
- `realmemory`: The amount of memory available on the host in MB.

By organizing hosts into groups, you can apply different roles and configurations to different sets of hosts in your playbooks.

### SSH Key

The `ssh-key` role is responsible for managing SSH keys across the nodes. This role ensures that the necessary SSH keys are distributed and configured correctly for secure communication between the control node and worker nodes.

#### Configuration

To configure the `ssh-key` role, you need to provide your SSH keys. Place your private key and public key in the `roles/ssh-key/files` directory with the filenames `private-key` and `public-key`, respectively.

#### Usage

To apply the `ssh-key` role, include it in your playbook as shown below:

```yaml
- name: Configure SSH keys
  hosts: all
  become: yes
  roles:
    - ssh-key
```

This will ensure that the specified SSH key is distributed to all nodes in the inventory, allowing for secure and passwordless SSH access.

The `ssh-key` role performs the following tasks:
- Ensures the `/root/.ssh` directory exists with the correct permissions.
- Copies the provided private key to `/root/.ssh/id_rsa`.
- Copies the provided public key to `/root/.ssh/id_rsa.pub`.

Here is an example of the tasks defined in the `ssh-key` role:

```yaml
- name: Ensure /root/.ssh directory exists
  file:
    path: /root/.ssh
    state: directory
    mode: '0700'
    owner: root
    group: root

- name: Copy private key to /root/.ssh/id_rsa
  copy:
    src: files/private-key
    dest: /root/.ssh/id_rsa
    owner: root
    group: root
    mode: '0600'

- name: Copy public key to /root/.ssh/id_rsa.pub
  copy:
    src: files/public-key
    dest: /root/.ssh/id_rsa.pub
    owner: root
    group: root
    mode: '0644'
```

By following these steps, you can ensure that your SSH keys are properly set up and distributed across your nodes.

### Prometheus

The Prometheus configuration is managed through the `prometheus.yml` file. The template for this file is located at `roles/docker/templates/prometheus.yml.j2`.

Example configuration:
```yaml
global:
  scrape_interval: 5s
  evaluation_interval: 5s

scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"
    static_configs:
      - targets: ["smn01:9100", "swn02:9100", "swn03:9100", "swn04:9100"]

  - job_name: 'slurm-scraper'
    scrape_interval: 30s
    scrape_timeout: 30s
    static_configs:
        - targets: ['smn01:9092']

  - job_name: 'slurm_exporter'
    scrape_interval: 30s
    scrape_timeout: 30s
    static_configs:
        - targets: ['smn01:9341']

  - job_name: "grafana"
    static_configs:
      - targets: ["grafana:3000"]
```

### Docker

Docker setup can be controlled via variables in the playbook. For example:
```yaml
- role: docker
  vars:
    perform_docker_setup: true
    perform_prometheus_setup: true
    perform_node_exporter_setup: true
    perform_grafana_setup: false
    perform_slurm_exporter_setup: false
```

## Contributing

Contributions are welcome! Please fork the repository and create a pull request with your changes.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
```
