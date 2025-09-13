# Linux Patching Automation Framework

Enterprise-grade Linux patching automation built on Red Hat Ansible Automation Platform (AAP) 2.4.

## Overview

This project provides comprehensive Linux patching capabilities with automatic rollback, health validation, and integration points for monitoring and workflow systems.

**Repository**: https://github.com/jisujit/homelab-ansible-patching

## Key Features

- **Multi-OS Support**: Ubuntu, RHEL, CentOS, Debian
- **Service-Aware Patching**: Intelligent service management during updates
- **Automatic Rollback**: Emergency recovery capabilities
- **Health Validation**: Pre and post-patch system verification
- **AAP Integration**: Complete job templates and workflow orchestration

## Implementation Status

- ✅ **Core Patching Engine**: Fully operational with 11 security updates successfully applied
- ✅ **Pre-Patch Assessment**: System analysis and backup creation
- ✅ **Health Validation**: Post-patch system verification
- 🚧 **Zabbix Integration**: Framework ready, implementation pending
- 🚧 **n8n Integration**: Webhook structure ready, workflows pending

## Documentation

- [Technical Documentation](technical-guide.md) - Complete technical reference
- [Quick Start Guide](quick-start.md) - Getting started instructions
- [Future Roadmap](Roadmap_09122025.md) - Enhancement plans and priorities
- [GitHub Repository](https://github.com/jisujit/homelab-ansible-patching) - Source code

## Architecture

Built for scaling from HomeLab environments to enterprise deployments managing 50-70+ systems.

**Tested Environment:**
- AAP 2.4 Server: 192.168.9.40
- Target: VM 401 Ubuntu 24.04.3 (Paperless-NGX)
- Integration: Synology NAS, n8n, Zabbix monitoring

---

*This framework demonstrates enterprise-grade automation capabilities suitable for professional DevOps environments.*
