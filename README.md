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
- **Dedicated Logging & Observability**: Optional rsyslog routing for kernel dynamic debug (`dyndbg`) and `wg-quick` events into `/var/log/wireguard/wireguard.log`, log rotation via `logrotate`, and systemd timer status snapshots into `/var/log/wireguard/wireguard-status.log`.
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

### Logging & Observability Architecture

When `wireguard_configure_logging: true` is set, kernel dynamic debug options (`options wireguard dyndbg=+p`) and `wg-quick` events are routed into dedicated log files. Note that `journald` receives `kmsg` events independently of rsyslog (so Grafana Alloy -> Loki pipelines remain completely unaffected).

```text
  [kernel dyndbg] + [wg-quick] ---> [rsyslog] ---> [/var/log/wireguard/wireguard.log]
                                                ---> [logrotate -> /var/log/archive/wireguard]

  [systemd timer] ------------> [wg show all dump] ---> [/var/log/wireguard/wireguard-status.log]
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

---

## 📋 Requirements

### Supported Operating Systems

| OS Family | Distribution | Version / Codename | Status |
| --- | --- | --- | --- |
| Debian | Ubuntu | 24.04 LTS (Noble Numbat) | Supported |
| Debian | Ubuntu | 26.04 LTS (Resolving / Resolute) | Supported |
| Debian | Debian | 12 (Bookworm) | Supported |
| Debian | Debian | 13 (Trixie) | Supported |

### Core Requirements

- **Ansible Core**: Version `>= 2.15`
- **Python**: Version `>= 3.9` on control node and target hosts
- **Collections**: `ansible.utils`, `ansible.posix`
- **Kernel Debugging (Optional Logging)**: Requires `CONFIG_DYNAMIC_DEBUG=y` and `CONFIG_DYNAMIC_DEBUG_CORE=y` in target kernel for WireGuard dyndbg debug events (present by default in official Ubuntu/Debian kernels).
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

### Transit Nodes & Routed Networks

By default a peer entry carries a single host route, so the mesh reaches mesh members and
nothing else. A node that forwards traffic for an additional network on behalf of the mesh —
for example a gateway bridging another VPN or a branch LAN into the tunnel — declares that
network in its own host variables:

```yaml
# host_vars/gateway.example.com/wireguard.yml
wireguard_address: "10.8.0.20/24"
wireguard_enable_forwarding: true
wireguard_routed_networks:
  - "100.64.0.0/10"
