# API Management Automation - Ansible Playbooks

## Corporate Standard Documentation

**Project:** API Management Automation  
**Version:** 1.0.0  
**Organization:** H1 Corporation  
**Date:** April 2026  

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Directory Structure](#directory-structure)
4. [Prerequisites](#prerequisites)
5. [Configuration](#configuration)
6. [Usage](#usage)
7. [Roles Description](#roles-description)
8. [Playbook Execution](#playbook-execution)
9. [Troubleshooting](#troubleshooting)
10. [Corporate Standards](#corporate-standards)

---

## Overview

This Ansible automation framework manages API operations including OAuth authentication, application management, attribute management, and data migration between source and destination environments. The playbooks are based on Postman collection workflows and follow corporate standards for enterprise automation.

### Key Features

- ✅ **OAuth Token Management**: Automated token generation and refresh
- ✅ **Application Management**: List, retrieve, and manage applications
- ✅ **Attribute Management**: Handle attribute CRUD operations
- ✅ **Data Migration**: Migrate data between environments with validation
- ✅ **Looping Constructs**: Process multiple items efficiently
- ✅ **Debug Statements**: Comprehensive logging for each operation
- ✅ **Error Handling**: Retry logic and validation checks
- ✅ **Corporate Standards**: Follows H1 Corporation coding standards

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Main Playbook                            │
│              (api_management_main.yml)                      │
└─────────────────────────────────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│   Phase 1    │    │   Phase 2    │    │   Phase 3    │
│   Source     │───▶│ Destination  │───▶│  Migration   │
│    Auth      │    │    Auth      │    │    Data      │
└──────────────┘    └──────────────┘    └──────────────┘
        │                   │                   │
        ▼                   ▼                   ▼
    [Roles]             [Roles]             [Roles]
```

---

## Directory Structure

```
ansible/
├── inventory/
│   └── hosts.yml                          # Inventory with host groups
├── group_vars/
│   ├── all.yml                            # Global variables
│   ├── source_environment.yml             # Source environment vars
│   └── destination_environment.yml        # Destination environment vars
├── roles/
│   ├── oauth_token/
│   │   └── tasks/
│   │       └── main.yml                   # OAuth token management
│   ├── application_management/
│   │   └── tasks/
│   │       ├── main.yml                   # Application operations
│   │       └── process_application.yml    # Individual app processing
│   ├── attribute_management/
│   │   └── tasks/
│   │       ├── main.yml                   # Attribute operations
│   │       └── process_attribute.yml      # Individual attr processing
│   └── data_migration/
│       └── tasks/
│           ├── main.yml                   # Migration orchestration
│           └── migrate_item.yml           # Individual item migration
├── playbooks/
│   └── api_management_main.yml            # Main orchestration playbook
├── README.md                              # This file
└── ansible.cfg                            # Ansible configuration
```

---

## Prerequisites

### System Requirements

- **Ansible Version**: 2.9 or higher
- **Python**: 3.6 or higher
- **Operating System**: Linux, macOS, or Windows (with WSL)

### Python Packages

```bash
pip install ansible
pip install requests
```

### Network Requirements

- Access to source API endpoint
- Access to destination API endpoint
- HTTPS connectivity (port 443)

---

## Configuration

### 1. Environment Variables

Set the following environment variables or update the group_vars files:

```bash
# Source Environment
export SRC_TENANT="https://source-api.example.com"
export SRC_CLIENT_ID="your-source-client-id"
export SRC_CLIENT_SECRET="your-source-client-secret"

# Destination Environment
export DST_TENANT="https://destination-api.example.com"
export DST_CLIENT_ID="your-destination-client-id"
export DST_CLIENT_SECRET="your-destination-client-secret"
```

### 2. Update Inventory

Edit `inventory/hosts.yml` to match your environment:

```yaml
all:
  children:
    api_management:
      children:
        source_environment:
          hosts:
            src_api_server:
              ansible_host: localhost
        destination_environment:
          hosts:
            dst_api_server:
              ansible_host: localhost
```

### 3. Configure Variables

Update `group_vars/all.yml` for global settings:

```yaml
http_timeout: 30
retry_count: 3
enable_debug: true
```

---

## Usage

### Basic Execution

```bash
# Run the complete workflow
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml

# Run with verbose output
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -v

# Run with extra verbosity (debug)
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -vvv
```

### Tag-Based Execution

```bash
# Run only OAuth authentication
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --tags oauth

# Run only source environment tasks
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --tags source

# Run only migration tasks
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --tags migration

# Skip debug tasks
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --skip-tags debug
```

### Dry Run (Check Mode)

```bash
# Check what would be executed without making changes
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml --check
```

---

## Roles Description

### 1. oauth_token

**Purpose**: Manages OAuth 2.0 token generation and validation

**Key Tasks**:
- Validates required OAuth parameters
- Requests access token using client credentials
- Stores token for subsequent API calls
- Provides debug output for token information

**Variables Required**:
- `tenant_url`: API endpoint URL
- `client_id`: OAuth client ID
- `client_secret`: OAuth client secret
- `token_endpoint`: Token endpoint path (default: /oauth2/token)

### 2. application_management

**Purpose**: Manages application lifecycle operations

**Key Tasks**:
- Lists all applications from the API
- Retrieves detailed information for each application
- Processes applications using loop constructs
- Provides comprehensive debug output

**Loop Implementation**:
```yaml
loop: "{{ applications_list }}"
loop_control:
  loop_var: current_application
  index_var: app_index
```

### 3. attribute_management

**Purpose**: Manages attribute CRUD operations

**Key Tasks**:
- Lists all attributes from the API
- Retrieves detailed information for each attribute
- Processes attributes with loop constructs
- Removes ID field for migration preparation

**Loop Implementation**:
```yaml
loop: "{{ attributes_list }}"
loop_control:
  loop_var: current_attribute
  index_var: attr_index
```

### 4. data_migration

**Purpose**: Migrates data from source to destination

**Key Tasks**:
- Validates migration prerequisites
- Migrates items sequentially with retry logic
- Provides detailed migration status
- Implements pause between migrations

**Loop Implementation**:
```yaml
loop: "{{ migration_data }}"
loop_control:
  loop_var: migration_item
  index_var: migration_index
```

---

## Playbook Execution

### Execution Flow

1. **Phase 1: Source Environment**
   - Authenticate with source API
   - Retrieve applications list
   - Retrieve attributes list
   - Store data for migration

2. **Phase 2: Destination Environment**
   - Authenticate with destination API
   - Prepare for data reception

3. **Phase 3: Data Migration**
   - Migrate attributes to destination
   - Validate each migration
   - Provide comprehensive summary

### Debug Statements

Each role includes debug statements at key points:

```yaml
- name: "Debug - Operation info"
  ansible.builtin.debug:
    msg:
      - "Operation: {{ operation }}"
      - "Status: {{ status }}"
```

Enable debug output:
```bash
ansible-playbook ... -e "enable_debug=true"
```

---

## Troubleshooting

### Common Issues

#### 1. Authentication Failures

**Symptom**: OAuth token request fails

**Solution**:
```bash
# Verify credentials
echo $SRC_CLIENT_ID
echo $SRC_CLIENT_SECRET

# Check connectivity
curl -X POST https://source-api.example.com/oauth2/token
```

#### 2. Connection Timeouts

**Symptom**: API requests timeout

**Solution**:
```yaml
# Increase timeout in group_vars/all.yml
http_timeout: 60
retry_count: 5
```

#### 3. Migration Failures

**Symptom**: Items fail to migrate

**Solution**:
- Check destination API logs
- Verify data format compatibility
- Review debug output with `-vvv`

### Debug Mode

Enable comprehensive debugging:

```bash
# Maximum verbosity
ansible-playbook -i inventory/hosts.yml playbooks/api_management_main.yml -vvv -e "enable_debug=true"

# Save output to file
ansible-playbook ... 2>&1 | tee execution.log
```

---

## Corporate Standards

### Code Standards

1. **Naming Conventions**
   - Use descriptive task names with role prefix
   - Follow snake_case for variables
   - Use UPPERCASE for constants

2. **Documentation**
   - Every task must have a descriptive name
   - Complex logic requires inline comments
   - All roles must have README files

3. **Error Handling**
   - Implement retry logic for API calls
   - Use assertions for validation
   - Provide meaningful error messages

4. **Security**
   - Never hardcode credentials
   - Use environment variables or vault
   - Validate SSL certificates

### Best Practices

1. **Idempotency**: All tasks should be idempotent
2. **Modularity**: Use roles for reusable components
3. **Logging**: Comprehensive debug statements
4. **Testing**: Test in non-production first
5. **Version Control**: Track all changes in Git

---

## Support

For issues or questions:

- **Email**: devops@h1corporation.com
- **Slack**: #ansible-automation
- **Wiki**: https://wiki.h1corporation.com/ansible

---

## License

Copyright © 2026 H1 Corporation. All rights reserved.

This automation framework is proprietary and confidential.

---

## Changelog

### Version 1.0.0 (2026-04-01)
- Initial release
- OAuth token management
- Application management with looping
- Attribute management with looping
- Data migration functionality
- Comprehensive debug statements
- Corporate standard compliance

---

**Document Version**: 1.0.0  
**Last Updated**: April 1, 2026  
**Maintained By**: H1 Corporation DevOps Team