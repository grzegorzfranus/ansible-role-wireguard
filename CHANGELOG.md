# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.4.8] - 2025-09-03

### Added ✅
- Comprehensive variable validation for all role parameters in assert.yml
- Debugging section in README with wireguard_debug documentation

### Fixed 🔧
- Respect wireguard_generate_keys setting with proper failure handling
- Use wireguard_service_timeout for service operations
- Missing type checks and range validations for network, service, and logging parameters

## [1.4.8] - 2025-09-03

### Added ✅
- Cluster-wide PSK discovery and consolidation with runtime-only distribution

### Changed 🔄
- Stop persisting PSK to files and host-local facts; use in-memory facts and template injection

### Fixed 🔧
- Idempotency for PSK handling across mesh nodes, including consistent reuse and divergence resolution

## [1.4.7] - 2025-09-01

### Fixed 🔧
- Peer public key discovery during configuration: switched from `ansible_local` lookup to in-memory facts (`hostvars[*].wireguard_local_public_key`) and guarded undefined peers to prevent failures when running the full group.

### Changed 🔄
- RSyslog routing template now captures wg-quick unit logs by matching `programname == 'wg-quick'` and `syslogtag` starting with `wg-quick@{{ wireguard_interface }}` in addition to the configured `wireguard_syslog_identifier`.
- README requirements: clarified that the role must be executed against the entire `wireguardservers` group (single-host runs are not supported).

## [1.4.6] - 2025-08-11

### Fixed 🔧
- Hardened secrets handling in `tasks/keys.yml` with `no_log: true` on private key/PSK generation, slurp, and fact-setting tasks.
- Secured on-host facts file permissions to `0600` for `/etc/ansible/facts.d/wireguard.fact`.

### Changed 🔄
- Documentation consistency: clarified role action naming to prefer `configure` over `config` in `README.md` and references.
- Minor wording adjustments in docs for clarity.

### Added ✅
- Endpoint configurability via `wireguard_endpoint` for NAT/DNS use-cases.
- Logging backend selection with `wireguard_logging_backend` (`rsyslog`|`journald`).
- Check-mode safety for package, sysctl, module load, templating, and service operations.
- Port conflict validation using `ss` during assertions.
- Kernel minimum validation now uses per-OS `wireguard_minimum_kernel_version`.
- Backup logic fixed to use a proper `stat` check for existing config.
- Readme updates: endpoint usage, logging backend, and security notes.

## [1.4.5] - 2025-06-24

### Changed 🔄
- **Professional Workflow Naming** - Renamed `galaxy.yml` to `publish-to-galaxy.yml` for better clarity
- **CI Pipeline Rebranding** - Renamed `ci.yml` to `test-and-validation.yml` with professional naming
- **Enhanced CI/CD Presentation** - Added emoji indicators throughout all GitHub Actions workflows
- **Improved Job Organization** - Renamed workflow jobs for better semantic meaning and clarity
- **Step Name Clarity** - Enhanced step descriptions with more specific and professional language

### Fixed 🔧
- **CI Badge Reference** - Updated README.md CI status badge to point to renamed `test-and-validation.yml` workflow
- **Documentation Consistency** - Fixed broken badge link that was still referencing old `ci.yml` workflow file

### Workflow Improvements 🚀
- **Galaxy Publishing**: Changed to `📦 Publish to Ansible Galaxy` with descriptive emoji
- **Testing Pipeline**: Updated to `🧪 Test & Validation Pipeline` for professional appearance
- **Job Naming**: Consistent professional naming across all workflows
- **Step Emojis**: Added relevant emoji to all workflow steps:
  - 📥 Check out the codebase
  - 🐍 Set up Python 3
  - 📦 Install dependencies / Install Ansible dependencies
  - 🔍 Lint code
  - 🧪 Molecule test
  - 🚀 Upload role to Ansible Galaxy / Run molecule test

### Repository Organization 📁
- **Workflow Consolidation**: Two professionally named workflows for complete CI/CD pipeline
- **Consistency**: Aligned all workflow naming and emoji usage for cohesive presentation
- **Documentation**: Enhanced workflow readability and maintenance across the board
- **Badge Accuracy**: Ensured CI status badges reflect current workflow structure

## [1.4.4] - 2024-06-23

### Fixed 🔧
- **Template Idempotence Issues** - Complete resolution of Molecule test failures caused by dynamic template content
- **Ansible Managed Comments** - Removed timestamp-based `ansible_managed` comments from all templates
- **Backup File Creation** - Disabled backup files in test mode to prevent timestamp-related idempotence failures
- **Undefined Variable Error** - Fixed `rsyslog_dir_check` undefined variable in logging status display

