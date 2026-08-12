# Ansible Role: WireGuard

| Source | Version | CI | License |
| --- | --- | --- | --- |
| [![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-wireguard) | [![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-wireguard)](https://github.com/grzegorzfranus/ansible-role-wireguard/releases) | [![CI](https://github.com/grzegorzfranus/ansible-role-wireguard/actions/workflows/ci.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-wireguard/actions/workflows/ci.yml) | [![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE) |

An enterprise-grade, production-ready Ansible role to install and configure [WireGuard](https://www.wireguard.com/) on Ubuntu and Debian distributions via native APT packages. This role automates the build of a full-mesh Virtual Private Network (VPN) topology across all hosts of an inventory group, ensuring encrypted point-to-point communication between every pair of nodes with host-isolated private keys and unique per-peer-pair pre-shared keys (PSK).

---

## ✨ Features

- **Native APT Integration**: Installs WireGuard kernel tooling and utility CLI packages (`wireguard`, `wireguard-tools`, `iproute2`) using native APT package management.
- **Automated Full-Mesh VPN Topology**: Automatically establishes direct peer-to-peer WireGuard tunnels between every host pair in the designated mesh group.
- **Host-Local Cryptographic Key Management**: Private keys are generated directly on target hosts (`wg genkey`) under strict `0700`/`0600` root permissions and never leave the node. All sensitive key tasks enforce `no_log: true`.
- **Per-Pair Pre-Shared Keys (PSK)**: Generates unique 256-bit pre-shared keys (`wg genpsk`) for each peer host pair for post-quantum security defense-in-depth.
- **Zero Downtime Configuration Reloads**: Applies configuration updates seamlessly via `wg syncconf`, preserving active tunnels without dropping state.
- **Dual-Layer Input Validation**: Combines static argument specification checks (`meta/argument_specs.yml`) with runtime assertion ladders (`tasks/assert.yml`).
- **State-Driven Lifecycle**: Supports complete service installation (`wireguard_state: present`) and clean uninstallation (`wireguard_state: absent`).

---

## 🎯 Architecture

### Mesh Topology Diagram

In a full-mesh topology, every node connects directly to every other node in the `wireguard_mesh_group`. Unlike hub-and-spoke topologies that route traffic through a central gateway, full-mesh eliminates single points of failure, minimizes latency, and maximizes inter-host bandwidth.

```text
               +-------------------+
               |    Host Alpha     |
               | (10.8.0.11/24)    |
               +---------+---------+
                        / \
                       /   \
                      /     \
                     /       \
  +-----------------+         +-----------------+
  |    Host Beta    |<------->|    Host Gamma   |
  | (10.8.0.12/24)  |         | (10.8.0.13/24)  |
  +-----------------+         +-----------------+
```

### Key Distribution Flow

```text
 [ Target Host A ]                             [ Target Host B ]
        │                                              │
 1. wg genkey ──> /etc/wireguard/keys/private.key       │
        │                                              │
 2. wg pubkey  ──> public key A                        │
        │                                              │
 3. wg genpsk  ──> psk_A_B (lexicographically lower)   │
        │                                              │
        └───────────────── Slurp PSK ─────────────────>│ (Stores psk_A_B)
                                                       │
 4. Render wg0.conf with peer B public key & PSK       │ 4. Render wg0.conf with peer A public key & PSK
```

### Topology Comparison: Full-Mesh vs Hub-and-Spoke

- **Why Full-Mesh?**: Ideal for cluster nodes (e.g. Kubernetes worker/control plane nodes, distributed database nodes, storage clusters) requiring low-latency direct node-to-node communication.
- **When to prefer Hub-and-Spoke?**: Prefer hub-and-spoke when managing thousands of dynamic ephemeral client devices or road-warrior laptops connecting to central corporate gateways.

### Delivery Method Decision: Native Package (APT)

#### Architecture Rationale

- **Low Overhead & Performance**: WireGuard runs in kernel space (`wireguard.ko`). Installing native APT packages ensures kernel module integration and maximum throughput without container networking overhead.
- **Systemd Integration**: Uses native `wg-quick@wg0` systemd unit template for system lifecycle management and automated startup.
- **Security Isolation**: Operates directly under Linux kernel networking boundaries, allowing precise firewall enforcement (`iptables`/`nftables`).

---

## 📋 Requirements

### Supported Operating Systems

| OS Family | Distribution | Version / Codename | Status |
| --- | --- | --- | --- |
| Debian | Ubuntu | 24.04 LTS (Noble Numbat) | Supported |
| Debian | Ubuntu | 26.04 LTS (Resolute) | Supported |
| Debian | Debian | 12 (Bookworm) | Supported |
| Debian | Debian | 13 (Trixie) | Supported |

### Core Requirements

- **Ansible Core**: Version `>= 2.15`
- **Python**: Version `>= 3.9` on control node and target hosts
- **Collections**: `ansible.utils`, `ansible.posix`
- **Privileges**: Root privilege escalation (`become: true`) on target hosts
- **Playbook Scope**: Playbooks executing this role **must target the entire mesh group** simultaneously (e.g., `hosts: wireguard`) so Ansible can gather facts and public keys across all nodes in the mesh.

---

## 🚀 Quick Start

### Step 1: Define Inventory Group & Host Variables

Create an inventory file `hosts.yml` containing the mesh inventory group:

```yaml
all:
  children:
    wireguard:
      hosts:
        node1.example.com:
          ansible_host: 192.0.2.11
          wireguard_address: 10.8.0.11/24
        node2.example.com:
          ansible_host: 192.0.2.12
          wireguard_address: 10.8.0.12/24
        node3.example.com:
          ansible_host: 192.0.2.13
          wireguard_address: 10.8.0.13/24
```

### Step 2: Write Minimal Playbook

Create `site.yml`:

```yaml
---
- name: Deploy WireGuard Full-Mesh VPN
  hosts: wireguard
  become: true
  roles:
    - role: grzegorzfranus.wireguard
```

### Step 3: Execute Playbook

```bash
ansible-playbook -i hosts.yml site.yml
```

---

## ⚙️ Configuration

### Default Configuration

By default, the role creates interface `wg0` listening on UDP port `51820`, generates 256-bit host keypairs and per-pair PSKs, and renders `/etc/wireguard/wg0.conf`.

### Rendered `/etc/wireguard/wg0.conf` Example

```ini
# Ansible managed

[Interface]
Address = 10.8.0.11/24
ListenPort = 51820
PrivateKey = <SECRET_HOST_PRIVATE_KEY>
SaveConfig = false

[Peer]
# Peer: node2.example.com
PublicKey = <PUBLIC_KEY_NODE2>
PresharedKey = <SHARED_PSK_NODE1_NODE2>
Endpoint = 192.0.2.12:51820
AllowedIPs = 10.8.0.12/32

[Peer]
# Peer: node3.example.com
PublicKey = <PUBLIC_KEY_NODE3>
PresharedKey = <SHARED_PSK_NODE1_NODE3>
Endpoint = 192.0.2.13:51820
AllowedIPs = 10.8.0.13/32
```

---

## 📊 Variables

### Lifecycle Options

| Variable | Description | Default |
| --- | --- | --- |
| `wireguard_state` | Target state for WireGuard installation (`present` or `absent`) | `'present'` |
| `wireguard_remove_packages` | Uninstall WireGuard packages when `wireguard_state` is `absent` | `true` |
| `wireguard_purge_keys` | Purge key directory and main config directory on removal | `false` |

### General & Network Settings

| Variable | Description | Default |
| --- | --- | --- |
| `wireguard_mesh_group` | Name of inventory group whose members form the mesh | `'wireguard'` |
| `wireguard_interface` | Name of the WireGuard network interface | `'wg0'` |
| `wireguard_port` | UDP listening port for WireGuard service | `51820` |
| `wireguard_network` | WireGuard mesh network CIDR subnet | `'10.8.0.0/24'` |
| `wireguard_address` | Per-host IP address with prefix inside mesh network | `""` |
| `wireguard_endpoint` | Public IP or FQDN used by peers to connect to this host | `"{{ ansible_host }}"` |
| `wireguard_mtu` | Optional interface MTU size (null uses system default) | `null` |
| `wireguard_persistent_keepalive` | Keepalive interval in seconds (0 disables keepalive) | `0` |
| `wireguard_enable_forwarding` | Enable kernel IPv4 packet forwarding via sysctl | `false` |

### Key Management & Security Settings

| Variable | Description | Default |
| --- | --- | --- |
| `wireguard_use_preshared_keys` | Generate unique per-peer-pair pre-shared keys (PSK) | `true` |
| `wireguard_regenerate_keys` | Force regeneration of target host private/public keys and PSKs | `false` |

---

## 📌 Role Properties

| Property | Value | Description |
| --- | --- | --- |
| **Idempotent** | Yes | Running the role multiple times produces identical state without unnecessary changes. |
| **Atomic** | Yes | Configurations pass validation (`wg-quick strip`) post-rendering before handlers run. |
| **Check Mode** | Supported | Supports `--check` dry-run mode without mutating target state. |
| **Diff Mode** | Supported | Generates git-style diffs for configuration template updates. |
| **Upgrade-Safe** | Yes | Role updates package versions without destroying host private key material. |

---

## 📤 Role Output

This role does not set any public output facts. All internal facts use the double-underscore prefix (e.g., `__wireguard_private_key`, `__wireguard_public_key`, `__wireguard_psks`).

---

## 🔍 Verification

Execute the following verification commands on any mesh node after deployment:

### 1. Inspect WireGuard Interface Status

```bash
sudo wg show
```

### 2. Verify Systemd Service State

```bash
sudo systemctl status wg-quick@wg0
```

### 3. Verify End-to-End Tunnel Connectivity

```bash
ping -c 3 10.8.0.12
```

---

## 🛡️ Security Features

- **Host-Local Private Keys**: Generated on target hosts using `wg genkey` with strict `0700`/`0600` permissions. Private keys never leave the node.
- **Sensitive Parameter Masking**: All tasks reading or processing key material enforce `no_log: true` to prevent secrets leakage in CI/CD logs.
- **Per-Pair Pre-Shared Keys**: Generates unique PSKs per peer host pair (`wg genpsk`), adding post-quantum symmetric encryption defense-in-depth.
- **Strict AllowedIPs Boundaries**: Restricts peer `AllowedIPs` strictly to that peer's `/32` address, preventing unwanted routing or traffic spoofing.
- **Disabled Forwarding by Default**: `wireguard_enable_forwarding` defaults to `false`, preventing nodes from acting as unintentional transit routers.
- **Post-Render Validation**: Validates rendered configurations with `wg-quick strip` prior to handler execution.
- **Firewall Requirements**: Ensure UDP port `51820` (or `wireguard_port`) is open on ingress firewalls (e.g. `ufw allow 51820/udp` or `nftables`/`iptables`).

---

## Uninstall & Roll-back

To cleanly uninstall WireGuard and remove configuration files from target nodes, execute with `wireguard_state: absent`:

```yaml
---
- name: Uninstall WireGuard
  hosts: wireguard
  become: true
  vars:
    wireguard_state: "absent"
    wireguard_remove_packages: true
    wireguard_purge_keys: true
  roles:
    - role: grzegorzfranus.wireguard
```

---

## 🧪 Check mode behavior

When executed with `--check` mode:
- Static assertions and specification validations run normally.
- Template rendering dry-runs display proposed file diffs.
- Key generation, service state mutations, and package installations are safely skipped.

---

## 🔧 Troubleshooting

### Common Symptoms & Diagnostics

#### Interface fails to start (Kernel module missing)

```bash
# Check if kernel module is loaded
lsmod | grep wireguard

# Attempt manual module load
sudo modprobe wireguard
```

#### No connectivity between peers

```bash
# Inspect handshakes and endpoint status
sudo wg show

# Verify UDP port listening status
sudo ss -tuln | grep 51820

# Monitor systemd journal logs
sudo journalctl -u wg-quick@wg0 -f
```

#### MTU and Packet Fragmentation

For nodes behind PPPoE or cloud provider networks with lower MTU, set `wireguard_mtu: 1420` in `host_vars` or `group_vars`.

#### NAT / Firewall Timeout Issues

For nodes behind NAT gateways or stateful firewalls, set `wireguard_persistent_keepalive: 25` in `host_vars`.

---

## 📁 File Structure

```text
ansible-role-wireguard/
├── .github/
│   ├── ISSUE_TEMPLATE/          # Issue report templates (bug, feature, task)
│   │   ├── bug_report.yml
│   │   ├── config.yml
│   │   ├── feature_request.yml
│   │   └── task.yml
│   ├── PULL_REQUEST_TEMPLATE/    # Pull request description template
│   │   └── pull_request_template.md
│   ├── workflows/               # Centralized GitHub Actions CI/CD workflows
│   │   ├── ci.yml
│   │   └── release.yml
│   └── dependabot.yml           # Dependabot configuration for GitHub Actions
├── defaults/
│   └── main.yml                 # Default configuration variables
├── handlers/
│   └── main.yml                 # Dynamic reload and service restart handlers
├── meta/
│   ├── main.yml                 # Role metadata and Galaxy specifications
│   └── argument_specs.yml       # Native argument specification validation
├── molecule/                    # Molecule testing framework
│   └── default/                 # Default testing scenario
│       ├── converge.yml
│       ├── molecule.yml
│       ├── prepare.yml
│       └── verify.yml
├── tasks/
│   ├── main.yml                 # Main task orchestration
│   ├── assert.yml               # Preflight parameter assertions
│   ├── prerequisites.yml        # OS-family prerequisites dispatcher
│   ├── prerequisites_debian.yml # APT repository prerequisites (Debian/Ubuntu)
│   ├── install.yml              # Package installation dispatcher
│   ├── install_debian.yml       # APT package installation (Debian/Ubuntu)
│   ├── keys.yml                 # Host keypair & per-pair PSK management
│   ├── mesh_facts.yml           # Full-mesh topology discovery & key exchange
│   ├── configure.yml            # WireGuard configuration deployment
│   ├── service.yml              # Service state management
│   ├── remove.yml               # Removal dispatcher
│   └── remove_debian.yml        # APT package removal (Debian/Ubuntu)
├── templates/
│   └── wg.conf.j2               # Main WireGuard interface configuration template
└── vars/
    ├── main.yml                 # Internal constants
    └── debian.yml               # Debian/Ubuntu OS package definitions
```

---

## 🏷️ Tags

| Tag | Description |
| --- | --- |
| `wireguard_assert` | Run variable assertion ladder |
| `wireguard_prerequisites` | Install system update prerequisites |
| `wireguard_install` | Install WireGuard package binaries |
| `wireguard_keys` | Manage host cryptographic keys and peer PSKs |
| `wireguard_mesh` | Discover mesh topology and peer endpoints |
| `wireguard_configure` | Render interface configuration files |
| `wireguard_service` | Manage systemd service state |
| `wireguard_remove` | Uninstall WireGuard configuration and packages |

---

## CI/CD Pipeline

- **Continuous Integration (`ci.yml`)**: Runs on every pull request targeting `main`. Executes branch name linting, PR title Conventional Commit verification, `yamllint`, `ansible-lint`, `actionlint`, and Molecule test matrix (`ubuntu2404`, `ubuntu2604`, `debian12`, `debian13`).
- **Release & Publishing (`release.yml`)**: Automates version bumping, changelog generation via Release Please, and publishes tagged releases to Ansible Galaxy upon merging to `main`.

---

## Example Playbooks

### Production Mesh Playbook with NATed Node

```yaml
---
- name: Deploy Production WireGuard Full-Mesh VPN
  hosts: wireguard
  become: true
  vars:
    wireguard_mesh_group: "wireguard"
    wireguard_port: 51820
    wireguard_use_preshared_keys: true
    wireguard_enable_forwarding: false
  roles:
    - role: grzegorzfranus.wireguard
```

#### Inventory File (`inventories/production/hosts.yml`)

```yaml
all:
  children:
    wireguard:
      hosts:
        dc-node-1.example.com:
          ansible_host: 198.51.100.10
          wireguard_address: 10.8.0.1/24
        dc-node-2.example.com:
          ansible_host: 198.51.100.20
          wireguard_address: 10.8.0.2/24
        branch-nat-1.example.com:
          ansible_host: 203.0.113.50
          wireguard_address: 10.8.0.10/24
          wireguard_persistent_keepalive: 25
```

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your branch from `main`
- Use [Conventional Commits](https://www.conventionalcommits.org/) for commit messages:
  - `feat:` — new features
  - `fix:` — bug fixes
  - `refactor:` — code refactoring
  - `docs:` — documentation changes
  - `ci:` — CI/CD pipeline updates
  - `build:` — dependency and build configuration updates
  - `chore:` — maintenance tasks
  - `test:` — test additions or corrections
  - `perf:` — performance improvements
  - `revert:` — code reverts
  - `style:` — code formatting and style
- Use branch naming convention: `feature/`, `bugfix/`, `fix/`, `hotfix/`, `release/`, `chore/`, `docs/`, `refactor/`, `test/`, `build/`, `ci/`, `perf/`, `revert/`
- Ensure your code passes all CI checks (YAML lint, Ansible lint, Molecule tests)
- Centralized workflows from [github-workflows](https://github.com/grzegorzfranus/github-workflows) are used to run CI/CD pipelines
- Submit a pull request describing your changes (a template is available under `.github/PULL_REQUEST_TEMPLATE/pull_request_template.md` to help structure your PR description)
- For major changes, please open an issue first to discuss what you would like to change (issue templates for bug reports, feature requests, and tasks are available under `.github/ISSUE_TEMPLATE/`)

## 📝 License

This project is licensed under the Apache-2.0 License - see the LICENSE file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).