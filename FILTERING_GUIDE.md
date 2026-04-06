# Application and Attribute Filtering Guide

## Overview

This guide explains how to filter applications and attributes by ID when running the API Management playbook. By default, the playbook processes ALL applications and attributes. You can specify which ones to process by configuring filter lists.

## Configuration

### Location

Edit the filter configuration in: `inventory/group_vars/all.yml`

### Filter Variables

```yaml
# Filter Configuration
# Specify application IDs to process (empty list means process all)
filter_application_ids: []

# Specify attribute IDs to process (empty list means process all)
filter_attribute_ids: []
```

## Usage Examples

### Example 1: Process All (Default Behavior)

```yaml
# inventory/group_vars/all.yml
filter_application_ids: []
filter_attribute_ids: []
```

This will process ALL applications and attributes found in the source environment.

### Example 2: Filter Specific Applications

```yaml
# inventory/group_vars/all.yml
filter_application_ids:
  - "app-12345-abcde"
  - "app-67890-fghij"
  - "app-11111-klmno"
filter_attribute_ids: []
```

This will:
- Process ONLY the 3 specified applications
- Process ALL attributes

### Example 3: Filter Specific Attributes

```yaml
# inventory/group_vars/all.yml
filter_application_ids: []
filter_attribute_ids:
  - "attr-aaaaa-11111"
  - "attr-bbbbb-22222"
```

This will:
- Process ALL applications
- Process ONLY the 2 specified attributes

### Example 4: Filter Both Applications and Attributes

```yaml
# inventory/group_vars/all.yml
filter_application_ids:
  - "app-12345-abcde"
  - "app-67890-fghij"
filter_attribute_ids:
  - "attr-aaaaa-11111"
  - "attr-bbbbb-22222"
  - "attr-ccccc-33333"
```

This will:
- Process ONLY the 2 specified applications
- Process ONLY the 3 specified attributes

## How to Find IDs

### Method 1: Run Playbook with Debug Mode

```bash
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -e "enable_debug=true"
```

The debug output will show all application and attribute IDs:

```
TASK [Application Management | Debug - Application names]
ok: [src_api_server] => (item=MyApp1) => {
    "msg": "  - MyApp1 (ID: app-12345-abcde)"
}
ok: [src_api_server] => (item=MyApp2) => {
    "msg": "  - MyApp2 (ID: app-67890-fghij)"
}
```

### Method 2: Check API Directly

Use the API endpoints to list resources:
- Applications: `GET https://your-tenant.com/v1.0/applications`
- Attributes: `GET https://your-tenant.com/v1.0/attributes`

## Running the Playbook

### Standard Run (Process All)

```bash
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml
```

### With Debug Output

```bash
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -e "enable_debug=true"
```

### With Verbose Output

```bash
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -vvv
```

## Verification

When filters are applied, you'll see debug messages (if debug is enabled):

```
TASK [Application Management | Debug - Filtered applications]
ok: [src_api_server] => {
    "msg": [
        "Filter applied: 2 application IDs specified",
        "Applications after filtering: 2"
    ]
}
```

## Best Practices

1. **Test First**: Run with debug mode to see all available IDs before filtering
2. **Verify IDs**: Ensure the IDs you specify actually exist in the source environment
3. **Empty Lists**: Use empty lists `[]` to process all items (default behavior)
4. **Documentation**: Keep a record of which IDs you're filtering and why
5. **Incremental Migration**: Use filters to migrate in batches for large environments

## Troubleshooting

### No Items Processed

If you see "0 items after filtering":
- Verify the IDs in your filter list are correct
- Check for typos in the IDs
- Ensure the IDs exist in the source environment

### Filter Not Applied

If all items are still being processed:
- Check that `filter_application_ids` or `filter_attribute_ids` is not empty
- Verify the syntax in `all.yml` is correct (proper YAML list format)
- Ensure you're editing the correct file: `inventory/group_vars/all.yml`

## Example Workflow

1. **Discovery Phase**:
   ```bash
   ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -e "enable_debug=true" --tags "list"
   ```

2. **Update Filter Configuration**:
   Edit `inventory/group_vars/all.yml` with desired IDs

3. **Test Migration**:
   ```bash
   ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -e "enable_debug=true" --check
   ```

4. **Execute Migration**:
   ```bash
   ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml
   ```

## Notes

- Filtering happens AFTER retrieving the full list from the API
- The filter uses exact ID matching (case-sensitive)
- Invalid IDs in the filter list are silently ignored
- Empty filter lists mean "process all" (no filtering applied)

---

**Last Updated**: 2026-04-06  
**Version**: 1.0.0