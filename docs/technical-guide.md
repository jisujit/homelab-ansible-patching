# Technical Documentation
# Linux Patching Automation Framework - Technical Documentation

## Executive Summary

This document describes a comprehensive Linux patching automation framework built on Red Hat Ansible Automation Platform (AAP) 2.4. The system provides enterprise-grade patch management capabilities with automatic rollback, health validation, and integration points for monitoring and workflow systems.

**Repository**: https://github.com/jisujit/homelab-ansible-patching

## Architecture Overview

### Core Components
- **Ansible Automation Platform 2.4**: Central orchestration engine
- **Multi-OS Support**: Ubuntu, RHEL, CentOS, Debian
- **Git-Based Source Control**: GitHub integration for version control
- **Service-Aware Patching**: Intelligent service management during updates
- **Automatic Rollback**: Emergency recovery capabilities
- **Health Validation**: Pre and post-patch system verification

### Integration Points
- **Synology NAS**: Centralized backup storage
- **Zabbix Monitoring**: System health and maintenance mode
- **n8n Workflows**: Notification and approval automation
- **ServiceNow**: Change management integration (configurable)

## Technical Architecture

### Directory Structure
```
homelab-ansible-patching/
├── ansible.cfg                 # AAP-optimized configuration
├── requirements.yml            # Ansible collections dependencies
├── playbooks/                  # Core automation logic
│   ├── pre_patch_assessment.yml
│   ├── main_patching_playbook.yml
│   ├── health_checks.yml
│   ├── rollback_system.yml
│   └── site.yml                # Main workflow orchestrator
├── inventory/
│   └── production.yml          # System inventory (file-based backup)
├── group_vars/
│   ├── all.yml                 # Global variables
│   └── ubuntu_systems.yml      # OS-specific variables
├── templates/
│   └── pre_patch_report.j2     # Reporting templates
└── README.md                   # Project documentation
```

### Configuration Management

#### AAP-Optimized ansible.cfg
```ini
[defaults]
# AAP manages these via web interface:
# host_key_checking, inventory, forks, timeout

roles_path = roles
collections_path = collections
stdout_callback = yaml
gathering = smart
fact_caching = memory
deprecation_warnings = False

[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=60s
pipelining = True
control_path_dir = /tmp/.ansible/cp
retries = 3

[privilege_escalation]
become = True
become_method = sudo
become_ask_pass = False
```

## Core Features

### 1. Pre-Patch Assessment

**File**: `playbooks/pre_patch_assessment.yml`

**Capabilities**:
- System information gathering (OS, kernel, uptime, disk space)
- Available updates detection and counting
- Critical service identification and status verification
- Automatic backup creation with timestamped directories
- Configuration file preservation
- Package inventory capture
- Integration webhook notifications

**Key Tasks**:
```yaml
- Create timestamped backup directory: /var/backups/pre-patch-{epoch}
- Verify minimum disk space (2GB threshold)
- Identify running critical services
- Backup package lists (dpkg -l / rpm -qa)
- Backup configuration files (/etc/fstab, /etc/hosts, etc.)
- Generate JSON assessment report
```

**Output**: Structured JSON report containing system state snapshot

### 2. Main Patching Engine

**File**: `playbooks/main_patching_playbook.yml`

**Capabilities**:
- Multi-OS package management (apt/dnf/yum)
- Selective patching (security-only vs. all updates)
- Service-aware patching with controlled stop/start sequences
- Automatic reboot detection and handling
- Real-time error handling with rollback triggers
- Batch processing with configurable group sizes

**Patch Types**:
- **Security**: Only security-related updates
- **All**: Complete system updates including feature updates

**Safety Features**:
- Serial execution (configurable batch sizes)
- Automatic rollback on failure
- Service dependency management
- Reboot requirement detection

### 3. Health Validation System

**File**: `playbooks/health_checks.yml`

**Capabilities**:
- Post-patch system responsiveness verification
- Critical service status validation
- Web service connectivity testing
- Disk space monitoring
- Failed service detection
- Network connectivity verification
- Comprehensive health scoring

**Health Check Matrix**:
```yaml
health_check_results:
  system_responsive: boolean
  services_healthy: boolean
  web_services_healthy: boolean
  disk_space_ok: boolean
  network_ok: boolean
  failed_services_count: integer
```

### 4. Emergency Rollback System

**File**: `playbooks/rollback_system.yml`

**Capabilities**:
- Automatic package rollback (dnf history undo)
- Configuration file restoration from backups
- Service restart and stabilization
- Limited Ubuntu/Debian rollback (package downgrade complexity)
- Integration notifications for rollback events

**Rollback Triggers**:
- Patch installation failures
- Post-patch health check failures
- Service restart failures
- Manual execution via AAP job template

### 5. Workflow Orchestration

**File**: `playbooks/site.yml`

