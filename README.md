# Ansible Role: MSSQL

An Ansible role that installs [MSSQL](https://learn.microsoft.com/en-us/sql/sql-server/?view=sql-server-ver16) on Windows systems with support for TCP enablement, mixed mode authentication, SQL and AD logins, linked servers, impersonation, and xp_cmdshell configuration.

## Requirements

None

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    # General
    ludus_install_directory: /opt/ludus
    ludus_mssql_iso_directory: "C:\\ludus"
    # Valid ludus_mssql_version options are "2019" or "2022"
    ludus_mssql_version: "2019"
    ludus_mssql_sql_config_path: "{{ ludus_mssql_iso_directory }}\\sqlsrv_{{ ludus_mssql_version }}_config.ini"
    ludus_mssql_iso_url: https://archive.org/download/en_sql_server_2019_standard_x64_dvd_814b57aa_202211/en_sql_server_2019_standard_x64_dvd_814b57aa.iso
    ludus_mssql_iso_checksum: "sha256:1e56705b3544e77039584b3b38461df0321834822776aef8e50847fdd9edad44"
    ludus_mssql_instance_name: MSSQLSERVER
    ludus_mssql_sql_license_key:
    ludus_mssql_ssms_url: "https://aka.ms/ssmsfullsetup"
    ludus_mssql_install_ssms: true
    # Enable TCP/IP protocol and open Windows Firewall port 1433 (default: false)
    ludus_mssql_enable_tcp: true
    # Enable SQL Server mixed mode authentication (Windows + SQL logins) (default: false)
    ludus_mssql_mixed_mode_auth: false
    # Enable xp_cmdshell (default: false)
    ludus_mssql_enable_xp_cmdshell: false

    # SQL logins to create (requires ludus_mssql_mixed_mode_auth: true)
    ludus_mssql_sql_users: []
    # Example:
    # ludus_mssql_sql_users:
    #   - username: "sqladmin"
    #     password: "Password123!"
    #     sysadmin: true
    #   - username: "appuser"
    #     password: "Password123!"
    #     sysadmin: false

    # Active Directory Windows logins to create
    ludus_mssql_ad_logins: []
    # Example:
    # ludus_mssql_ad_logins:
    #   - username: "domain\\jsmith"
    #     sysadmin: false
    #   - username: "domain\\user"
    #     sysadmin: true

    # Impersonation grants between logins
    ludus_mssql_impersonations: []
    # Example:
    # ludus_mssql_impersonations:
    #   - grantor: "sqladmin"
    #     grantee: "appuser"

    # Linked servers to configure
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

## Example Playbook

```yaml
- hosts: mssql_hosts
  roles:
    - badsectorlabs.ludus_mssql
```

## Example Ludus Range Config

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

## Ludus Setup

```bash
ludus ansible roles add badsectorlabs.ludus_mssql
ludus range config get > config.yml
# Edit config to add the role to the VMs you wish to install MSSQL on
ludus range config set -f config.yml
ludus range deploy -t user-defined-roles
```

## Task Execution Order

1. Install MSSQL (skipped if already installed)
2. Enable mixed mode authentication (if `ludus_mssql_mixed_mode_auth: true`)
3. Create SQL logins (if `ludus_mssql_sql_users` defined)
4. Create AD logins (if `ludus_mssql_ad_logins` defined)
5. Configure impersonations (if `ludus_mssql_impersonations` defined)
6. Enable TCP and firewall rule (if `ludus_mssql_enable_tcp: true`)
7. Configure linked servers (if `ludus_mssql_linked_servers` defined)
8. Enable xp_cmdshell (if `ludus_mssql_enable_xp_cmdshell: true`)
9. Install SSMS (skipped if already installed)

## License

GPLv3

## Author Information

This role was created by [Bad Sector Labs](https://badsectorlabs.com/), for [Ludus](https://ludus.cloud/).