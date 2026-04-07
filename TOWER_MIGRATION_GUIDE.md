# Ansible Tower Migration Guide
## Fixing "dict object has no attribute 'source_environment'" Error

**Issue:** The original playbook fails in Ansible Tower/AAP Controller with the error:
```
'dict object' has no attribute 'source_environment'
```

**Root Cause:** Ansible Tower/AAP Controller doesn't populate the `groups` variable the same way as standalone Ansible, causing `hostvars[groups['source_environment'][0]]` to fail.

---

## Solution Overview

The Tower-compatible playbook (`api_management_main_tower.yml`) uses a file-based approach to share data between plays instead of relying on `hostvars` and `groups`.

### Key Changes

1. **Data Persistence**: Source environment data is saved to a temporary file
2. **Data Loading**: Destination environment loads data from the file
3. **No hostvars Dependency**: Eliminates reliance on `groups` dictionary
4. **Cleanup**: Automatically removes temporary files after completion

---

## Migration Steps

### Step 1: Backup Original Playbook

```bash
cd C:\project\2026\Isracard\gitansible\migration\playbooks
cp api_management_main.yml api_management_main_original.yml
```

### Step 2: Replace with Tower-Compatible Version

```bash
# Option A: Replace the original
cp api_management_main_tower.yml api_management_main.yml

# Option B: Use alongside (recommended for testing)
# Keep both files and update Tower job template to use api_management_main_tower.yml
```

### Step 3: Update Ansible Tower Job Template

1. Log in to Ansible Tower/AAP Controller
2. Navigate to **Templates** → **SaaS Migration - Full Workflow**
3. Update the **Playbook** field:
   - Change from: `playbooks/api_management_main.yml`
   - Change to: `playbooks/api_management_main_tower.yml`
4. Click **Save**

### Step 4: Sync Project

1. Navigate to **Projects** → **SaaS Migration Ansible**
2. Click the **Sync** button
3. Wait for sync to complete
4. Verify the new playbook appears in the playbook dropdown

### Step 5: Test the Migration

```bash
# Run a test with minimal data
# In Tower UI:
# 1. Go to Templates → SaaS Migration - Full Workflow
# 2. Click Launch
# 3. Monitor the job output
```

---

## Technical Details

### Original Approach (Fails in Tower)

```yaml
- name: "API Management | Data Migration - Applications"
  hosts: destination_environment
  vars:
    # This fails in Tower because groups['source_environment'] is not available
    migration_data: "{{ hostvars[groups['source_environment'][0]]['source_applications'] | default([]) }}"
```

### Tower-Compatible Approach

```yaml
# Play 1: Source Environment - Save data to file
- name: "API Management | Source Environment"
  hosts: source_environment
  tasks:
    - name: "Source | Save migration data to temp file"
      ansible.builtin.copy:
        content: |
          ---
          src_access_token: "{{ src_access_token }}"
          source_applications: {{ source_applications | to_nice_json }}
          source_attributes: {{ source_attributes | to_nice_json }}
          source_workflows: {{ source_workflows | to_nice_json }}
        dest: "/tmp/migration_data_{{ ansible_date_time.epoch }}.yml"
      register: migration_data_file

# Play 2: Load Migration Data
- name: "API Management | Load Migration Data"
  hosts: destination_environment
  gather_facts: yes
  tasks:
    - name: "Migration | Find migration data file"
      ansible.builtin.find:
        paths: /tmp
        patterns: "migration_data_*.yml"
        age: -1h
      register: found_files

    - name: "Migration | Load migration data from file"
      ansible.builtin.include_vars:
        file: "{{ found_files.files | sort(attribute='mtime') | last | json_query('path') }}"
      when: found_files.matched > 0

# Play 3: Use loaded data
- name: "API Management | Data Migration - Applications"
  hosts: destination_environment
  vars:
    # Now source_applications is available from include_vars
    migration_data: "{{ source_applications | default([]) }}"
```

---

## Comparison: Original vs Tower-Compatible

| Feature | Original Playbook | Tower-Compatible Playbook |
|---------|------------------|---------------------------|
| **Data Sharing** | `hostvars[groups[...]]` | File-based persistence |
| **Tower Support** | ❌ Fails | ✅ Works |
| **Standalone Ansible** | ✅ Works | ✅ Works |
| **Cleanup** | Not needed | ✅ Automatic |
| **Complexity** | Lower | Slightly higher |
| **Reliability** | Tower-dependent | Platform-independent |

---

## Verification Checklist

After migration, verify the following:

