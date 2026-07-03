# Proxmox VM Collection

An Ansible collection for managing Proxmox virtual machines and templates.

## Description

This collection provides playbooks, roles, and modules for managing Proxmox virtual machines and templates. It includes functionality for creating VM templates from cloud images and cloning VMs from existing templates with cloud-init configuration.

## Requirements

- Ansible 2.13.9+
- Proxmox VE 6.0+

## Installation

To install this collection, use the ansible-galaxy command:

```bash
ansible-galaxy collection install opscores.proxmox_vm
```

## Usage

### Creating VM Templates from Cloud Images

```yaml
- name: Create Proxmox Templates from Cloud Images
  hosts: proxmox
  collections:
    - opscores.proxmox_vm
  vars:
    vm_memory: 4096
    vm_cores: 2
    vm_disk_size: "12G"
    vm_network_bridge: "vmbr0"
    cleanup_downloaded_images: false
    create_ubuntu24_template: true
    create_ubuntu26_template: true
    create_almalinux_template: true
    create_fedora_template: true
  tasks:
    - name: Create Templates
      include_role:
        name: create_templates
```

### Cloning VMs from Templates

```yaml
- name: Clone and Configure VM from Template
  hosts: proxmox
  collections:
    - opscores.proxmox_vm
  vars:
    # Source Template Configuration
    source_vm_id: 3002      # ID of the source template (e.g., 3000 for Ubuntu 24, 3004 for Ubuntu 26, 3001 for AlmaLinux, 3002 for Fedora)
    os_type: "fedora"      # Type of the OS, used to select the cloud-init template ('ubuntu24', 'ubuntu26', 'almalinux', or 'fedora')

    # New VM Configuration
    new_vm_id: 8001         # ID for the new VM
    new_vm_name: "test-fedora" # Desired hostname for the new VM
    cloud_user: "ansible"
    cloud_password: "{{ vault_cloud_password | default('changeme') }}"  # Use ansible-vault
    ssh_public_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}" # Path to your public SSH key

    # Proxmox Environment (values from inventory)
    target_node: "{{ proxmox_node }}"
    storage_pool: "{{ vm_storage_pool }}"
    full_clone: true
  tasks:
    - name: Clone VM
      include_role:
        name: clone_vm
```

## Roles

### create_templates

This role creates VM templates from cloud images. It supports Ubuntu, AlmaLinux, and Fedora distributions.

#### Variables

- `create_ubuntu24_template`: Whether to create Ubuntu 24.04 template (default: true)
- `create_ubuntu26_template`: Whether to create Ubuntu 26.04 template (default: true)
- `create_almalinux_template`: Whether to create AlmaLinux template (default: true)
- `create_fedora_template`: Whether to create Fedora template (default: true)
- `recreate_templates`: Whether to recreate existing templates (default: false)
- `recreate_ubuntu24_template`: Whether to recreate Ubuntu 24.04 template (default: follows recreate_templates)
- `recreate_ubuntu26_template`: Whether to recreate Ubuntu 26.04 template (default: follows recreate_templates)
- `recreate_almalinux_template`: Whether to recreate AlmaLinux template (default: follows recreate_templates)
- `recreate_fedora_template`: Whether to recreate Fedora template (default: follows recreate_templates)
- `cleanup_ubuntu24_image`: Whether to remove Ubuntu 24 image after import (default: follows cleanup_downloaded_images)
- `cleanup_ubuntu26_image`: Whether to remove Ubuntu 26 image after import (default: follows cleanup_downloaded_images)
- `cleanup_almalinux_image`: Whether to remove AlmaLinux image after import (default: follows cleanup_downloaded_images)
- `cleanup_fedora_image`: Whether to remove Fedora image after import (default: follows cleanup_downloaded_images)
- `ubuntu24_template_id`: ID for Ubuntu 24.04 template (default: 3000)
- `ubuntu26_template_id`: ID for Ubuntu 26.04 template (default: 3004)
- `almalinux_template_id`: ID for AlmaLinux template (default: 3001)
- `fedora_template_id`: ID for Fedora template (default: 3002)
- `ubuntu24_image_base_url`: Base URL for Ubuntu 24.04 cloud image
- `ubuntu26_image_base_url`: Base URL for Ubuntu 26.04 cloud image
- `almalinux_image_base_url`: Base URL for AlmaLinux cloud image
- `fedora_image_base_url`: Base URL for Fedora cloud image
- `ubuntu24_image_url`: Full URL for Ubuntu 24.04 cloud image
- `ubuntu26_image_url`: Full URL for Ubuntu 26.04 cloud image
- `almalinux_image_url`: Full URL for AlmaLinux cloud image
- `fedora_image_url`: Full URL for Fedora cloud image
- `ubuntu24_image_filename`: Filename for Ubuntu 24.04 image download
- `ubuntu26_image_filename`: Filename for Ubuntu 26.04 image download
- `almalinux_image_filename`: Filename for AlmaLinux image download
- `fedora_image_filename`: Filename for Fedora image download
- `vm_memory`: Memory size for VM templates (default: 1024 MB)
- `vm_cores`: Number of CPU cores for VM templates (default: 1)
- `vm_disk_size`: Disk size for VM templates (default: "12G")
- `vm_network_bridge`: Network bridge for VM templates (default: "vmbr0")
- `vm_cpu_type`: CPU type for VM templates (default: "kvm64")
- `vm_ostype`: OS type for VM templates (default: "l26")
- `vm_machine`: Machine type for VM templates (default: "q35")
- `vm_bios`: BIOS type for VM templates (default: "seabios")
- `cleanup_downloaded_images`: Whether to remove downloaded images after import (default: false)