### Template Improvements 📝
- **Static Comments**: Replaced `{{ ansible_managed | comment }}` with static comments in all templates
- **RSyslog Template**: `templates/rsyslog/wireguard.conf.j2` now uses static header
- **Logrotate Template**: `templates/logrotate/wireguard.j2` now uses static header
- **Backup Control**: Added conditional `backup: false` in test mode for all template tasks

### Logic Simplification 🎯
- **RSyslog Configuration**: Simplified logic to always ensure `/etc/rsyslog.d` directory exists
- **Conditional Complexity**: Removed complex conditional logic that caused inconsistent behavior between runs
- **Container Compatibility**: Maintained full container support while ensuring consistent behavior
- **Status Display**: Updated logging status display to use available variables only

### Testing Reliability 🧪
- **Idempotence Tests**: All Molecule tests now pass idempotence verification consistently
- **YAML Validation**: Fixed indentation and syntax errors in `molecule/default/verify.yml`
- **Template Stability**: Templates generate identical output across multiple runs
- **Variable Safety**: Eliminated undefined variable references in all tasks

### Technical Details 🔧
- **Template Headers**: All templates now use static "managed by Ansible" comments
- **Backup Management**: `backup: "{{ not (wireguard_molecule_test_mode | default(false)) }}"` pattern applied
- **Directory Creation**: Always ensure required directories exist before configuration
- **Error Prevention**: Comprehensive variable validation and safe defaults

## [1.4.3] - 2024-06-23

### Fixed 🔧
- **RSyslog Container Compatibility** - Complete resolution of rsyslog configuration issues in Docker containers
- **Missing Package Dependencies** - Automatic rsyslog installation in production environments
- **Directory Existence Validation** - Proper checking for `/etc/rsyslog.d` directory before configuration
- **Handler Container Safety** - Protection of systemd handlers from execution in container environments

### Container Environment Improvements 🐳
- **RSyslog Installation**: Added automatic rsyslog package installation (skipped in test mode)
- **Directory Creation**: Automatic creation of `/etc/rsyslog.d` directory when missing
- **Graceful Degradation**: Proper skipping of rsyslog configuration when not available in containers
- **Test Mode Safety**: All handlers now respect `wireguard_molecule_test_mode` variable

### Handler Enhancements 🔄
- **Container Protection**: Added `wireguard_molecule_test_mode` conditions to all handlers
- **Service Safety**: Skip `restart wireguard`, `reload systemd`, and `restart rsyslog` in test environments
- **Error Prevention**: Eliminated handler execution failures in Docker containers

### Logging Configuration Improvements 📝
- **Conditional Configuration**: RSyslog setup only when directory exists and not in test mode
- **Enhanced Validation**: Comprehensive directory and service availability checks
- **Clear Messaging**: Informative debug messages when skipping configuration in containers
- **SystemD Override Protection**: Skip systemd service overrides in container environments

### Testing Reliability 🧪
- **Verification Improvements**: Container-aware verification tasks in `verify.yml`
- **Conditional Assertions**: RSyslog verification only when applicable
- **Debug Messaging**: Clear indication when features are skipped in container environment
- **YAML Quality**: Fixed all trailing spaces and formatting issues

## [1.4.2] - 2024-06-23

### Fixed 🔧
- **Molecule Test Idempotence Issues** - Complete resolution of test stability problems
- **Cross-Platform Logging Configuration** - OS-specific syslog user/group handling
- **Container Testing Compatibility** - Comprehensive container environment support
- **Network Validation Dependencies** - Removed external library requirements

### Testing Infrastructure Improvements 🧪
- **Idempotence Fixes**: Removed timestamp from WireGuard configuration template
- **Debug Task Stability**: Added `changed_when: false` to all debug and display tasks
- **Backup Management**: Disabled configuration backups in test mode to prevent timestamp issues
- **Key File Verification**: Fixed file path verification to match actual generated filenames

### Cross-Platform Compatibility 🌐
- **Debian 12 Support**: Added proper syslog user/group configuration (`root:root`)
- **Ubuntu Support**: Maintained traditional syslog configuration (`syslog:adm`)
- **OS Version Validation**: Fixed Debian version checking to use major version (12 vs 12.11)
- **Fallback Configuration**: Added safe defaults for maximum compatibility

