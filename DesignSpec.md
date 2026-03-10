# poc-infra

Bare-minimum proof of concept for an on-premise infrastructure stack.
**AIM · VM Provisioning · WireGuard Security**

## Stack

| Component    | Tool                 | Role                        |
|--------------|----------------------|-----------------------------|
| Identity     | Keycloak 24          | OIDC / user management      |
| Secrets      | HashiCorp Vault 1.16 | Key/secret lifecycle        |
| Policy       | OPA 0.65             | Provisioning authorization  |
| Hypervisor   | KVM / libvirt        | VM runtime                  |
| Provisioning | Terraform + libvirt  | Declarative VM lifecycle    |
| Tunnel       | WireGuard            | Encrypted VM network        |

## Requirements

- Ubuntu 22.04 LTS
- 16 GB RAM, 4 cores, 100 GB free disk
- VT-x / AMD-V enabled in BIOS
- Internet access

## Quickstart

```bash
# 1. Install dependencies
sudo apt update && sudo apt install -y \
  qemu-kvm libvirt-daemon-system libvirt-clients virtinst \
  wireguard wireguard-tools docker.io docker-compose-plugin \
  curl jq git unzip

curl -fsSL https://apt.releases.hashicorp.com/gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp.gpg] \
  https://apt.releases.hashicorp.com $(lsb_release -cs) main" \
  | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install -y terraform

sudo usermod -aG libvirt,docker $USER && newgrp libvirt

# 2. Start AIM stack
docker compose up -d

# 3. Bootstrap Vault + OPA + WireGuard
bash scripts/bootstrap.sh

# 4. Provision a VM (full workflow)
bash scripts/provision-vm.sh vm-operators dev poc-vm-001

# 5. Test policy denial
bash scripts/provision-vm.sh dev-team prod poc-vm-002
```

## Access

| Service   | URL                    | Credentials |
|-----------|------------------------|-------------|
| Keycloak  | http://localhost:8080  | admin/admin |
| Vault     | http://localhost:8200  | token: root |
| OPA       | http://localhost:8181  | —           |

## Teardown

```bash
bash scripts/teardown.sh poc-vm-001
```