### clone_vm

This role clones VMs from existing templates with cloud-init configuration.

The role includes the following checks:
- Verifies that the source template exists
- Confirms that the source VM is actually a template (template: 1)
- Validates that disk resize operation completes successfully (if enabled)

#### Variables

- `source_vm_id`: ID of the source template
- `new_vm_id`: ID for the new VM
- `new_vm_name`: Hostname for the new VM
- `os_type`: Type of the OS ('ubuntu24', 'ubuntu26', 'almalinux', or 'fedora')
- `target_node`: Proxmox node to create VM on
- `storage_pool`: Storage pool to use for VM
- `full_clone`: Whether to perform a full clone
- `recreate_vm`: Whether to recreate the VM if it already exists (default: false)
- `clone_vm_memory`: Memory size for the cloned VM (overrides default)
- `clone_vm_cores`: Number of CPU cores for the cloned VM (overrides default)
- `clone_vm_ostype`: OS type for the cloned VM (overrides default)
- `clone_vm_machine`: Machine type for the cloned VM (overrides default)
- `clone_vm_bios`: BIOS type for the cloned VM (overrides default)
- `clone_vm_cpu`: CPU type for the cloned VM (overrides default)
- `clone_vm_scsihw`: SCSI hardware type for the cloned VM (overrides default)
- `clone_vm_agent`: Guest agent configuration for the cloned VM (overrides default)
- `clone_vm_disk_size`: Disk size for the cloned VM (overrides default)
- `clone_vm_disk_slot`: Disk slot to resize (default: scsi0)
- `vm_resize_disk`: Whether to resize disk when cloning (default: true)
- `cloud_user`: Username for the cloud-init user
- `cloud_password`: Password for the cloud-init user
- `ssh_public_key`: SSH public key for the cloud-init user
- `dns_servers`: DNS servers for the VM
- `search_domains`: Search domains for the VM
- `use_dhcp_ip`: Whether to use DHCP instead of static IP (default: true)
- `static_ip_address`: Static IP address to assign to the VM (e.g., "192.168.1.100")
- `static_subnet_mask`: Subnet mask in CIDR format (default: "24")
- `static_gateway`: Static gateway address (e.g., "192.168.1.1")

## Security Considerations

For production use, consider the following security improvements:

1. Use Ansible Vault to encrypt sensitive variables like passwords
2. Use SSH key authentication instead of passwords
3. Implement proper network security policies

## License

GPL-3.0-or-later