### Container Testing Enhancements 🐳
- **Molecule Test Mode**: Added `wireguard_molecule_test_mode` variable for container-specific behavior
- **Kernel Module Handling**: Skip `modprobe` operations in container environments
- **Service Management**: Bypass systemd operations that fail in containers
- **Network Configuration**: Skip IP forwarding and connectivity tests in test mode

### Network Validation Improvements 🔧
- **Dependency Removal**: Replaced `ansible.utils.ipaddr` with regex-based validation
- **Library Independence**: Eliminated requirement for external `netaddr` Python library
- **Validation Robustness**: Enhanced error messages with format examples
- **Performance**: Faster validation using built-in Ansible functions

### Documentation Updates 📝
- **OS-Specific Variables**: Documented syslog user/group configuration per distribution
- **Test Mode Variables**: Added documentation for molecule test mode functionality
- **Validation Requirements**: Updated network validation requirements and examples

## [1.4.1] - 2024-06-23

### Added ✅
- **Comprehensive Molecule Test Suite** for automated role testing
- Complete test infrastructure based on ansible-role-chrony patterns
- Multi-platform testing support (Ubuntu 22.04, 24.04, Debian 12)
- Enhanced CI/CD pipeline with status emojis following repository standards

### Testing Infrastructure 🧪
- **molecule.yml**: Docker-based testing configuration with privileged containers
- **converge.yml**: Role execution playbook with test-specific variables
- **prepare.yml**: System preparation including file permissions and kernel module checks
- **verify.yml**: Comprehensive verification of installation, configuration, and security

### Verification Coverage 🔍
- **Package Installation**: WireGuard packages and binary verification
- **Configuration Files**: Interface configs, directory structure, and permissions
- **Cryptographic Security**: Private/public key generation and PSK validation
- **Logging Setup**: Rsyslog integration and log rotation configuration
- **Service Management**: SystemD service configuration verification
- **Content Validation**: WireGuard configuration structure and required sections

### CI/CD Enhancements 🚀
- Updated GitHub Actions workflow with status emojis
- Enhanced job and step naming with appropriate emoji indicators
- Maintained compatibility with existing CI pipeline and matrix strategy
- Added verification emojis (🔍, 🧪, ✅, ❌) throughout test tasks

### Documentation 📝
- Comprehensive test file headers with execution flow documentation
- Enhanced task descriptions with emoji status indicators
- Following repository English-only policy and emoji requirements

## [1.4.0] - 2025-06-23

### Added ✅
- **Pre-Shared Key (PSK) Support** for enhanced quantum-resistant security
- Automatic PSK generation using `wg genpsk` on each server
- PSK configuration variables: `wireguard_enable_psk`, `wireguard_generate_psk`, `wireguard_psk_permissions`
- Enhanced WireGuard configuration template with conditional PSK support
- PSK validation in assert tasks with proper security checks
- Comprehensive PSK documentation in README.md with security benefits

### Security Enhancements 🔐
- **Quantum-Resistant Encryption**: PSK adds symmetric encryption layer on top of public key cryptography
- **Defense in Depth**: Combined asymmetric + symmetric cryptography for enhanced security
- **Minimal Attack Surface**: Each peer uses `/32` AllowedIPs (single IP addresses only)
- **Secure PSK Management**: PSK files protected with 0600 permissions like private keys
- **Automatic PSK Distribution**: PSK automatically shared between mesh peers through Ansible facts

### Documentation Updates 📝
- Added comprehensive PSK section to README.md Security Features
- Updated Variables section with all PSK-related configuration options
- Enhanced troubleshooting guide with PSK-specific commands
- Added quantum resistance and defense-in-depth explanations

### Technical Implementation 🛠️
- Extended `keys.yml` with PSK generation and management tasks
- Updated `wireguard.conf.j2` template with conditional PresharedKey entries
- Enhanced `assert.yml` with PSK variable validation
- Modified Ansible facts storage to include PSK distribution
- Added proper conditional logic for PSK enablement/disablement

## [1.3.3] - 2025-06-23

### Fixed 🔧
- Removed invalid `delegate_to: localhost` attribute from `include_tasks` in main.yml
- Kept `run_once: true` for variable validation as it only needs to run once per playbook execution
- Fixed Ansible task execution error: 'delegate_to' is not a valid attribute for a TaskInclude

## [1.3.2] - 2025-06-23

