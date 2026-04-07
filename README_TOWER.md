# Ansible Tower/AAP Controller Setup - Quick Start
## SaaS Migration Project - Isracard

**Status**: ✅ Tower-Compatible Version Available  
**Version**: 2.0.0  
**Date**: April 7, 2026

---

## 🚨 Important Notice

The original playbook (`api_management_main.yml`) **fails in Ansible Tower/AAP Controller** with this error:

```
'dict object' has no attribute 'source_environment'
```

**Solution**: Use the Tower-compatible playbook: [`api_management_main_tower.yml`](playbooks/api_management_main_tower.yml)

---

## Quick Start Guide

### 1. Update Your Tower Job Template

```yaml
Job Template Settings:
  Name: SaaS Migration - Full Workflow
  Playbook: playbooks/api_management_main_tower.yml  # ← Changed from api_management_main.yml
  Inventory: SaaS Migration Inventory
  Credentials:
    - Source API Credentials
    - Destination API Credentials
```

### 2. Sync Your Project

1. Navigate to **Projects** → **SaaS Migration Ansible**
2. Click **Sync** button
3. Verify sync completes successfully

### 3. Test the Migration

1. Go to **Templates** → **SaaS Migration - Full Workflow**
2. Click **Launch**
3. Monitor job output for successful completion

---

## What Changed?

### Problem
The original playbook used `hostvars[groups['source_environment'][0]]` to share data between plays, which doesn't work in Tower.

### Solution
The Tower-compatible playbook uses a file-based approach:

1. **Source play** saves data to `/tmp/migration_data_<timestamp>.yml`
2. **Destination play** loads data from the file
3. **Cleanup play** removes temporary files

### Benefits
- ✅ Works in Ansible Tower/AAP Controller
- ✅ Works in standalone Ansible
- ✅ Automatic cleanup
- ✅ No code changes to roles
- ✅ Same functionality as original

---

## File Locations

| File | Purpose | Location |
|------|---------|----------|
| **Tower-Compatible Playbook** | Main playbook for Tower | [`playbooks/api_management_main_tower.yml`](playbooks/api_management_main_tower.yml) |
| **Migration Guide** | Detailed migration instructions | [`TOWER_MIGRATION_GUIDE.md`](TOWER_MIGRATION_GUIDE.md) |
| **Original Playbook** | Standalone Ansible version | [`playbooks/api_management_main.yml`](playbooks/api_management_main.yml) |

---

## Tower Configuration Files

Additional configuration files for Tower setup:

| File | Purpose | Location |
|------|---------|----------|
| **Tower Setup Guide** | Complete Tower configuration | `C:\project\2026\Isracard\saas_migration_ansible\ANSIBLE_TOWER_SETUP.md` |
| **Tower Config YAML** | Configuration as code | `C:\project\2026\Isracard\saas_migration_ansible\tower_config.yml` |
| **Setup Script** | Automated Tower setup | `C:\project\2026\Isracard\saas_migration_ansible\scripts\setup_tower.sh` |

---

## Key Differences

### Original Playbook (Standalone Ansible)
```yaml
- name: "Data Migration - Applications"
  hosts: destination_environment
  vars:
    # ❌ Fails in Tower
    migration_data: "{{ hostvars[groups['source_environment'][0]]['source_applications'] }}"
```

### Tower-Compatible Playbook
```yaml
# Play 1: Save data
- name: "Source Environment"
  hosts: source_environment
  tasks:
    - name: "Save migration data"
      ansible.builtin.copy:
        content: "{{ source_applications | to_nice_json }}"
        dest: "/tmp/migration_data_{{ ansible_date_time.epoch }}.yml"

# Play 2: Load data
- name: "Load Migration Data"
  hosts: destination_environment
  tasks:
    - name: "Load migration data"
      ansible.builtin.include_vars:
        file: "/tmp/migration_data_*.yml"

# Play 3: Use data
- name: "Data Migration - Applications"
  hosts: destination_environment
  vars:
    # ✅ Works in Tower
    migration_data: "{{ source_applications | default([]) }}"
```

---

## Verification Steps

After updating your Tower configuration:

1. **Verify Project Sync**
   ```
   Projects → SaaS Migration Ansible → Sync → ✅ Success
   ```

2. **Verify Playbook Available**
   ```
   Templates → Job Template → Playbook dropdown → 
   ✅ api_management_main_tower.yml appears
   ```

3. **Test Source Retrieval**
   ```
   Launch job with tags: source,oauth,applications
   ✅ Data retrieved and saved to /tmp/migration_data_*.yml
   ```

4. **Test Full Migration**
   ```
   Launch full workflow job
   ✅ Source data retrieved
   ✅ Destination authenticated
   ✅ Data migrated successfully
   ✅ Temporary files cleaned up
   ```

---

## Troubleshooting

### Error: "dict object has no attribute 'source_environment'"

**Cause**: Using original playbook in Tower

**Solution**: Update job template to use `api_management_main_tower.yml`

### Error: "Migration data file not found"

