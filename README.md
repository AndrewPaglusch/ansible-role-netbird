# Ansible NetBird Role

A simple and opinionated Ansible role for installing the NetBird agent and registering hosts with a NetBird management server.

## Features

- Installs the NetBird agent from official package repositories (Debian and RHEL families)
- Registers the host with a NetBird management server using a setup key
- Idempotent registration. Only runs `netbird up` when the host is not already connected
- Supports passing arbitrary extra flags through to `netbird up`

## Limitations

- No setup keys are generated for you. You must provide them
- Only Debian and RHEL family systems are supported
- This role manages the NetBird agent only. It does not deploy or configure a NetBird management server

## Requirements

- Ansible 2.4+
- A reachable NetBird management server and a valid setup key

## Role Variables

### Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `netbird_setup_key` | Setup key used to register the host with the management server | `"{{ vault_netbird_setup_key }}"` |

### Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `netbird_management_url` | URL of the NetBird management server | `"https://netbird.example.com:443"` |
| `netbird_extra_flags` | List of extra flags appended verbatim to `netbird up` | `[]` (empty list) |
| `netbird_package_state` | Package state for the `netbird` package. Set to `latest` to upgrade to the newest version on each run | `present` |

## Example Playbooks

### Simple Registration

This example installs NetBird and registers the host with a management server:

```yaml
---
- hosts: all
  become: yes
  roles:
    - role: netbird
      vars:
        netbird_management_url: "https://netbird.example.com:443"
        netbird_setup_key: "{{ vault_netbird_setup_key }}"
```

### Registration with Extra Flags

This example disables NetBird's DNS handling and sets a custom hostname:

```yaml
---
- hosts: all
  become: yes
  roles:
    - role: netbird
      vars:
        netbird_management_url: "https://netbird.example.com:443"
        netbird_setup_key: "{{ vault_netbird_setup_key }}"
        netbird_extra_flags:
          - --disable-dns
          - --hostname {{ inventory_hostname_short }}
```

## Updating Flags

Changes to `netbird_extra_flags` are detected and re-applied automatically on the next role run. The host's mesh IP is preserved, but expect a brief mesh connectivity blip while the new flags are applied.

## Setup Key Management

This role does not generate setup keys. You must create them ahead of time in your NetBird management dashboard and provide them via a variable. It's recommended to store setup keys using Ansible Vault.
