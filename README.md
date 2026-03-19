# cert-automation

Ansible-based certificate lifecycle management for IIS servers across multiple customers and CAs.

## Project Structure

```
cert-automation/
├── inventory/
│   ├── customer_a/hosts.yml       # Per-customer inventory
│   └── customer_b/hosts.yml
├── group_vars/
│   └── all.yml                    # Shared config (thresholds, PasswordSafe URL, etc.)
├── roles/
│   ├── passwordsafe/              # Secret retrieval abstraction (Rackspace Identity + PasswordSafe API)
│   ├── cert_monitor/              # Phase 1 — expiry scanning and alerting
│   ├── iis_bind/                  # Phase 2 — cert import and IIS binding
│   ├── cert_renew/                # Phase 3 — renewal orchestration and scheduling
│   └── ca_digicert/               # Phase 4 — DigiCert REST API module
└── playbooks/
    ├── monitor.yml                # Run cert expiry checks across all customers
    ├── renew.yml                  # Trigger renewal workflow
    └── bind.yml                   # Import and bind a cert to IIS
```

## Phases

| Phase | Role | Status |
|-------|------|--------|
| 1 | cert_monitor | ✅ Complete |
| 2 | iis_bind | 🔜 Planned |
| 3 | cert_renew | 🔜 Planned |
| 4 | ca_digicert | 🔜 Planned |

## Prerequisites

- Ansible 2.14+ on a Linux control node
- `community.windows` and `ansible.windows` collections installed
- WinRM configured on all IIS target hosts
- A PasswordSafe project per customer containing credentials (see `roles/passwordsafe/README.md`)
- Identity service credentials for PasswordSafe auth stored in `~/.vault_pass` or passed via `--ask-vault-pass`

## Quick Start

```bash
# Install required collections
ansible-galaxy collection install ansible.windows community.windows community.general

# Check certs for a specific customer
ansible-playbook playbooks/monitor.yml -i inventory/customer_a/hosts.yml

# Check certs for all customers
for inv in inventory/*/hosts.yml; do
  ansible-playbook playbooks/monitor.yml -i "$inv"
done
```

## Secrets

All secrets are retrieved at runtime from PasswordSafe. No credentials are stored in this repository.
The only value that should be stored (encrypted with Ansible Vault) is the Identity service password
used to bootstrap the PasswordSafe auth token.

See `group_vars/all.yml` for configuration.
