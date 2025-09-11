# HomeLab Ansible Patching Automation

Enterprise-grade Linux patching automation for HomeLab and workplace deployment using Red Hat Ansible Automation Platform.

## 🏗️ Architecture

This automation framework provides comprehensive Linux patching with:
- **Service-aware patching** with automatic rollback capabilities
- **Multi-OS support** (Ubuntu, RHEL, CentOS, Debian)
- **Health monitoring** and validation
- **Integration** with Synology NAS, n8n workflows, and Zabbix monitoring
- **Professional workflow** suitable for enterprise deployment

## 🎯 Current Lab Environment

- **AAP Server**: 192.168.9.40 (Red Hat Ansible Automation Platform 2.4)
- **Target Systems**: 
  - VM 401: Ubuntu 24.04.3 LTS (Paperless-NGX) - 192.168.9.180
- **Storage**: Synology NAS (192.168.9.13) for backup integration
- **Monitoring**: Zabbix (192.168.9.820)
- **Automation**: n8n workflows (192.168.9.830)

## 🚀 Quick Start

### 1. Test Connectivity
```bash
ansible vm-401-paperless -m ping