### Changed 🔄
- Removed artificial 5-server limitation from the role
- Updated host capacity validation to use subnet limits (253 hosts for /24)
- Updated documentation examples to use 3 servers instead of 5
- Simplified mesh topology diagram for better clarity

### Documentation Updates 📝
- Updated Architecture section with simplified 3-server mesh diagram
- Changed inventory examples to show 3 servers configuration
- Updated IP Address Assignment section to indicate unlimited scaling
- Added note about subnet capacity (any number of servers within subnet range)
- Removed outdated references to 5-server maximum

### Technical Improvements 🔧
- Updated assert.yml to validate against subnet capacity (253 hosts) instead of arbitrary 5-host limit
- Updated defaults/main.yml comments to reflect unlimited server support
- Enhanced validation messages to show actual capacity limits
- Standardized `when:` statement format across all task files using `when: >` multiline syntax
- Improved code consistency and readability for conditional statements

## [1.3.1] - 2025-06-23

### Changed 🔄
- Updated licensing from MIT to Apache-2.0 license
- Changed company from "Local Infrastructure" to "EWARE"
- Updated minimum Ansible version from 2.12 to 2.15
- Updated author information to "Grzegorz Franus"
- Updated namespace to "grzegorzfranus"

### License & Meta Updates 📝
- Replaced complete LICENSE file with Apache-2.0 license text
- Updated all documentation references to Apache-2.0 license
- Updated badges in README.md to reflect new license and Ansible version
- Updated Galaxy metadata with new author and company information

### Documentation Updates 📝
- Restructured README.md header with table-based badges following chrony role pattern
- Updated title from "WireGuard Mesh VPN" to "WireGuard" for consistency
- Added "Supported operating systems" section with status table
- Enhanced role description with better feature highlights
- Improved documentation structure and visual consistency

### Variables Section Restructuring 📋
- Reorganized "Available Variables" into "Variables" section with categorized subsections
- Added "Network Configuration", "Service Configuration", "Logging Configuration", and "Log Rotation Settings" categories
- Enhanced variable descriptions with more detailed explanations
- Improved table formatting following chrony role patterns
- Added "Tags" section documenting all available Ansible tags for selective task execution
- Added emoji to all level 2 headers for improved visual consistency and navigation

### Complete Variables Documentation 📚
- Added comprehensive documentation for all variables from defaults/main.yml
- Documented all missing variables including General Settings, Interface Settings, and Security Settings
- Added complete Service Configuration section with all service management variables
- Extended Logging Configuration with file permissions and directory settings
- Completed Log Rotation Settings with all logrotate options (missingok, notifempty, delaycompress, etc.)
- Ensured all 35+ configurable variables are properly documented with descriptions and defaults

### File Structure Documentation Update 📁
- Updated File Structure section with complete and accurate directory tree
- Added all missing files and directories (CHANGELOG.md, LICENSE, logging.yml)
- Enhanced descriptions for all files with their specific purposes
- Added OS-specific variable files documentation (ubuntu_22.04.yml, ubuntu_24.04.yml, debian_12.yml)
- Included template subdirectories (logrotate/, rsyslog/) with their contents
- Fixed Features section to remove outdated "Iptables Integration" reference

## [1.3.0] - 2025-06-22

### Added ✅
- Extended operating system support to Ubuntu 22.04 LTS and Debian 12
- OS-specific variable files for each supported platform
- Comprehensive platform compatibility validation
- Enhanced platform support documentation

### Platform Support 🖥️
- Added Ubuntu 22.04 LTS (Jammy Jellyfish) support
- Added Debian 12 (Bookworm) support  
- Maintained Ubuntu 24.04 LTS (Noble Numbat) support
- Created OS-specific variable files:
  - `vars/ubuntu_22.04.yml` - Ubuntu 22.04 LTS configuration
  - `vars/ubuntu_24.04.yml` - Ubuntu 24.04 LTS configuration
  - `vars/debian_12.yml` - Debian 12 configuration

### Enhanced Validation 🧪
- Updated OS validation logic to support multiple distributions
- Enhanced error messages with supported OS information
- Improved platform detection and variable loading

### Documentation Updates 📝
- Updated README.md with multi-platform support information
- Enhanced Galaxy metadata with additional platform entries
- Updated badges to reflect extended OS support
- Modified role description to emphasize cross-platform compatibility

### Technical Details 🔧
- Kernel version requirements per platform (Ubuntu 22.04: 5.15+, Ubuntu 24.04: 6.8+, Debian 12: 6.1+)
- Consistent WireGuard package naming across all supported platforms
- Unified service naming convention for all distributions