```

Every other mesh node then renders that prefix into the gateway peer entry, and `wg-quick`
installs the matching route on each of them:

```ini
[Peer]
# Peer: gateway.example.com
PublicKey = <PUBLIC_KEY_GATEWAY>
Endpoint = 192.0.2.20:51820
AllowedIPs = 10.8.0.20/32, 100.64.0.0/10
```

Two rules govern this variable. A prefix must be advertised by exactly one mesh node, because
WireGuard cryptokey routing rejects a configuration whose peers declare overlapping
`AllowedIPs`; the role asserts this across the mesh group at run time. The gateway itself also
needs `wireguard_enable_forwarding: true` and a host firewall policy that permits the transit,
neither of which follows automatically from declaring the network.

---

## 📊 Variables

### Lifecycle Options

| Variable | Description | Default |
| --- | --- | --- |
| `wireguard_state` | Target state for WireGuard installation (`present` or `absent`) | `'present'` |
| `wireguard_remove_packages` | Uninstall WireGuard packages when `wireguard_state` is `absent` | `true` |
| `wireguard_purge_keys` | Purge key directory and main config directory on removal | `false` |
| `wireguard_purge_logs` | Purge log directory and archive directory when `wireguard_state` is `absent` | `false` |

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
| `wireguard_sysctl_file` | Drop-in the forwarding setting is persisted to, so it survives a reboot | `/etc/sysctl.d/60-wireguard.conf` |
| `wireguard_routed_networks` | Extra CIDR networks this host routes for the mesh, appended by peers to its `AllowedIPs` | `[]` |

### Key Management & Security Settings

| Variable | Description | Default |
| --- | --- | --- |
| `wireguard_use_preshared_keys` | Generate unique per-peer-pair pre-shared keys (PSK) | `true` |
| `wireguard_regenerate_keys` | Force regeneration of target host private/public keys and PSKs | `false` |

### Logging & Observability Settings

| Variable | Description | Default |
| --- | --- | --- |
| `wireguard_configure_logging` | Master toggle to enable dedicated WireGuard logging | `false` |
| `wireguard_install_rsyslog` | Auto-install rsyslog and logrotate APT packages if missing | `true` |
| `wireguard_log_dir` | Base directory for WireGuard dedicated log files | `"/var/log/wireguard"` |
| `wireguard_log_file` | Primary log file path for kernel dyndbg and wg-quick messages | `"{{ wireguard_log_dir }}/wireguard.log"` |
| `wireguard_log_dir_permissions` | Directory octal permissions for `wireguard_log_dir` | `"0750"` |
| `wireguard_log_file_permissions` | File octal permissions for WireGuard log files | `"0640"` |
| `wireguard_log_user` | User owner for WireGuard log directory and files | `"root"` |
| `wireguard_log_group` | Group owner for WireGuard log directory and files | `"adm"` |
| `wireguard_rsyslog_config_file` | Filename for rsyslog fragment under `/etc/rsyslog.d/` | `"49-wireguard.conf"` |
| `wireguard_enable_kernel_debug_logging` | Enable kernel dynamic debug (`dyndbg`) for wireguard module | `true` |
| `wireguard_kernel_debug_persistent` | Persist kernel dyndbg options via `/etc/modprobe.d/wireguard.conf` | `true` |
| `wireguard_enable_status_snapshot` | Enable periodic WireGuard status snapshot service and timer | `true` |
| `wireguard_status_snapshot_file` | Output log file path for periodic systemd status snapshots | `"{{ wireguard_log_dir }}/wireguard-status.log"` |
| `wireguard_status_snapshot_interval` | Systemd timer execution interval for status snapshots | `"5min"` |
| `wireguard_logrotate_options` | Dictionary specifying log rotation frequency, retention count, and archive dir | *(see `defaults/main.yml`)* |

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

### 2. Tail Dedicated WireGuard Event Logs

```bash
sudo tail -f /var/log/wireguard/wireguard.log
```

### 3. Verify Logrotate Configuration

```bash
sudo logrotate -d /etc/logrotate.d/wireguard
```

### 4. Check Status Snapshot Timer & Log File

```bash
sudo systemctl status wireguard-status.timer
sudo tail -n 20 /var/log/wireguard/wireguard-status.log
```

### 5. Inspect Kernel Dynamic Debug Flags

```bash
sudo grep wireguard /proc/dynamic_debug/control
```

---

## 🛡️ Security Features

- **Host-Local Private Keys**: Generated on target hosts using `wg genkey` with strict `0700`/`0600` permissions. Private keys never leave the node.
- **Sensitive Parameter Masking**: All tasks reading or processing key material enforce `no_log: true` to prevent secrets leakage in CI/CD logs.
- **Per-Pair Pre-Shared Keys**: Generates unique PSKs per peer host pair (`wg genpsk`), adding post-quantum symmetric encryption defense-in-depth.
- **Kernel Debug Privacy**: Dynamic debug (`dyndbg`) logs peer indices and endpoint addresses to system logs for debugging, but **NEVER logs host private keys**.
- **Restricted Log File Permissions**: Dedicated log files and directories are created with `0750`/`0640` permissions owned by `root:adm`.
- **Clean Artifact Purging**: When `wireguard_state: absent` and `wireguard_purge_logs: true`, all log files and archive directories are safely purged.

---

## Uninstall & Roll-back

To cleanly uninstall WireGuard, remove configuration files, and purge log artifacts from target nodes:

```yaml
---
- name: Uninstall WireGuard and Purge Logs
  hosts: wireguard
  become: true
  vars:
    wireguard_state: "absent"
    wireguard_remove_packages: true
    wireguard_purge_keys: true
    wireguard_purge_logs: true
  roles:
    - role: grzegorzfranus.wireguard
```

---

## 🧪 Check Mode Behavior

When executed with `--check` mode:
- Static assertions and specification validations run normally.
- Template rendering dry-runs display proposed file diffs.
- Key generation, service state mutations, and package installations are safely skipped.
- Runtime dynamic debug initialization (`echo 'module wireguard +p' > /proc/dynamic_debug/control`) is safely skipped.

---

## 🔧 Troubleshooting

### Common Symptoms & Diagnostics

#### Interface fails to start (Kernel module missing)

```bash
lsmod | grep wireguard
sudo modprobe wireguard
```

#### Log file `/var/log/wireguard/wireguard.log` is empty

- **Check Kernel Config**: Ensure target kernel was compiled with `CONFIG_DYNAMIC_DEBUG=y` (`zgrep DYNAMIC_DEBUG /proc/config.gz` or `/boot/config-*`).
- **Check Modprobe Options**: Confirm modprobe options are present (`cat /etc/modprobe.d/wireguard.conf`).
- **Check Dynamic Debug Control**: Verify `=p` is set for wireguard in `/proc/dynamic_debug/control` (`grep wireguard /proc/dynamic_debug/control`).
- **Check Rsyslog Status**: Confirm rsyslog service is running and active (`systemctl status rsyslog`).

#### `wg-quick` events do not appear in log file

The rsyslog rule `$programname == "wg-quick"` relies on `systemd-journald` forwarding messages to `syslog` (`ForwardToSyslog`). Default `journald` settings vary across distributions and systemd versions.

**Diagnostic steps**:

```bash
systemd-analyze cat-config systemd/journald.conf | grep -i forwardtosyslog
grep -r ForwardToSyslog /etc/systemd/journald.conf /etc/systemd/journald.conf.d/
journalctl -u wg-quick@wg0 --no-pager -n 20
```

**Workaround**:

Set `ForwardToSyslog=yes` in `/etc/systemd/journald.conf.d/forward.conf`:

```ini
[Journal]
ForwardToSyslog=yes
```

*Note: Kernel dynamic debug (`dyndbg`) events reach rsyslog directly via `imklog` independently of journald settings.*

---

## 📁 File Structure

```text
ansible-role-wireguard/
├── defaults/
│   └── main.yml                 # Default configuration variables
├── handlers/
│   └── main.yml                 # Dynamic reload and service restart handlers
├── meta/
│   ├── main.yml                 # Role metadata and Galaxy specifications
│   └── argument_specs.yml       # Native argument specification validation
├── molecule/                    # Molecule testing framework
│   ├── default/                 # Default testing scenario
│   │   ├── converge.yml
│   │   ├── molecule.yml
│   │   ├── prepare.yml
│   │   └── verify.yml
│   └── logging/                 # Dedicated logging testing scenario
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
│   ├── logging.yml              # Dedicated logging orchestration
│   ├── logging_debian.yml       # APT logging prerequisites (rsyslog, logrotate)
│   ├── service.yml              # Service state management
│   ├── remove.yml               # Removal dispatcher
│   └── remove_debian.yml        # APT package removal (Debian/Ubuntu)
├── templates/
│   ├── logrotate/
│   │   └── wireguard.j2         # Logrotate configuration template
│   ├── modprobe/
│   │   └── wireguard.conf.j2    # Modprobe dyndbg configuration template
│   ├── rsyslog/
│   │   └── wireguard.conf.j2    # Rsyslog rules configuration template
│   ├── systemd/
│   │   ├── wireguard-status.service.j2 # Systemd status snapshot service template
│   │   └── wireguard-status.timer.j2   # Systemd status snapshot timer template
│   └── wireguard/
│       └── wg.conf.j2           # Main WireGuard interface configuration template
└── vars/
    ├── main.yml                 # Internal constants
    └── debian.yml               # Debian/Ubuntu OS package definitions
