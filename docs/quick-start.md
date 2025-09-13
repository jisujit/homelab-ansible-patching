# Quick Start Guide

This guide walks you through deploying the Linux patching automation framework in your environment.

## Prerequisites

### Infrastructure Requirements
- Red Hat Ansible Automation Platform 2.4 (or AWX)
- Target Linux systems (Ubuntu, RHEL, CentOS)
- Network connectivity between AAP and target systems

### Access Requirements
- SSH access to target systems
- Sudo privileges on target systems
- GitHub account for repository access

## Initial Setup

### 1. Clone Repository
```bash
git clone https://github.com/jisujit/homelab-ansible-patching.git
cd homelab-ansible-patching
```

### 2. Configure SSH Keys
On your AAP server:
```bash
# Switch to awx user
sudo su - awx

# Generate SSH key pair
ssh-keygen -t rsa -b 4096 -f ~/.ssh/homelab_patching -N ""

# Copy public key to target systems
ssh-copy-id -i ~/.ssh/homelab_patching.pub user@target-system
```

### 3. Configure Passwordless Sudo
On each target system:
```bash
# Add user to sudoers with NOPASSWD
echo "username ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/username
sudo chmod 0440 /etc/sudoers.d/username
```

## AAP Configuration

### 1. Create AAP Project
1. Navigate to **Resources → Projects → Add**
2. Configure:
   - **Name**: `HomeLab Patching Project`
   - **Source Control Type**: `Git`
   - **SCM URL**: `https://github.com/jisujit/homelab-ansible-patching.git`
   - **SCM Branch**: `main`
   - **Update Revision on Launch**: ✓

### 2. Create Machine Credential
1. Navigate to **Resources → Credentials → Add**
2. Configure:
   - **Name**: `HomeLab SSH Key`
   - **Type**: `Machine`
   - **Username**: `target-system-username`
   - **SSH Private Key**: (paste content of homelab_patching private key)

### 3. Create Inventory
1. Navigate to **Resources → Inventories → Add**
2. Configure:
   - **Name**: `HomeLab Patching`
   - **Description**: `Systems for automated patching`

### 4. Add Hosts to Inventory
For each target system:
1. Click on inventory → **Hosts → Add**
2. Configure:
   - **Name**: `descriptive-hostname`
   - **Variables**:
```yaml
ansible_host: 192.168.x.x
ansible_user: target-username
patch_group: "group-name"
criticality: "medium"
services:
  - service-name-1
  - service-name-2
os_family: "ubuntu"  # or "rhel"
```

## Create Job Templates

### 1. Pre-Patch Assessment Template
1. Navigate to **Resources → Templates → Add → Job Template**
2. Configure:
   - **Name**: `HomeLab Pre-Patch Assessment`
   - **Job Type**: `Run`
   - **Inventory**: `HomeLab Patching`
   - **Project**: `HomeLab Patching Project`
   - **Playbook**: `playbooks/pre_patch_assessment.yml`
   - **Credentials**: `HomeLab SSH Key`
   - **Variables**:
```yaml
backup_retention_days: 14
```

### 2. Security Patching Template
1. Create another job template with:
   - **Name**: `HomeLab Security Patching`
   - **Playbook**: `playbooks/main_patching_playbook.yml`
   - **Ask Variables on Launch**: ✓
   - **Variables**:
```yaml
patch_type: "security"
batch_size: 1
rollback_on_failure: true
```

## Testing and Validation

### 1. Test Connectivity
1. Navigate to your inventory
2. Select a host
3. Run **Command** with `ping` module
4. Verify successful connection

### 2. Run Pre-Patch Assessment
1. Launch `HomeLab Pre-Patch Assessment`
2. Limit to single test system
3. Monitor execution for successful completion

### 3. Apply Security Patches
1. Launch `HomeLab Security Patching`
2. Set **Limit** to test system
3. Monitor patching process
4. Verify successful completion

## Expected Results

### Successful Pre-Patch Assessment
- System information gathered
- Available updates detected
- Backup directory created: `/var/backups/pre-patch-{timestamp}`
- Configuration files backed up

### Successful Patching
- Package cache updated
- Security updates applied
- Services restarted if needed
- System rebooted if kernel updated
- Health validation passed

## Troubleshooting

### SSH Connection Failures
```bash
# Test manual connection
ssh -i /var/lib/awx/.ssh/homelab_patching user@target

# Check key permissions
ls -la /var/lib/awx/.ssh/homelab_patching*
```

### Sudo Permission Errors
```bash
# Test passwordless sudo
ssh user@target sudo whoami

# Verify sudoers configuration
sudo visudo -c
```

### Package Manager Issues
```bash
# Check available updates
ssh user@target apt list --upgradable  # Ubuntu
ssh user@target dnf check-update       # RHEL
```

## Scaling to Production

Once testing is successful:

1. **Add remaining systems** to inventory
2. **Create system groups** based on criticality
3. **Configure batch sizes** for production scale
4. **Set up monitoring** integration (Zabbix, n8n)
5. **Implement change management** workflows

## Integration Setup

### Synology NAS Backup
Configure NAS mounting for centralized backups:
```yaml
nas_backup_enabled: true
nas_server: "192.168.x.x"
nas_backup_path: "/volume1/backups/patching"
```

### Monitoring Integration
Framework ready for Zabbix and n8n integration:
```yaml
zabbix_server: "monitoring-server-ip"
n8n_server: "workflow-server-ip"
```

For complete technical details, see [Technical Documentation](technical-guide.md).