# HomeLab Ansible Patching Automation

Enterprise-grade Linux patching automation for HomeLab and workplace deployment using Red Hat Ansible Automation Platform (AAP) 2.4.

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
- **Storage**: Synology NAS (192.168.9.13) - Web: http://192.168.9.13:5000/
- **Monitoring**: Zabbix (192.168.9.122)
- **Automation**: n8n workflows (192.168.9.128)

## 🚀 Quick Start

### 1. Test Connectivity
```bash
# Via AAP: Run Ad Hoc Command with ping module
# Or locally: ansible vm-401-paperless -m ping
ansible vm-401-paperless -m ping