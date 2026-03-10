Start here for the complete scaffold as an [interactive file browser](poc_generator.html).

## Getting the Files

**Option A — Download everything at once:**
Click **📦 Download All (.zip)** — this gives you `poc-infra.sh`, a self-extracting shell archive. Then on your Ubuntu machine:
```bash
bash poc-infra.sh        # extracts the full project
cd poc-infra
```

**Option B — Copy individual files:**
Click any file in the sidebar, then hit **📋 Copy** to copy it to clipboard or **⬇ File** to download just that one.

---

## From Zero to Running PoC — 4 Commands

```bash
# 1. Install deps (one-time)
sudo apt update && sudo apt install -y qemu-kvm libvirt-daemon-system \
  wireguard docker.io docker-compose-plugin curl jq terraform
sudo usermod -aG libvirt,docker $USER && newgrp libvirt

# 2. Start AIM stack
docker compose up -d

# 3. Seed Vault, OPA, WireGuard
bash scripts/bootstrap.sh

# 4. Run the full workflow
bash scripts/provision-vm.sh vm-operators dev poc-vm-001
```



## What Each File Does

- **`docker-compose.yml`** — spins up Keycloak, Vault, and OPA as containers
- **`opa/policies/vm_provision.rego`** — the policy engine; gates every provision request
- **`vault/policy/*.hcl`** — scoped permissions for dev vs prod machine identities
- **`terraform/main.tf`** — declares the KVM VM, clones the Ubuntu image, injects cloud-init
- **`terraform/cloud-init.tpl`** — runs inside the VM at first boot: fetches WireGuard key from Vault, brings up the tunnel
- **`scripts/bootstrap.sh`** — one-shot setup: configures Vault, generates WireGuard keys, smoke-tests OPA
- **`scripts/provision-vm.sh`** — the end-to-end demo loop: OPA → Vault → Terraform → WireGuard