**Capabilities**:
- Complete end-to-end patching workflow
- Conditional execution (skip assessment/health checks)
- Integration point for complex deployment scenarios
- Workflow customization via variables

## Variable Management

### Global Variables (group_vars/all.yml)
```yaml
# Operational Settings
patch_maintenance_window: "02:00-06:00"
rollback_on_failure: true
health_check_retries: 3
backup_retention_days: 14

# Integration Settings
zabbix_server: "192.168.9.122"
n8n_server: "192.168.9.128"
nas_server: "192.168.9.13"
nas_backup_path: "/volume1/backups/patching"

# Security Settings
nas_backup_enabled: true
```

### Host-Specific Variables
```yaml
# Per-host configuration in AAP inventory
ansible_host: 192.168.9.180
ansible_user: paperlessadmin
patch_group: "document_management"
criticality: "medium"
services:
  - paperless-webserver
  - paperless-worker
os_family: "ubuntu"
os_version: "24.04"
```

## AAP Integration

### Job Templates

#### 1. Pre-Patch Assessment
- **Purpose**: System analysis and backup creation
- **Execution Time**: 5-10 minutes
- **Prerequisites**: SSH connectivity, sudo access
- **Output**: System assessment report and backup directory

#### 2. Security Patching
- **Purpose**: Apply security updates with rollback capability
- **Execution Time**: 15-60 minutes (depending on update count)
- **Variables**:
  - `patch_type: "security"`
  - `rollback_on_failure: true`
  - `batch_size: 1`

#### 3. Full System Patching
- **Purpose**: Apply all available updates
- **Execution Time**: 30-120 minutes
- **Risk Level**: Higher (includes feature updates)

#### 4. Emergency Rollback
- **Purpose**: Manual rollback execution
- **Execution Time**: 10-30 minutes
- **Use Case**: Manual intervention after failed patches

### Workflow Templates
- **Complete Patching Workflow**: Orchestrates assessment → patching → validation
- **Conditional Execution**: Skip components based on variables
- **Failure Handling**: Automatic rollback on any workflow failure

## Integration Capabilities

### Synology NAS Integration
- **Backup Storage**: Centralized backup repository
- **NFS Mount**: Automatic mounting for backup operations
- **Retention Management**: Configurable backup retention policies

### Zabbix Monitoring Integration
- **Maintenance Mode**: Automatic alert suppression during patching
- **Health Metrics**: Post-patch system health reporting
- **API Integration**: RESTful API calls for status updates

### n8n Workflow Integration
- **Webhook Notifications**: Real-time patching status updates
- **Approval Workflows**: Integration point for change approval
- **Escalation Procedures**: Automated escalation for failed patches

### ServiceNow Integration (Framework Ready)
- **Change Management**: Automatic change ticket creation
- **Approval Gates**: Integration with ITSM approval processes
- **Compliance Reporting**: Automated compliance documentation

## Security Implementation

### Access Control
- **SSH Key Management**: Dedicated automation keys with restricted access
- **Privilege Escalation**: Passwordless sudo for automation accounts
- **Credential Storage**: AAP vault for sensitive information
- **Audit Logging**: Complete activity logging via AAP

### Network Security
- **Encrypted Communication**: SSH and HTTPS for all connections
- **Network Segmentation**: Support for VLAN-based isolation
- **Firewall Integration**: Configurable network access controls

## Scaling and Expansion

### Multi-Environment Support
```yaml
# Environment-specific variables
environments:
  development:
    change_window: "anytime"
    rollback_on_failure: false
    batch_size: 10

  production:
    change_window: "Sunday 01:00-05:00"
    rollback_on_failure: true
    batch_size: 1
    require_approval: true
```

### Batch Processing Configuration
```yaml
# Criticality-based batching
critical_systems:
  batch_size: 1          # One at a time
  max_fail_percentage: 0 # Zero tolerance

standard_systems:
  batch_size: 10         # Larger batches
  max_fail_percentage: 20 # Higher tolerance
```

### Custom Service Handling
```yaml
# Service-specific configurations
services_to_stop_before_patch:
  - application-service
  - worker-service

services_to_restart_after_patch:
  - critical-service
  - monitoring-agent
```

## What's Actually Working vs Framework Ready

### Production-Ready Components
These components are fully implemented and tested:

**Core Patching Automation**:
- Multi-OS package management (apt/dnf) with 11 security updates successfully applied
- Pre-patch system assessment and backup creation
- Post-patch health validation and system verification
- Emergency rollback procedures (package-level and configuration restoration)
- AAP job template integration with real-time monitoring

**Infrastructure Integration**:
- SSH key management and passwordless sudo configuration
- Local backup creation in timestamped directories
- Basic file system operations and service management
- Git-based version control with professional workflow

### Framework-Ready But Not Implemented
These components have variable structures and placeholder code but require completion:

