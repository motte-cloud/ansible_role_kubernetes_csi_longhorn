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

# Container runtime details
container_runtime: "containerd"
container_runtime_version: "1.3.7"

# Minimum Kubernetes version
kubernetes_min_version: "1.25"
```

---

## Tasks

### 1. Operating System Preparation

File: `tasks/os_prepare.yml`

```yaml
- name: Install necessary packages for Longhorn
  ansible.builtin.apt:
    name:
      - open-iscsi
      - nfs-common
      - bash
      - curl
      - grep
      - awk
      - lsblk
    state: present
    update_cache: yes

- name: Enable and start iscsid service
  ansible.builtin.systemd:
    name: iscsid
    enabled: true
    state: started
```

### 2. Verify Prerequisites

File: `tasks/verify_prerequisites.yml`

```yaml
- name: Check Kubernetes version
  ansible.builtin.shell: |
    kubectl version --output=json | jq -r '.serverVersion.gitVersion' | cut -c 2-
  register: kubernetes_version
  changed_when: false

- name: Fail if Kubernetes version is below minimum requirement
  ansible.builtin.fail:
    msg: "Kubernetes version {{ kubernetes_version.stdout }} is below the required version {{ kubernetes_min_version }}."
  when: kubernetes_version.stdout is version(kubernetes_min_version, '<')

- name: Check container runtime installation
  ansible.builtin.shell: |
    {{ container_runtime }} --version
  register: runtime_check
  changed_when: false

- name: Fail if container runtime is not installed
  ansible.builtin.fail:
    msg: "Container runtime {{ container_runtime }} is not installed or the version is below {{ container_runtime_version }}."
  when: runtime_check.stdout is not regex(container_runtime_version)
```

### 3. Deploy Longhorn via Helm

File: `tasks/deploy_helm.yml`

```yaml
- name: Add Longhorn Helm repository
  ansible.builtin.shell: |
    helm repo add longhorn https://charts.longhorn.io && helm repo update
  args:
    warn: false

- name: Install Longhorn CSI Helm chart
  community.kubernetes.helm:
    name: longhorn
    chart: longhorn/longhorn
    namespace: longhorn-system
    create_namespace: true
    values:
      defaultSettings:
        minimalAvailablePercentage: "{{ longhorn_helm_values.minimalAvailablePercentage }}"
        overProvisioningPercentage: "{{ longhorn_helm_values.overProvisioningPercentage }}"
    update_repo_cache: yes
```

---

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

If you have additional requirements or use cases, feel free to create an issue or submit a pull request in this repository.