- [ ] Project syncs successfully in Tower
- [ ] New playbook appears in job template dropdown
- [ ] Job template updated to use new playbook
- [ ] Test run completes without `groups` error
- [ ] Source data is retrieved successfully
- [ ] Destination receives and processes data
- [ ] Migration completes end-to-end
- [ ] Temporary files are cleaned up
- [ ] All job templates updated (if multiple exist)

---

## Troubleshooting

### Issue: File Not Found

**Symptom:**
```
Migration data file not found
```

**Solution:**
1. Check `/tmp` directory permissions on Tower execution nodes
2. Verify `gather_facts: yes` is set in the "Load Migration Data" play
3. Ensure source play completes successfully before destination play

### Issue: Permission Denied

**Symptom:**
```
Permission denied: /tmp/migration_data_*.yml
```

**Solution:**
1. Check Tower execution environment user permissions
2. Verify `/tmp` is writable
3. Consider using alternative path if `/tmp` is restricted:
   ```yaml
   dest: "{{ lookup('env', 'HOME') }}/.ansible/tmp/migration_data_{{ ansible_date_time.epoch }}.yml"
   ```

### Issue: Old Data Being Used

**Symptom:**
Migration uses data from previous run

**Solution:**
1. Cleanup is automatic, but verify it's working
2. Manually clean old files:
   ```bash
   find /tmp -name "migration_data_*.yml" -mtime +1 -delete
   ```
3. Check the `age: -1h` parameter in find task

---

## Rollback Procedure

If you need to rollback to the original playbook:

```bash
# Step 1: Restore original playbook
cd C:\project\2026\Isracard\gitansible\migration\playbooks
cp api_management_main_original.yml api_management_main.yml

# Step 2: Update Tower job template
# In Tower UI:
# - Navigate to Templates → SaaS Migration - Full Workflow
# - Change Playbook back to: playbooks/api_management_main.yml
# - Save

# Step 3: Sync project
# - Navigate to Projects → SaaS Migration Ansible
# - Click Sync button
```

---

## Best Practices

### 1. Testing Strategy

```yaml
# Test in this order:
1. Source retrieval only (--tags source)
2. Destination auth only (--tags destination)
3. Single application migration (--limit to 1 app)
4. Full migration with small dataset
5. Production migration
```

### 2. Monitoring

```yaml
# Enable debug mode for initial runs:
Extra Variables:
  enable_debug: true
  
# Monitor these logs:
- Tower job output
- /tmp/migration_data_*.yml file creation
- Cleanup task execution
```

### 3. Performance Optimization

```yaml
# For large datasets, consider:
1. Increase timeout values
2. Use tags to run specific phases
3. Split into multiple job templates
4. Use workflow templates for orchestration
```

---

## Additional Tower Configuration

### Update All Job Templates

If you have multiple job templates, update each one:

1. **SaaS Migration - Full Workflow**
   - Playbook: `playbooks/api_management_main_tower.yml`

2. **SaaS Migration - Source Retrieval**
   - Playbook: `playbooks/api_management_main_tower.yml`
   - Tags: `source,oauth,applications,attributes,workflows`

3. **SaaS Migration - Applications Only**
   - Playbook: `playbooks/api_management_main_tower.yml`
   - Tags: `oauth,applications,migration`

4. **SaaS Migration - Attributes Only**
   - Playbook: `playbooks/api_management_main_tower.yml`
   - Tags: `oauth,attributes,migration`

5. **SaaS Migration - Workflows Only**
   - Playbook: `playbooks/api_management_main_tower.yml`
   - Tags: `oauth,workflows,migration`

### Workflow Template Considerations

If using workflow templates, no changes needed - they reference job templates which now use the correct playbook.

---

## Support

For issues or questions:

- **DevOps Team**: devops@isracard.com
- **Documentation**: C:\project\2026\Isracard\gitansible\migration\TOWER_MIGRATION_GUIDE.md
- **Tower Setup Guide**: C:\project\2026\Isracard\saas_migration_ansible\ANSIBLE_TOWER_SETUP.md

---

## Changelog

### Version 2.0.0 (2026-04-07)
- ✅ Fixed Tower compatibility issue with `groups` variable
- ✅ Implemented file-based data sharing between plays
- ✅ Added automatic cleanup of temporary files
- ✅ Maintained backward compatibility with standalone Ansible
- ✅ Added comprehensive error handling
- ✅ Improved debugging capabilities

### Version 1.0.0 (2026-04-01)
- Initial release (Tower-incompatible)

---

**Document Version**: 2.0.0  
**Last Updated**: April 7, 2026  
**Maintained By**: Isracard DevOps Team