## [1.2.9] - 2025-06-22

### Added ✅
- Complete emoji integration for all tasks in main.yml
- Enhanced task visibility with functional emoji indicators
- Improved role structure with cleaner organization

### Main Tasks Enhancement 🎯
- `main.yml`: Added emoji to all task names for better visual organization
  - 🧪 Validation tasks (OS check, variable validation)
  - 📂 Variable loading (OS-specific variables)
  - 🛠️ Prerequisites (system preparation)
  - 📦 Installation (package management)
  - 🔐 Key management (cryptographic operations)
  - 🌐 Configuration (service setup)
  - 📝 Logging (optional logging configuration)
  - 🚀 Service management (startup and monitoring)

### Removed ❌
- Removed `examples/` directory and all contained files
- Cleaned up Usage Examples section from README.md
- Updated CHANGELOG.md to remove outdated usage examples references

### Documentation Cleanup 📝
- Streamlined README.md by removing redundant examples section
- Focused documentation on configuration rather than basic usage examples
- Enhanced role clarity by removing unused example files

## [1.2.8] - 2025-06-22

### Added ✅
- Comprehensive emoji integration for all validation, debugging, and verification tasks
- Status indicators using emojis (🧪 for validation, ✅ for success, ❌ for errors, 🔍 for checks)
- Enhanced user experience with visual feedback for task execution status
- Improved readability of task output with contextual emojis

### Enhanced Tasks 🎯
- `assert.yml`: Added 🧪 emoji to all validation tasks with ✅/❌ status indicators
- `main.yml`: Enhanced validation tasks with 🧪 emoji for consistency
- `install.yml`: Added verification emojis (🧪 for checks, ✅ for success, 🔍 for searches)
- `keys.yml`: Added 🔍 for checks and ✅ for status display
- `prerequisites.yml`: Enhanced with 🔍 for checks, 🧪 for validation, ⚠️ for warnings
- `service.yml`: Added 🔍 for status checks, ✅ for displays, 🧪 for connectivity tests, 🌐 for network
- `logging.yml`: Enhanced with ✅ for status and 📝📄🔄📊 for different log components

### Visual Improvements 👁️
- Consistent emoji usage across all validation and debugging tasks
- Clear visual distinction between different types of operations
- Enhanced error and success message clarity with emoji indicators

## [1.2.7] - 2025-06-22

### Changed 🔄
- Completely restructured all task files according to chrony role patterns
- Enhanced task file headers with detailed descriptions and flow documentation
- Reorganized all tasks into logical numbered sections with clear separators
- Removed excessive emojis from task names (keeping only in validation tasks)
- Improved task naming conventions following standard patterns
- Added comprehensive comments explaining the purpose of each section

### Task Structure Improvements 📋
- `main.yml`: Added professional header with execution flow and tagged appropriately
- `assert.yml`: Restructured validation into 5 logical sections with clear boundaries
- `configure.yml`: Organized into network assignment, key collection, and deployment sections
- `install.yml`: Structured package installation with verification steps
- `keys.yml`: Improved cryptographic key management with proper flow documentation
- `prerequisites.yml`: Enhanced system preparation with validation sections
- `service.yml`: Organized service management with connectivity verification
- `logging.yml`: Structured logging configuration with rsyslog and rotation sections

### Documentation 📝
- Enhanced all task files with descriptive headers explaining purpose and flow
- Added section separators and comments throughout all task files
- Improved readability and maintainability of task organization

## [1.2.6] - 2025-06-22

### Changed 🔄
- Completely restructured `defaults/main.yml` with professional documentation format
- Enhanced all variable descriptions with detailed explanations and examples
- Added comprehensive comments explaining options, ranges, and best practices
- Improved section organization with clear headers and descriptions
- Restructured `vars/main.yml` with proper sectioning and internal variable documentation
- Aligned documentation style with industry best practices (based on chrony role)

### Documentation 📝
- Added detailed comments for all configuration options with usage examples
- Included value ranges, security considerations, and operational impact explanations
- Enhanced variable descriptions to include format specifications and recommendations
- Improved file structure with numbered sections and clear separators

## [1.2.5] - 2025-06-22

### Fixed 🔧
- Fixed handler dependencies in service tasks by adding proper variable checks
- Removed unused handlers (`reload wireguard`, `start wireguard`) to clean up code
- Corrected handler invocations to match available handlers
- Fixed potential issues with `wireguard_config_result` variable dependency

