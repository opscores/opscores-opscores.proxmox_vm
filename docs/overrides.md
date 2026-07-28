# Переопределение параметров

## Переопределение через inventory

Ansible поддерживает глубокое слияние словарей (deep merge). Определите `os_templates` в `group_vars` или `host_vars` для переопределения параметров конкретных ОС.

### Отключить создание шаблона Fedora

```yaml
# group_vars/proxmox.yml
os_templates:
  fedora:
    enabled: false
```

### Своё зеркало для Ubuntu 24

```yaml
# group_vars/proxmox.yml
os_templates:
  ubuntu24:
    image_url: "https://internal-mirror.corp/noble-server-cloudimg-amd64.img"
```

### Изменить template_id

```yaml
# group_vars/proxmox.yml
os_templates:
  almalinux:
    template_id: 9001
```

### Переопределить параметры VM для всех ОС

```yaml
# group_vars/proxmox.yml
vm_defaults:
  memory: 2048
  cores: 2
  disk_size: "20G"
  network_bridge: "vmbr1"
```

### Переопределить параметры VM для конкретной ОС

```yaml
# group_vars/proxmox.yml
os_templates:
  ubuntu24:
    memory: 4096
    cores: 4
```

## Добавление новой ОС

1. Добавьте запись в `os_templates` в `defaults/main.yml`
2. Создайте файл `cloud-init-<os_name>.yaml.j2` в `clone_vm/templates/`
3. Готово — задачи обнаружат новую ОС автоматически

### Пример: Debian 12

```yaml
# defaults/main.yml → os_templates
os_templates:
  # ... существующие ОС ...
  debian12:
    template_id: 3005
    vm_name: "debian-12-template"
    image_url: "https://mirror.yandex.ru/debian-cloud-images/bookworm/current/amd64/archive/debian-12-genericcloud-amd64.qcow2"
    image_filename: "debian-12-genericcloud-amd64.qcow2"
    enabled: true
    cloud_init_template: debian12
```

```yaml
# clone_vm/templates/cloud-init-debian12.yaml.j2
{% include '_common.yaml.j2' %}

packages:
  - qemu-guest-agent
  - curl
  - wget
  - htop
  - net-tools
  - sudo
  - openssh-server

runcmd:
  - [sed, -i, 's/#PermitRootLogin.*/PermitRootLogin no/', /etc/ssh/sshd_config]
  - [systemctl, restart, sshd]

final_message: "Debian 12 cloud-init completed at $UPTIME"
```

## Клонирование по имени ОС

Для клонирования по имени ОС (а не по VMID) используйте `os_templates` из inventory:

```yaml
# playbooks/clone_ubuntu24.yml
- hosts: proxmox
  vars:
    src_vm_id: "{{ os_templates['ubuntu24']['template_id'] | default(3000) }}"
    os_type: "ubuntu24"
  roles:
    - clone_vm
```