**Cause**: Source play didn't complete or file permissions issue

**Solution**:
1. Check source play completed successfully
2. Verify `/tmp` directory is writable
3. Check `gather_facts: yes` in Load Migration Data play

### Error: "Permission denied: /tmp/migration_data_*.yml"

**Cause**: Tower execution environment lacks write permissions

**Solution**:
1. Verify Tower user has write access to `/tmp`
2. Or use alternative path:
   ```yaml
   dest: "{{ lookup('env', 'HOME') }}/.ansible/tmp/migration_data_{{ ansible_date_time.epoch }}.yml"
   ```

---

## Migration Checklist

Use this checklist when migrating to Tower-compatible playbook:

- [ ] Backup original playbook
- [ ] Review [`TOWER_MIGRATION_GUIDE.md`](TOWER_MIGRATION_GUIDE.md)
- [ ] Update job template playbook path
- [ ] Sync Tower project
- [ ] Test source retrieval only
- [ ] Test destination authentication
- [ ] Test single item migration
- [ ] Test full migration workflow
- [ ] Verify cleanup executes
- [ ] Update all related job templates
- [ ] Update workflow templates (if any)
- [ ] Document changes in runbook
- [ ] Train team on new playbook

---

## Support Resources

### Documentation
- **Tower Migration Guide**: [`TOWER_MIGRATION_GUIDE.md`](TOWER_MIGRATION_GUIDE.md) - Detailed migration instructions
- **Tower Setup Guide**: `C:\project\2026\Isracard\saas_migration_ansible\ANSIBLE_TOWER_SETUP.md` - Complete Tower configuration
- **Filtering Guide**: [`FILTERING_GUIDE.md`](FILTERING_GUIDE.md) - Data filtering options
- **Quick Start**: [`QUICKSTART.md`](QUICKSTART.md) - Getting started guide

### Contacts
- **DevOps Team**: devops@isracard.com
- **API Team**: api-support@isracard.com
- **Tower Admin**: tower-admin@isracard.com

### Useful Commands

```bash
# Test playbook locally (if needed)
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main_tower.yml

# Check Tower job status
tower-cli job list --job-template "SaaS Migration - Full Workflow"

# Monitor running job
tower-cli job monitor <job-id>

# View job output
tower-cli job stdout <job-id>

# Sync project
tower-cli project update --name "SaaS Migration Ansible" --wait
```

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Ansible Tower/AAP                         │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Job Template: SaaS Migration - Full Workflow      │    │
│  │  Playbook: api_management_main_tower.yml           │    │
│  └────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Play 1: Source Environment                        │    │
│  │  - Authenticate                                     │    │
│  │  - Retrieve data                                    │    │
│  │  - Save to /tmp/migration_data_<timestamp>.yml     │    │
│  └────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Play 2: Destination Environment                   │    │
│  │  - Authenticate                                     │    │
│  └────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Play 3: Load Migration Data                       │    │
│  │  - Find migration data file                        │    │
│  │  - Load data with include_vars                     │    │
│  └────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Play 4-6: Data Migration                          │    │
│  │  - Migrate applications                            │    │
│  │  - Migrate attributes                              │    │
│  │  - Migrate workflows                               │    │
│  └────────────────────────────────────────────────────┘    │
│                          │                                   │
│                          ▼                                   │
│  ┌────────────────────────────────────────────────────┐    │
│  │  Play 7: Cleanup                                   │    │
│  │  - Remove temporary files                          │    │
│  │  - Display summary                                 │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Next Steps

1. **Review Migration Guide**: Read [`TOWER_MIGRATION_GUIDE.md`](TOWER_MIGRATION_GUIDE.md) for detailed instructions
2. **Update Tower**: Follow the Quick Start Guide above
3. **Test Migration**: Run test migrations with small datasets
4. **Production Deployment**: Schedule production migrations
5. **Monitor**: Set up monitoring and alerting in Tower

---

## Changelog

### Version 2.0.0 (2026-04-07)
- ✅ Created Tower-compatible playbook
- ✅ Fixed `groups` variable issue
- ✅ Implemented file-based data sharing
- ✅ Added automatic cleanup
- ✅ Maintained backward compatibility

### Version 1.0.0 (2026-04-01)
- Initial release (Tower-incompatible)

---

**Document Version**: 2.0.0  
**Last Updated**: April 7, 2026  
**Maintained By**: Isracard DevOps Team

---

## Quick Reference

| Task | Command/Action |
|------|----------------|
| **Update Job Template** | Templates → Edit → Change Playbook to `api_management_main_tower.yml` |
| **Sync Project** | Projects → SaaS Migration Ansible → Sync |
| **Launch Job** | Templates → SaaS Migration - Full Workflow → Launch |
| **View Logs** | Jobs → Select Job → Output |
| **Check Status** | Jobs → Select Job → Details |

---

**Need Help?** Contact devops@isracard.com or refer to [`TOWER_MIGRATION_GUIDE.md`](TOWER_MIGRATION_GUIDE.md)