### Changed 🔄
- Fixed handler dependencies and organization
- Improved handler organization and removed redundant handlers
- Enhanced task conditions to prevent undefined variable errors

## [1.2.4] - 2025-06-22

### Changed 🔄
- Refactored rsyslog configuration to use Jinja2 template
- Updated rsyslog configuration from inline content to dedicated template file
- Added configurable rsyslog file name variable
- Improved maintainability and consistency across logging components

### Added ✅
- Template-based rsyslog configuration (`templates/rsyslog/wireguard.conf.j2`)
- Configurable rsyslog configuration file name
- Enhanced rsyslog template with proper documentation and ansible_managed header

### Technical Details 🔧
- RSyslog config: `/etc/rsyslog.d/30-wireguard.conf` (configurable)
- Template follows same patterns as logrotate template
- Improved separation of concerns between configuration and content

## [1.2.3] - 2025-06-22

### Added ✅
- Archive directory management with `olddir` option in logrotate
- Date extension for archived log files with `dateext` option
- Automatic creation of archive directory for rotated logs
- Enhanced logrotate template following chrony role patterns

### Features 📝
- Archived logs stored in dedicated directory (e.g., `/var/log/wireguard/archived`)
- Date-stamped log files (e.g., `wireguard.log.2024-12-19.gz`)
- Organized log archival with proper directory structure

## [1.2.2] - 2025-06-22

### Changed 🔄
- Simplified logrotate configuration to use standard options
- Changed default log retention from 30 to 7 days for better disk management
- Updated logrotate to use `create` instead of complex archive directory setup
- Improved postrotate command to use `systemctl reload rsyslog`
- Streamlined template to focus on essential logrotate features

### Features 📝
- Standard logrotate configuration: daily, 7 rotations, compress with delay
- Create new log file with permissions 0640 root:root
- Simple and maintainable configuration structure

## [1.2.1] - 2025-06-22

### Changed 🔄
- Refactored logrotate configuration to use Jinja2 template
- Updated logrotate variables structure to match role standards
- Improved logrotate configuration flexibility and maintainability
- Added archive directory creation for rotated logs

### Added ✅
- Template-based logrotate configuration (`templates/logrotate/wireguard.j2`)
- Structured logrotate options similar to other roles
- Archive directory management for rotated logs
- Enhanced logrotate customization examples in documentation

## [1.2.0] - 2025-06-22

### Added ✅
- Dedicated WireGuard logging configuration
- Systemd service override for centralized logging
- Custom rsyslog configuration for WireGuard logs
- Automatic log rotation with logrotate
- Log directory and file creation with proper permissions
- Comprehensive logging validation in assertions
- Enhanced documentation with logging verification steps

### Features 📝
- Logs stored in `/var/log/wireguard/wireguard.log` by default
- Configurable log rotation (size, retention, compression)
- Separate syslog identifier for easy filtering
- Integration with existing service management

## [1.1.0] - 2025-06-22

### Changed 🔄
- Removed automatic iptables configuration from the role
- Added comprehensive manual iptables configuration guide in README
- Simplified role focus to WireGuard installation and configuration only
- Updated documentation with detailed firewall setup instructions

### Removed ❌
- Automatic iptables rule configuration
- Iptables-related variables and tasks
- Firewall rule templates and handlers

## [1.0.0] - 2025-06-22

### Added ✅
- Initial release of WireGuard mesh VPN role
- Full mesh networking support for up to 5 Ubuntu 24.04 servers
- Automatic WireGuard installation and configuration
- Private/public key generation and management
- Iptables firewall integration with automatic rule configuration
- Systemd service management with automatic startup
- Mesh connectivity testing and verification
- Comprehensive variable validation and error handling
- Support for custom network ranges and ports
- Configuration backup before changes
- Detailed logging and status reporting with emojis
- English-only documentation and comments
- Role action control (all/install/configure modes)

### Security Features 🔐
- Secure key file permissions (600 for private, 644 for public)
- Automatic iptables rule management
- IP forwarding configuration
- Persistent keepalive for stable connections
- Protected configuration directory structure

### Infrastructure Features 🛠️
- Ubuntu 24.04 LTS support
- Kernel version validation (5.6+ required)
- Package cache management
- Service health monitoring
- Peer connectivity testing
- Automatic IP address assignment based on inventory order

### Documentation 📝
- Comprehensive README with detailed configuration instructions
- Variable reference table
- Troubleshooting guide
- Architecture diagrams
- Security best practices guide 