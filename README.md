# Ansible Role Kubernetes CSI Longhorn

An Ansible role for installing and configuring Longhorn as a Container Storage Interface (CSI) for Kubernetes using Helm.

## Role Structure

```plaintext
ansible_role_kubernetes_csi_longhorn/
├── defaults/
│   └── main.yml
├── tasks/
│   ├── main.yml
│   ├── os_prepare.yml
│   ├── verify_prerequisites.yml
│   └── deploy_helm.yml
├── templates/
├── files/
├── meta/
│   └── main.yml
└── README.md
```

---

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

---

## Variables

### `defaults/main.yml`

```yaml
# Helm chart configuration values
longhorn_helm_values:
  minimalAvailablePercentage: 25
  overProvisioningPercentage: 100
  systemManagedComponents:
    nodeSelector:
      node-role.kubernetes.io/control-plane: "true"
  userManagedComponents:
    nodeSelector:
      node-role.kubernetes.io/worker: "true"

# Minimum Kubernetes version
kubernetes_min_version: "1.25"
```

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

---

## Suggestions for Improvement

If you have additional requirements or use cases, feel free to create an issue or submit a pull request in the repository.
