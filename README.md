# Ansible Role: WireGuard

|Source|Version|CI|License|
|------|-------|-------|-------|
|[![Source Code](https://img.shields.io/badge/source-github-blue.svg)](https://github.com/grzegorzfranus/ansible-role-wireguard)|[![Version](https://img.shields.io/github/v/release/grzegorzfranus/ansible-role-wireguard)](https://github.com/grzegorzfranus/ansible-role-wireguard/releases)|[![tests](https://github.com/grzegorzfranus/ansible-role-wireguard/actions/workflows/test-and-validation.yml/badge.svg)](https://github.com/grzegorzfranus/ansible-role-wireguard/actions)|[![Repository License](https://img.shields.io/badge/license-apache2.0-brightgreen.svg)](LICENSE)|

This Ansible role configures a full mesh WireGuard VPN between Ubuntu and Debian servers with comprehensive logging and security features. Perfect for creating secure networks between multiple servers with automatic key management and mesh topology setup.

## ✨ Features

- 🌐 **Full Mesh Network**: Every server connects to every other server
- 🔐 **Automatic Key Management**: Generates and distributes WireGuard keys
- 🛡️ **Security Configuration**: Secure file permissions and configuration backup
- 🚀 **Service Management**: Handles systemd service configuration
- 🧪 **Connectivity Testing**: Tests mesh connectivity after setup
- 📊 **Status Monitoring**: Provides detailed status information
- 📝 **Dedicated Logging**: Configures separate log files for WireGuard
- 🧪 **Container Testing**: Comprehensive Molecule test suite for CI/CD

## 🎯 Architecture

The role creates a full mesh topology where each server in the `wireguardservers` inventory group:

- Gets a sequential IP address from `10.0.0.0/24` network
- Connects to all other servers in the group
- Maintains persistent connections with keepalive
- Supports any number of servers (no artificial limits)

```
Server 1 (10.0.0.1) ←→ Server 2 (10.0.0.2)
        ↕                    ↕
        └──── Server 3 (10.0.0.3)
```

## 📋 Requirements

- **Ansible**: 2.15 or higher (tested with Ansible core 2.20+)
- **Kernel**: 5.6+ (for native WireGuard support)
- **Network**: All servers must be able to reach each other on UDP port 51820
- **Privileges**: sudo/root access on target hosts
- **Execution**: run the play against the entire `wireguardservers` group; single-host runs are not supported

**Collections**: This role requires the following Ansible collections:
- `ansible.posix` (>= 1.5.0) - for system configuration (sysctl)
- `community.general` (>= 8.0.0) - for kernel module management (modprobe)

Install collections using:
```bash
ansible-galaxy collection install -r meta/requirements.yml
```

**Note**: Version 1.4.9+ is fully compatible with Ansible core 2.20 and its stricter boolean conditional requirements.

### Supported operating systems
List of officially supported operating systems:
| OS Family | Version | Status |
|-----------|---------|---------|
| Ubuntu | 24.04 (Noble) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Ubuntu | 22.04 (Jammy) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |
| Debian | 12 (Bookworm) | ![✓](https://img.shields.io/badge/✓-brightgreen.svg) |

## 🚀 Quick Start

### 1. Add servers to inventory

```ini
[wireguardservers]
server1 ansible_host=192.168.1.10
server2 ansible_host=192.168.1.11
server3 ansible_host=192.168.1.12
```

### 2. Create playbook

```yaml
---
- name: Configure WireGuard Mesh VPN
  hosts: wireguardservers
  become: true
  roles:
    - ansible-role-wireguard
```

### 3. Run the playbook

```bash
ansible-playbook -i inventory wireguard-mesh.yml
```

## ⚙️ Configuration

### Default Variables

The role comes with sensible defaults that work out of the box:

```yaml
# Network configuration
wireguard_network_base: "10.0.0.0/24"
wireguard_port: 51820

# Service settings
wireguard_service_enabled: true
wireguard_interface: "wg0"

# Security
wireguard_generate_keys: true
```

### Advanced Configuration

You can override any variable in your playbook:

```yaml
---
- name: Configure WireGuard Mesh VPN
  hosts: wireguardservers
  become: true
  vars:
    wireguard_network_base: "172.16.0.0/24"
    wireguard_port: 51821
    wireguard_persistent_keepalive: 30
    # Optional: set public endpoint per host behind NAT
    # wireguard_endpoint: "vpn.example.com"
  roles:
    - ansible-role-wireguard
```

## 📊 Variables

### General Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_role_action` | Role execution control (all/install/configure) | `"all"` |
| `wireguard_inventory_group` | Inventory group name containing WireGuard servers | `"wireguardservers"` |

### Network Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_network_base` | Base network for mesh topology (CIDR notation) | `"10.0.0.0/24"` |
| `wireguard_network_prefix` | Network prefix for IP assignment | `"10.0.0"` |
| `wireguard_port` | UDP port for WireGuard communication | `51820` |
| `wireguard_persistent_keepalive` | Keepalive interval in seconds for NAT traversal | `25` |
| `wireguard_dns_servers` | DNS servers for VPN interface (list) | `["1.1.1.1", "8.8.8.8"]` |
| `wireguard_configure_dns` | Enable DNS configuration in interface | `false` |
| `wireguard_endpoint` | Optional public endpoint (IP/DNS) per host | `""` |

### Interface Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_interface` | WireGuard interface name | `"wg0"` |
| `wireguard_mtu` | MTU size for WireGuard interface | `1420` |
| `wireguard_enable_ip_forwarding` | Enable IP forwarding for mesh traffic | `true` |

### Security Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_generate_keys` | Automatically generate private/public key pairs | `true` |
| `wireguard_config_dir` | Directory for WireGuard configuration files | `"/etc/wireguard"` |
| `wireguard_private_key_permissions` | File permissions for private keys | `"0600"` |
| `wireguard_public_key_permissions` | File permissions for public keys | `"0644"` |
| `wireguard_config_permissions` | File permissions for configuration files | `"0600"` |
| `wireguard_backup_config` | Create backup of existing configuration | `true` |
| `wireguard_enable_psk` | Enable Pre-Shared Key for quantum-resistant security | `true` |
| `wireguard_generate_psk` | Automatically generate Pre-Shared Keys | `true` |
| `wireguard_psk_permissions` | File permissions for PSK files | `"0600"` |

### Service Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_service_enabled` | Enable WireGuard service on boot | `true` |
| `wireguard_service_name` | SystemD service name (auto-constructed) | `"wg-quick@wg0"` |
| `wireguard_restart_on_change` | Restart service when configuration changes | `true` |
| `wireguard_enable_service` | Enable service in systemd | `true` |
| `wireguard_start_service` | Start service immediately after configuration | `true` |
| `wireguard_service_timeout` | Timeout for service operations (seconds) | `30` |

### Logging Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_enable_logging` | Enable dedicated WireGuard logging system | `true` |
| `wireguard_logging_backend` | Logging backend: `rsyslog` or `journald` | `"rsyslog"` |
| `wireguard_log_dir` | Directory path for WireGuard log files | `"/var/log/wireguard"` |
| `wireguard_log_file` | Full path to main WireGuard log file | `"/var/log/wireguard/wireguard.log"` |
| `wireguard_log_file_permissions` | File permissions for log files | `"0644"` |
| `wireguard_log_dir_permissions` | Directory permissions for log directory | `"0755"` |
| `wireguard_syslog_identifier` | Syslog program identifier for log entries | `"wireguard"` |
| `wireguard_rsyslog_config_file` | Filename for RSyslog configuration | `"30-wireguard.conf"` |

### Log Rotation Settings

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_configure_logrotate` | Enable automatic log rotation | `true` |
| `wireguard_logrotate_options.frequency` | Log rotation frequency (daily/weekly/monthly) | `"daily"` |
| `wireguard_logrotate_options.count` | Number of rotated log files to keep | `7` |
| `wireguard_logrotate_options.missingok` | Don't error if log file is missing | `true` |
| `wireguard_logrotate_options.notifempty` | Skip rotation if log file is empty | `true` |
| `wireguard_logrotate_options.compress` | Compress rotated log files | `true` |
| `wireguard_logrotate_options.delaycompress` | Delay compression until next rotation | `true` |
| `wireguard_logrotate_options.create` | Create new empty log file after rotation | `true` |
| `wireguard_logrotate_options.create_mode` | File permissions for new log files | `"0640"` |
| `wireguard_logrotate_options.create_owner` | Owner of newly created log files (OS-specific: Ubuntu=`syslog`, Debian=`root`) | OS-dependent |
| `wireguard_logrotate_options.create_group` | Group of newly created log files (OS-specific: Ubuntu=`adm`, Debian=`root`) | OS-dependent |
| `wireguard_logrotate_options.dateext` | Add date extension to archived files | `true` |
| `wireguard_logrotate_options.archive_directory_path` | Directory for archived log files | `"/var/log/wireguard/archived"` |

### Testing Configuration

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_molecule_test_mode` | Enable container-compatible testing mode (internal variable) | `false` |

### Debugging

| Variable | Description | Default |
|----------|-------------|---------|
| `wireguard_debug` | Enable additional debug output in role tasks (no secrets) | `false` |

**Note**: The `wireguard_molecule_test_mode` variable is used internally by Molecule tests to skip operations that don't work in containers (kernel modules, systemd services, etc.). This should not be modified in production environments.

### Security Notes

- Keys are generated on-host and sensitive tasks are protected with `no_log: true`.
- Public keys may be stored in host-local facts for discovery; PSKs are handled only at runtime.
- Prefer setting `wireguard_endpoint` when hosts are behind NAT or require DNS endpoints.

### Container Compatibility

When running in container environments (like Docker for testing), the role automatically:

- ✅ **Skips rsyslog installation** - Package installation is bypassed in test mode
- ✅ **Validates directory existence** - Checks for `/etc/rsyslog.d` before configuration
- ✅ **Creates missing directories** - Automatically creates required directories when possible
- ✅ **Graceful degradation** - Provides clear messages when features are unavailable
- ✅ **Handler protection** - Prevents systemd service operations in containers
- ✅ **Container-aware verification** - Test verification adapts to container limitations

This ensures reliable testing across different environments while maintaining full functionality in production systems.

### OS-Specific Variables

The following variables are automatically set based on the operating system and should not be overridden:

| Variable | Description | Ubuntu Value | Debian Value |
|----------|-------------|--------------|--------------|
| `wireguard_log_user` | User for log file ownership | `syslog` | `root` |
| `wireguard_log_group` | Group for log file ownership | `adm` | `root` |

## 🔍 Verification

After deployment, you can verify the mesh is working:

### Check WireGuard Status

```bash
sudo wg show
```

### Test Connectivity

```bash
# From any server, ping the others
ping 10.0.0.1  # Server 1
ping 10.0.0.2  # Server 2
ping 10.0.0.3  # Server 3
```

### Check Service Status

```bash
sudo systemctl status wg-quick@wg0
```

### Check Logs

```bash
# View WireGuard logs
sudo tail -f /var/log/wireguard/wireguard.log

# Check recent log entries
sudo journalctl -u wg-quick@wg0 -f

# View systemd service logs
sudo systemctl status wg-quick@wg0
```

## 🎯 IP Address Assignment

Servers get IP addresses based on their position in the inventory:

- First server: `10.0.0.1`
- Second server: `10.0.0.2`
- Third server: `10.0.0.3`
- Fourth server: `10.0.0.4`
- Fifth server: `10.0.0.5`
- ...and so on for additional servers

## 🛡️ Security Features

- ✅ Automatic private/public key generation
- ✅ **Pre-Shared Key (PSK) support** for quantum-resistant security
- ✅ Secure key file permissions (600)
- ✅ Configuration backup before changes
- ✅ Persistent connection monitoring
- ✅ Minimal AllowedIPs (/32) for each peer
- ⚠️ Manual iptables configuration required (see below)

### 🔐 Enhanced Security with Pre-Shared Keys (PSK)

This role implements **Pre-Shared Key (PSK)** support with cluster-wide consistency and runtime-only handling:

- **Defense in Depth**: Combines asymmetric (public key) + symmetric (PSK) cryptography
- **Cluster-wide consolidation**: Discovers existing PSKs across hosts; if none or divergent, generates one new PSK and distributes it runtime-only
- **Idempotent**: If a consistent PSK exists everywhere, subsequent runs make no changes
- **Runtime-only**: PSK is not stored on disk or in vault by this role; it is injected as an in-memory fact during execution and used by templates
- **Backward compatibility**: If a legacy PSK exists (persistent fact or `{{ wireguard_config_dir }}/psk` file), it will be detected and reused when consistent

Behavior rules when PSK is enabled (`wireguard_enable_psk: true`):

- No PSK found anywhere → a new PSK is generated with `wg genpsk` and distributed
- Exactly one unique PSK found across hosts → reused and distributed to missing hosts
- Multiple differing PSKs found → a new PSK is generated and used across all hosts

Check mode (`--check`):

- If generation would be required (none or divergent), the role fails fast with a clear message; rerun without `--check` to generate

## 🛡️ Manual Iptables Configuration

**Important**: This role does not configure iptables automatically. You must manually add the required firewall rules on each server to allow WireGuard traffic.

### Required Iptables Rules

After deploying the WireGuard mesh, configure the following iptables rules on **each server**:

```bash
# 1. Allow WireGuard UDP traffic on port 51820
sudo iptables -A INPUT -p udp --dport 51820 -j ACCEPT

# 2. Allow forwarding traffic through WireGuard interface
sudo iptables -A FORWARD -i wg0 -j ACCEPT
sudo iptables -A FORWARD -o wg0 -j ACCEPT

# 3. Save the rules (install iptables-persistent if needed)
sudo apt install iptables-persistent
sudo netfilter-persistent save
```

### Alternative: Using UFW

If you prefer UFW (Uncomplicated Firewall):

```bash
# Allow WireGuard port
sudo ufw allow 51820/udp

# Enable IP forwarding in UFW
sudo ufw route allow in on wg0
sudo ufw route allow out on wg0

# Enable UFW if not already enabled
sudo ufw enable
```

### Verification

Check that the rules are active:

```bash
# Check iptables rules
sudo iptables -L | grep -E "(51820|wg0)"

# Check UFW status
sudo ufw status

# Test WireGuard connectivity
ping 10.0.0.X  # Replace X with peer IP
```

## 🔧 Troubleshooting

### WireGuard service won't start

```bash
# Check configuration
sudo wg-quick up wg0

# Check systemd logs
sudo journalctl -u wg-quick@wg0

# Check dedicated WireGuard logs
sudo tail -f /var/log/wireguard/wireguard.log
```

### Connectivity issues

```bash
# Check interface
ip addr show wg0

# Check routes
ip route show table all | grep wg0

# Check firewall rules (see Manual Iptables Configuration section)
sudo iptables -L | grep -E "(51820|wg0)"
```

### Key issues

```bash
# Regenerate all keys (private/public keys and PSK)
sudo rm /etc/wireguard/privatekey /etc/wireguard/publickey
sudo rm /etc/wireguard/psk  # If PSK is enabled
# Re-run the playbook

# Check if PSK is properly configured
sudo cat /etc/wireguard/psk
```

### Deprecation warnings from collections

If you see warnings like `Importing 'to_native' from 'ansible.module_utils._text' is deprecated`, this is from older versions of the `ansible.posix` or `community.general` collections:

```bash
# Update all required collections to latest versions
ansible-galaxy collection install -r meta/requirements.yml --upgrade

# Verify installed collection versions
ansible-galaxy collection list | grep -E "ansible.posix|community.general"

# Expected versions:
# - ansible.posix: >= 1.5.0
# - community.general: >= 8.0.0
```

## 📁 File Structure

```
ansible-role-wireguard/
├── .github/                  # GitHub Actions workflows
│   └── workflows/           # CI/CD automation
│       ├── test-and-validation.yml # 🧪 Testing and validation workflow
│       └── publish-to-galaxy.yml # 📦 Ansible Galaxy publishing workflow
├── CHANGELOG.md              # Version history and changes
├── LICENSE                   # Apache-2.0 license
├── README.md                # This documentation file
├── defaults/
│   └── main.yml             # Default configuration variables
├── handlers/
│   └── main.yml             # Service restart and reload handlers
├── meta/
│   ├── main.yml             # Role metadata and Galaxy information
│   └── requirements.yml     # Ansible collections requirements and versions
├── molecule/                 # Molecule testing framework
│   └── default/             # Default test scenario
│       ├── molecule.yml     # Test configuration
│       ├── converge.yml     # Role execution playbook
│       ├── prepare.yml      # Test preparation tasks
│       └── verify.yml       # Verification tests
├── tasks/
│   ├── main.yml             # Main task orchestration and flow control
│   ├── assert.yml           # Variable validation and OS compatibility checks
│   ├── prerequisites.yml    # System preparation and kernel module loading
│   ├── install.yml          # Package installation and verification
│   ├── keys.yml             # Cryptographic key generation and management
│   ├── configure.yml        # WireGuard configuration file generation
│   ├── service.yml          # Service management and connectivity testing
│   └── logging.yml          # Logging system configuration (rsyslog, logrotate)
├── templates/
│   ├── wireguard.conf.j2    # Main WireGuard configuration template
│   ├── logrotate/
│   │   └── wireguard.j2     # Log rotation configuration template
│   └── rsyslog/
│       └── wireguard.conf.j2 # RSyslog configuration template
└── vars/
    ├── main.yml             # Internal role variables and constants
    ├── ubuntu_22.04.yml     # Ubuntu 22.04 LTS specific variables
    ├── ubuntu_24.04.yml     # Ubuntu 24.04 LTS specific variables
    └── debian_12.yml        # Debian 12 specific variables
```

## 🏷️ Tags

- `always` - Tasks that always run (variable loading and validation)
- `setup` - Setup tasks including OS-specific variables, requirements, installation, and configuration
- `init` - Initial setup tasks
- `validate` - Variable validation tasks
- `check` - Validation and verification tasks
- `requirements` - System requirements verification
- `install` - Package installation tasks
- `configure` - Service configuration tasks
- `config` - Configuration related tasks (alias; prefer `configure`)
- `logrotate` - Log rotation configuration tasks
- `keys` - Key generation and management tasks
- `service` - Service management and startup tasks
- `logging` - Logging configuration tasks
- `test` - Testing and verification tasks
- `verify` - Verification tasks

## 🧪 Testing

This role includes comprehensive Molecule tests that validate functionality across all supported operating systems:

- **Ubuntu 22.04 LTS** - Complete functionality testing
- **Ubuntu 24.04 LTS** - Complete functionality testing  
- **Debian 12** - Complete functionality testing

### Running Tests Locally

```bash
# Install testing dependencies
pip install molecule molecule-plugins[docker] ansible-lint

# Run all tests
molecule test

# Run tests for specific scenario
molecule test --scenario-name default

# Test only linting
molecule lint
```

## 🔧 CI/CD Integration

This role includes comprehensive GitHub Actions workflows for automated testing and deployment:

### Testing Pipeline 🧪
- **Workflow**: `.github/workflows/test-and-validation.yml`
- **Name**: `🧪 Test & Validation Pipeline`
- **Purpose**: Automated testing across multiple platforms
- **Triggers**: Push to main branch, pull requests
- **Features**:
  - Multi-platform testing (Ubuntu 22.04, 24.04, Debian 12)
  - Ansible lint validation
  - Molecule test execution
  - Cross-platform compatibility verification

### Galaxy Publishing 📦
- **Workflow**: `.github/workflows/publish-to-galaxy.yml`
- **Name**: `📦 Publish to Ansible Galaxy`
- **Purpose**: Automated role publishing to Ansible Galaxy
- **Triggers**: Tagged releases (v*)
- **Features**:
  - Automated version detection
  - Quality assurance checks
  - Secure deployment to Galaxy

### Professional Workflow Organization 🚀
Both workflows feature:
- **Emoji Indicators**: Clear visual status representation
- **Professional Naming**: Descriptive and semantic job/step names
- **Consistent Structure**: Standardized approach across all CI/CD operations
- **Enhanced Readability**: Improved maintainability and understanding

### Test Quality Assurance 🔍

The role has been extensively tested for:
- ✅ **Template Stability**: All Jinja2 templates generate identical output across multiple runs
- ✅ **Idempotent Operations**: No spurious changes reported on subsequent playbook runs
- ✅ **Container Compatibility**: Full testing in Docker containers without host dependencies
- ✅ **Variable Safety**: No undefined variable references in any execution path
- ✅ **Error Handling**: Graceful degradation in restricted environments

## 🤝 Contributing

Contributions, bug reports, and feature requests are welcome!

- Fork the repository and create your branch from `main`.
- Make your changes with clear, descriptive commit messages.
- Ensure your code passes all Molecule and lint tests.
- Submit a pull request describing your changes and the motivation.
- For major changes, please open an issue first to discuss what you would like to change.

If you have questions or suggestions, feel free to open an issue or contact the author via GitHub.

## 📝 License

This project is licensed under the Apache-2.0 License - see the LICENSE file for details.

## 👥 Author Information

This role was created by [Grzegorz Franus](https://github.com/grzegorzfranus).

---

**⚠️ Important Notes:**
- This role supports Ubuntu 22.04, 24.04 LTS and Debian 12
- Supports any number of servers in the mesh (no artificial limits)
- All servers must be able to reach each other on the configured port
- **Manual iptables configuration required** - see the "Manual Iptables Configuration" section above
- Ensure proper firewall rules are configured before testing connectivity
