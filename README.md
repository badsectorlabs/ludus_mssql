# Ansible Role: MSSQL

An Ansible role that installs [MSSQL](https://learn.microsoft.com/en-us/sql/sql-server/?view=sql-server-ver16) on Windows systems.

## Requirements

None

## Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

    ludus_mssql_version: "2019"
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
  - vm_name: "{{ range_id }}-MSSQL"
    hostname: "{{ range_id }}-MSSQL"
    template: win2019-server-x64-template
    vlan: 10
    ip_last_octet: 13
    ram_gb: 4
    cpus: 2
    windows:
      sysprep: true
    roles:
      - badsectorlabs.ludus_mssql
```

## Ludus setup

```
ludus ansible roles add badsectorlabs.ludus_mssql
ludus range config get > config.yml
# Edit config to add the role to the VMs you wish to install MSSQL on and define your desired MSSQL variables (see above)
ludus range config set -f config.yml
ludus range deploy -t user-defined-roles
```

## License

GPLv3

## Author Information

This role was created by [Bad Sector Labs](https://badsectorlabs.com/), for [Ludus](https://ludus.cloud/).