```

---

## 🏷️ Tags

Use `--tags` to run selective parts of the role.

| Tag | Description |
|-----|-------------|
| `wireguard_assert` | Run variable assertion ladder |
| `wireguard_prerequisites` | Install system update prerequisites |
| `wireguard_install` | Install WireGuard package binaries |
| `wireguard_keys` | Manage host cryptographic keys and peer PSKs |
| `wireguard_mesh` | Discover mesh topology and peer endpoints |
| `wireguard_configure` | Render interface configuration files |
| `logging` | Configure rsyslog routing, kernel dyndbg, logrotate, and status timer |
| `wireguard_service` | Manage systemd service state |
| `wireguard_remove` | Uninstall WireGuard configuration, logs, and packages |

## CI/CD Pipeline

This repository uses centralized, reusable GitHub Actions workflows from [github-workflows](https://github.com/grzegorzfranus/github-workflows) (`@main`) for quality assurance, security scanning, and release automation.

### CI Pipeline (`ansible-ci.yml`)

Runs on every Pull Request in a two-tier gate pattern:

1. **Branch Name Lint** — enforces naming conventions (`feature/`, `bugfix/`, `fix/`, `hotfix/`, `release/`, `chore/`, `docs/`, `refactor/`, `test/`, `build/`, `ci/`, `perf/`, `revert/`)
2. **PR Title Lint** — enforces [Conventional Commits](https://www.conventionalcommits.org/) format (`feat:`, `fix:`, `ci:`, etc.)
3. **YAML Syntax Lint** — validates YAML formatting via `yamllint`
4. **Ansible Lint** — checks Ansible best practices and role standards
5. **Galaxy Metadata Validation** — verifies `meta/main.yml` schema and requirements (`ansible-meta-validate.yml`)
6. **Security Scanning** — TruffleHog secret detection and Trivy IaC scanning (`ansible-security.yml`)
7. **Molecule Integration Tests** — executes Molecule test matrix (`default` and `logging` scenarios) across supported distros (`ansible-molecule.yml`)
8. **Merge Check Gate** — single authoritative status check aggregating all results for branch protection

### Release & Publish Pipeline (`ansible-publish.yml`)

Automated via [Release Please](https://github.com/googleapis/release-please):

1. **Push to `main`** → Release Please creates or updates a Release PR with automated changelog generation
2. **Release PR Validation** → validates YAML syntax and actions schema before setting `Merge Check` status
3. **Merge Release PR** → creates Git version tag and GitHub Release automatically
4. **Ansible Galaxy Publish** → publishes tagged release to Ansible Galaxy via `ansible-publish.yml`

## Example Playbooks

### Dedicated Logging Enabled Playbook

```yaml
---
- name: Deploy WireGuard Full-Mesh VPN with Dedicated Logging
  hosts: wireguard
  become: true
  vars:
    wireguard_mesh_group: "wireguard"
    wireguard_configure_logging: true
    wireguard_install_rsyslog: true
    wireguard_enable_kernel_debug_logging: true
    wireguard_enable_status_snapshot: true
    wireguard_status_snapshot_interval: "5min"
    wireguard_logrotate_options:
      enabled: true
      frequency: "daily"
      rotate_count: 30
      compress: true
      notifempty: true
      copytruncate: true
      dateext: true
      dateformat: "-%Y%m%d"
      olddir: "/var/log/archive/wireguard"
  roles:
    - role: grzegorzfranus.wireguard
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