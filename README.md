# Ansible Role Kubernetes CSI Longhorn

An Ansible role for installing and configuring Longhorn as a Container Storage Interface (CSI) for Kubernetes using Helm.

## Overview

This role automates the installation and configuration of Longhorn, ensuring that system-managed components run on control plane nodes and user-managed components run on worker nodes.

## Requirements

This role is optimized for **Ubuntu 24.04 LTS** and requires the following components:

1. Kubernetes v1.25 or newer.
2. A compatible container runtime such as Docker (v1.13+), containerd (v1.3.7+), or CRI-O.
3. Installed tools:
   - `bash`
   - `curl`
   - `grep`
   - `awk`
   - `lsblk`
4. Network and DNS functionality must be fully configured.

## Role Variables

### `longhorn_helm_values`

This dictionary contains key configuration settings for Longhorn:

- `minimalAvailablePercentage`: Defines the minimum storage percentage that must remain available. Default: `25%`.
- `overProvisioningPercentage`: Defines the over-provisioning percentage. Default: `100%`.
- `systemManagedComponents.nodeSelector`: Ensures system components run on control plane nodes.
- `userManagedComponents.nodeSelector`: Ensures user components run on worker nodes.

Example:

```yaml
longhorn_helm_values:
  minimalAvailablePercentage: 25
  overProvisioningPercentage: 100
  systemManagedComponents:
    nodeSelector:
      node-role.kubernetes.io/control-plane: "true"
  userManagedComponents:
    nodeSelector:
      node-role.kubernetes.io/worker: "true"
```

## Tasks

### 1. Operating System Preparation

Installs necessary dependencies and enables required services.

### 2. Verify Prerequisites

Ensures that the Kubernetes version is supported and all required tools are installed.

### 3. Deploy Longhorn via Helm

Adds the Longhorn Helm repository and installs the Longhorn Helm chart with the defined settings.

## Usage

1. Add the role to your `requirements.yml`:

   ```yaml
   - src: https://github.com/your-repo/ansible_role_kubernetes_csi_longhorn.git
     scm: git
     version: main
     name: ansible_role_kubernetes_csi_longhorn
   ```

2. Install the role:

   ```bash
   ansible-galaxy install -r requirements.yml
   ```

3. Use the role in your playbook:

   ```yaml
   - hosts: kube_nodes
     roles:
       - role: ansible_role_kubernetes_csi_longhorn
   ```

## Additional Notes

For a complete list of available Helm chart values, refer to the [Longhorn Helm Chart documentation](https://github.com/longhorn/charts/blob/v1.8.x/charts/longhorn/README.md#configuration).
