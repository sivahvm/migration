# Quick Start Guide - API Management Automation

## Corporate Standard - H1 Corporation

This guide will help you get started with the API Management Automation playbooks in 5 minutes.

---

## Prerequisites Check

Before starting, ensure you have:

- [ ] Ansible 2.9+ installed
- [ ] Python 3.6+ installed
- [ ] Access to source and destination API endpoints
- [ ] Valid OAuth credentials for both environments

---

## Quick Setup (5 Steps)

### Step 1: Set Environment Variables

```bash
# Source Environment
export SRC_TENANT="https://your-source-api.example.com"
export SRC_CLIENT_ID="your-source-client-id"
export SRC_CLIENT_SECRET="your-source-client-secret"

# Destination Environment
export DST_TENANT="https://your-destination-api.example.com"
export DST_CLIENT_ID="your-destination-client-id"
export DST_CLIENT_SECRET="your-destination-client-secret"
```

### Step 2: Verify Configuration

```bash
# Navigate to ansible directory
cd C:\project\h1\ansible

# Check inventory
cat inventory/hosts.yml

# Verify variables
cat group_vars/all.yml
```

### Step 3: Test Connectivity

```bash
# Ping hosts
ansible all -i inventory/hosts.yml -m ping

# Expected output:
# src_api_server | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

### Step 4: Run Playbook

```bash
# Execute the main playbook
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml

# With verbose output
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -v
```

### Step 5: Verify Results

Check the output for:
- ✓ OAuth tokens obtained
- ✓ Applications retrieved
- ✓ Attributes retrieved
- ✓ Data migrated successfully

---

## Common Commands

### Run Specific Phases

```bash
# Only authenticate (no data retrieval)
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --tags oauth

# Only retrieve data from source
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --tags source

# Only migrate data
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --tags migration
```

### Debug Mode

```bash
# Enable debug output
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -e "enable_debug=true"

# Maximum verbosity
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -vvv
```

### Dry Run

```bash
# Check what would happen without making changes
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --check
```

---

## Troubleshooting

### Issue: Authentication Failed

**Solution:**
```bash
# Verify credentials are set
echo $SRC_CLIENT_ID
echo $DST_CLIENT_ID

# Test API connectivity
curl -X POST $SRC_TENANT/oauth2/token \
  -d "grant_type=client_credentials" \
  -d "client_id=$SRC_CLIENT_ID" \
  -d "client_secret=$SRC_CLIENT_SECRET"
```

### Issue: Connection Timeout

**Solution:**
Edit `group_vars/all.yml`:
```yaml
http_timeout: 60  # Increase from 30
retry_count: 5    # Increase from 3
```

### Issue: Module Not Found

**Solution:**
```bash
# Install required Python packages
pip install ansible requests

# Verify installation
ansible --version
```

---

## Next Steps

1. **Review Logs**: Check `logs/ansible.log` for detailed execution logs
2. **Customize Variables**: Edit `group_vars/*.yml` for your environment
3. **Read Full Documentation**: See [README.md](README.md) for complete details
4. **Add Custom Roles**: Create new roles in `roles/` directory

---

## Support

- **Documentation**: [README.md](README.md)
- **Email**: devops@h1corporation.com
- **Slack**: #ansible-automation

---

## Example Output

```
PLAY [API Management | Source Environment - Authentication and Data Retrieval] ***

TASK [Source | Display playbook header] ******************************************
ok: [src_api_server] => {
    "msg": [
        "==========================================",
        "API MANAGEMENT AUTOMATION PLAYBOOK",
        "Corporate Standard - Version 1.0.0",
        "=========================================="
    ]
}

TASK [Source | Obtain OAuth access token] ****************************************
included: /path/to/roles/oauth_token/tasks/main.yml

TASK [OAuth Token | Request access token] ****************************************
ok: [src_api_server]

✓ OAuth token generated successfully
✓ Applications retrieved: 5
✓ Attributes retrieved: 12
✓ Data migrated successfully

PLAY RECAP ***********************************************************************
src_api_server             : ok=15   changed=0    unreachable=0    failed=0
dst_api_server             : ok=10   changed=5    unreachable=0    failed=0
```

---

**Last Updated**: April 1, 2026  
**Version**: 1.0.0