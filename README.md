# Ansible Role: MSSQL

An Ansible role that installs [MSSQL](https://learn.microsoft.com/en-us/sql/sql-server/?view=sql-server-ver16) on Windows systems with support for TCP enablement, mixed mode authentication, SQL and AD logins, linked servers, impersonation (login-level and database-level), SA account management, sysadmin configuration, xp_cmdshell, and SSMS installation.

## Requirements

None

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

### General Installation

    ludus_install_directory: /opt/ludus
    ludus_mssql_iso_directory: "C:\\ludus"
    # Valid ludus_mssql_version options are "2019" or "2022"
    ludus_mssql_version: "2019"
    ludus_mssql_sql_config_path: "{{ ludus_mssql_iso_directory }}\\sqlsrv_{{ ludus_mssql_version }}_config.ini"
    ludus_mssql_iso_url: https://archive.org/download/en_sql_server_2019_standard_x64_dvd_814b57aa/en_sql_server_2019_standard_x64_dvd_814b57aa.iso
    ludus_mssql_iso_checksum: "sha256:1e56705b3544e77039584b3b38461df0321834822776aef8e50847fdd9edad44"
    ludus_mssql_instance_name: MSSQLSERVER
    ludus_mssql_sql_license_key:
    ludus_mssql_ssms_url: "https://aka.ms/ssmsfullsetup"
    ludus_mssql_install_ssms: true
    ludus_mssql_enable_tcp: true
    ludus_mssql_mixed_mode_auth: false
    ludus_mssql_enable_xp_cmdshell: false

### Service Account

    # SQL Server service account (templated into config INI)
    # Default: local "NT Authority\\Network Service"
    # Set to "DOMAIN\\username" for a domain service account
    ludus_mssql_svc_account: "NT Authority\\Network Service"

### Pre-Install Fixes

    # Fix .NET machine.config XML corruption before SQL install
    # Prevents IIS/ADCS installations from breaking SQL Server installer
    # Safe no-op on clean systems
    ludus_mssql_fix_dotnet_config: true

### SA Account

    # Enable the SA account and set its password
    ludus_mssql_enable_sa: false
    ludus_mssql_sa_password: ""

### Firewall

    # Open UDP 1434 for SQL Browser discovery (only when enable_tcp is true)
    ludus_mssql_enable_udp_discovery: true

### SQL Logins

    # SQL logins to create (requires ludus_mssql_mixed_mode_auth: true)
    ludus_mssql_sql_users: []
    # Example:
    # ludus_mssql_sql_users:
    #   - username: "sqladmin"
    #     password: "Password123!"
    #     sysadmin: true

### AD Logins

    # Active Directory Windows logins to create
    ludus_mssql_ad_logins: []
    # Example:
    # ludus_mssql_ad_logins:
    #   - username: "domain\\user"
    #     sysadmin: true

### Sysadmins (Shorthand)

    # Add Windows accounts directly as sysadmins (creates login + adds to sysadmin role)
    ludus_mssql_sysadmins: []
    # Example:
    # ludus_mssql_sysadmins:
    #   - "DOMAIN\\sql_svc"
    #   - "DOMAIN\\jon.snow"

### Login-Level Impersonation (EXECUTE AS LOGIN)

    # Server-scoped impersonation grants between logins
    ludus_mssql_impersonations: []
    # Example:
    # ludus_mssql_impersonations:
    #   - grantor: "sa"
    #     grantee: "DOMAIN\\samwell.tarly"

### Database-Level Impersonation (EXECUTE AS USER)

    # Database-scoped impersonation: login → DB user → IMPERSONATE ON USER
    ludus_mssql_impersonation_users: []
    # Example (arya.stark can impersonate dbo in msdb):
    # ludus_mssql_impersonation_users:
    #   - login: "DOMAIN\\arya.stark"
    #     database: "msdb"
    #     impersonate_user: "dbo"

### Linked Servers

    # Linked servers for cross-domain/forest MSSQL trust exploitation
    ludus_mssql_linked_servers: []
    # Example:
    # ludus_mssql_linked_servers:
    #   - name: "DB-2"
    #     provider: "MSOLEDBSQL"
    #     data_source: "10.10.10.20"
    #     remote_user: "domain\\user"
    #     remote_password: "password"

## Dependencies

None

## Task Execution Order

1. Fix .NET machine.config corruption (if `ludus_mssql_fix_dotnet_config: true`)
2. Install MSSQL (skipped if already installed)
3. Enable mixed mode authentication (if `ludus_mssql_mixed_mode_auth: true`)
4. Create SQL logins (if `ludus_mssql_sql_users` defined)
5. Create AD logins (if `ludus_mssql_ad_logins` defined)
6. Configure login-level impersonations (if `ludus_mssql_impersonations` defined)
7. Configure database-level impersonation users (if `ludus_mssql_impersonation_users` defined)
8. Add sysadmin members (if `ludus_mssql_sysadmins` defined)
9. Enable TCP and firewall rules (if `ludus_mssql_enable_tcp: true`)
10. Configure linked servers (if `ludus_mssql_linked_servers` defined)
11. Enable xp_cmdshell (if `ludus_mssql_enable_xp_cmdshell: true`)
12. Enable SA account (if `ludus_mssql_enable_sa: true` or `ludus_mssql_sa_password` is set)
13. Install SSMS (skipped if already installed)

## Example Playbook

```yaml
- hosts: mssql_hosts
  roles:
    - badsectorlabs.ludus_mssql
```

## Example Ludus Range Config (Basic)

```yaml
ludus:
  - vm_name: "{{ range_id }}-DB-1"
    hostname: "DB-1"
    template: win2022-server-x64-template
    vlan: 30
    ip_last_octet: 14
    ram_gb: 4
    cpus: 2
    windows:
      sysprep: true
    roles:
      - badsectorlabs.ludus_mssql
    role_vars:
      ludus_mssql_version: "2019"
      ludus_mssql_enable_tcp: true
      ludus_mssql_mixed_mode_auth: true
      ludus_mssql_enable_xp_cmdshell: true
      ludus_mssql_sql_users:
        - username: "sqladmin"
          password: "Password123!"
          sysadmin: true
        - username: "appuser"
          password: "Password123!"
          sysadmin: false
      ludus_mssql_ad_logins:
        - username: "domain\\user"
          sysadmin: false
        - username: "domain\\user2"
          sysadmin: true
      ludus_mssql_impersonations:
        - grantor: "sqladmin"
          grantee: "appuser"
      ludus_mssql_linked_servers:
        - name: "DB-2"
          provider: "MSOLEDBSQL"
          data_source: "10.{{ range_second_octet }}.10.20"
          remote_user: "domain\\administrator"
          remote_password: "password"
```

## Example Ludus Range Config (GOAD — Attack Paths)

This example configures MSSQL with GOAD-style attack paths including impersonation chains, linked servers, and SA account access:

```yaml
ludus:
  # MSSQL Server 1 — castelblack (srv02 in GOAD)
  - vm_name: "{{ range_id }}-SRV02"
    hostname: "castelblack"
    template: win2022-server-x64-template
    vlan: 10
    ip_last_octet: 22
    ram_gb: 4
    cpus: 2
    windows:
      sysprep: true
    domain:
      fqdn: north.sevenkingdoms.local
      role: member
    roles:
      - badsectorlabs.ludus_mssql
    role_vars:
      ludus_mssql_version: "2019"
      ludus_mssql_enable_tcp: true
      ludus_mssql_mixed_mode_auth: true
      ludus_mssql_install_ssms: false
      # Enable SA with custom password
      ludus_mssql_enable_sa: true
      ludus_mssql_sa_password: "YouWillNotKerboroast1ngMeeeeee"
      # Add domain accounts as sysadmins
      ludus_mssql_sysadmins:
        - "NORTH\\sql_svc"
        - "NORTH\\jon.snow"
      # Login-level impersonation (samwell → sa, brandon → jon.snow)
      ludus_mssql_impersonations:
        - grantor: "sa"
          grantee: "NORTH\\samwell.tarly"
        - grantor: "NORTH\\jon.snow"
          grantee: "NORTH\\brandon.stark"
      # Database-level impersonation (arya → dbo in msdb)
      ludus_mssql_impersonation_users:
        - login: "NORTH\\arya.stark"
          database: "msdb"
          impersonate_user: "dbo"
      # Linked server to essos domain MSSQL
      ludus_mssql_linked_servers:
        - name: "BRAAVOS"
          data_source: "braavos.essos.local"
          remote_user: "sa"
          remote_password: "Admin123!"

  # MSSQL Server 2 — braavos (srv03 in GOAD)
  - vm_name: "{{ range_id }}-SRV03"
    hostname: "braavos"
    template: win2022-server-x64-template
    vlan: 10
    ip_last_octet: 23
    ram_gb: 4
    cpus: 2
    windows:
      sysprep: true
    domain:
      fqdn: essos.local
      role: member
    roles:
      - badsectorlabs.ludus_mssql
    role_vars:
      ludus_mssql_version: "2019"
      ludus_mssql_enable_tcp: true
      ludus_mssql_mixed_mode_auth: true
      ludus_mssql_install_ssms: false
      ludus_mssql_enable_sa: true
      ludus_mssql_sa_password: "Admin123!"
      ludus_mssql_sysadmins:
        - "ESSOS\\sql_svc"
```

## Ludus Setup

```bash
ludus ansible roles add badsectorlabs.ludus_mssql
ludus range config get > config.yml
# Edit config to add the role to the VMs you wish to install MSSQL on
ludus range config set -f config.yml
ludus range deploy -t user-defined-roles
```

## Backward Compatibility

All new features (SA account, database impersonation, sysadmins shorthand, .NET fix, UDP firewall) default to disabled or empty. Existing configurations continue to work with no changes required.

## License

GPLv3

## Author Information

This role was created by [Bad Sector Labs](https://badsectorlabs.com/), for [Ludus](https://ludus.cloud/).