**Zabbix Integration** (Configuration Ready):
- Variables configured but no active API calls
- Maintenance mode automation framework exists but not functional
- Health metric reporting structure ready but not connected

**n8n Integration** (Endpoints Ready):
- Webhook calls configured but workflows don't exist on n8n server
- JSON payload structure defined but no receiving workflows built
- Error handling prevents failures but no actual integration working

**NAS Integration** (Partially Working):
- Local backup creation works perfectly
- Remote NFS mounting framework exists but not tested
- Centralized backup aggregation planned but not implemented

### Daily Operations
1. **Morning Health Check**: Verify overnight patching results
2. **Security Advisory Review**: Check for new vulnerabilities
3. **Patch Queue Management**: Prioritize systems for patching

### Emergency Procedures
1. **Critical Vulnerability Response**: 0-day exploitation mitigation
2. **Mass Rollback**: Multiple system failure recovery
3. **Service Recovery**: Post-patch service restoration

### Maintenance Procedures
1. **Backup Validation**: Verify backup integrity and accessibility
2. **Rollback Testing**: Regular rollback procedure validation
3. **Integration Testing**: Verify all integration points

## Performance Metrics

### Key Performance Indicators
- **Patch Success Rate**: Target >95%
- **Mean Time to Patch**: <72 hours for critical vulnerabilities
- **Rollback Rate**: Target <2%
- **System Availability**: >99.5% during maintenance windows

### Monitoring Points
- **Job Execution Time**: Track performance trends
- **Error Rates**: Monitor failure patterns
- **Resource Usage**: System resource consumption during patching

## Troubleshooting Guide

### Common Issues

#### SSH Authentication Failures
```bash
# Verify key permissions
ls -la /var/lib/awx/.ssh/
# Test manual connection
sudo -u awx ssh -i /var/lib/awx/.ssh/homelab_patching user@target
```

#### Sudo Permission Errors
```bash
# Verify sudoers configuration
sudo visudo -c
# Test passwordless sudo
ssh user@target sudo whoami
```

#### Package Manager Conflicts
```bash
# Check package manager status
ssh user@target sudo apt list --upgradable
ssh user@target sudo dnf check-update
```

#### Service Management Issues
```bash
# Verify service status
ssh user@target systemctl --failed
# Check service dependencies
ssh user@target systemctl list-dependencies service-name
```

### Log Analysis
- **AAP Job Logs**: Web interface → Jobs → Job Details
- **System Logs**: `/var/log/` on target systems
- **Backup Logs**: `/var/backups/pre-patch-*/` directories

## Development and Extension

### Adding New Operating Systems
1. **Create OS-specific group_vars**: `group_vars/new_os_systems.yml`
2. **Update package management tasks**: Add conditional tasks for new package manager
3. **Test reboot detection**: Verify reboot requirement detection methods
4. **Update service management**: Add OS-specific service management commands

### Custom Integration Development
1. **Webhook Endpoints**: Add new webhook notifications
2. **API Integrations**: Extend monitoring and ITSM integrations
3. **Custom Health Checks**: Application-specific validation logic
4. **Reporting Extensions**: Custom report formats and delivery methods

### Best Practices for Extension
- **Variable Defaults**: Always provide fallback values
- **Error Handling**: Use `ignore_errors` and `rescue` blocks appropriately
- **Idempotency**: Ensure playbooks can run multiple times safely
- **Documentation**: Update this document for any significant changes

## Migration and Backup

### AAP to AWX Migration
The framework is fully compatible with AWX (open-source version):
1. **Export AAP Configuration**: Backup projects, inventories, credentials
2. **Install AWX**: Docker or Kubernetes deployment
3. **Import Configuration**: Restore all AAP settings
4. **Update Git Repository**: No playbook changes required

### Backup Strategy
- **Git Repository**: Complete version control via GitHub
- **AAP Configuration**: Regular configuration exports
- **System Backups**: Automated backup creation before patching
- **Documentation**: Maintain up-to-date technical documentation

## Future Enhancements

### Planned Features
- **Vulnerability Scanner Integration**: OpenSCAP, Nessus API integration
- **Compliance Automation**: PCI DSS, SOX, HIPAA reporting
- **Advanced Scheduling**: Maintenance window automation
- **Multi-Cloud Support**: AWS, Azure, GCP integration points

### Integration Roadmap
- **Container Platforms**: Kubernetes cluster patching
- **Network Devices**: Switch and router firmware updates
- **Application Patching**: Application-specific update procedures
- **Database Maintenance**: Automated database patching workflows

---

## Contact and Support

**Repository**: https://github.com/jisujit/homelab-ansible-patching
**Documentation**: README.md in repository
**Issue Tracking**: GitHub Issues
**Knowledge Base**: Technical documentation and operational procedures

This framework represents an enterprise-grade implementation suitable for scaling from HomeLab environments to production deployments managing hundreds of systems.# Linux Patching Automation Framework - Technical Documentation
