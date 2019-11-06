# Ansible AWX Installation Notes

[TOC]

## LDAP Authentication Setup

#### LDAP User Search
```json
[
 "ou=users,dc=ldap,dc=home,dc=prettybaked,dc=com",
 "SCOPE_SUBTREE",
 "(uid=%(user)s)"
]
```
#### LDAP Group Search
```json
[
 "ou=groups,dc=ldap,dc=home,dc=prettybaked,dc=com",
 "SCOPE_SUBTREE",
 "(objectClass=posixGroup)"
]
```
#### LDAP User Attribute Map
```json
{
 "first_name": "givenName",
 "last_name": "sn",
 "email": "mail"
}
```
#### LDAP Group Type Parameters
```json
{
 "name_attr": "cn"
}
```
#### LDAP User Flags by Group
```json
{
 "is_superuser": [
  "cn=admin,ou=groups,dc=ldap,dc=home,dc=prettybaked,dc=com"
 ]
}
```
#### LDAP Organization Map
```json
{
 "LDAP Organization": {
  "admins": "cn=admin,ou=groups,dc=ldap,dc=home,dc=prettybaked,dc=com",
  "remove_admins": false,
  "remove_users": false,
  "users": [
   "cn=users,ou=groups,dc=ldap,dc=home,dc=prettybaked,dc=com"
  ]
 }
}
```
#### LDAP Team Map
```json
{
 "LDAP Admins": {
  "organization": "LDAP Organization",
  "users": "cn=admin,ou=groups,dc=ldap,dc=home,dc=prettybaked,dc=com",
  "remove": true
 },
 "LDAP Users": {
  "organization": "LDAP Organization",
  "users": "cn=users,ou=groups,dc=ldap,dc=home,dc=prettybaked,dc=com",
  "remove": true
 }
}
```

## Backup/Restore

#### Install the Tower command-line utility
```bash
pip3 install ansible-tower-cli
```
#### Set up API endpoint and login credentials
```bash
tower-cli config host http://10.0.10.12:8080
tower-cli config username admin
tower-cli config password <pw>
tower-cli config insecure True
```
#### Generate Backup
```bash
tower-cli receive --all > `date +%s`.json
```

## Using Vault

#### Create a new project containing a group_vars-like directory structure
```yaml
vaults/<group1>/vault.yml
vaults/<group2>/vault.yml
```
#### Modify your playbooks to include the proper vault file
 ```yaml
- name: Example Playboook
  hosts: all
  remote_user: root
  vars_files:
    - "/var/lib/awx/projects/vaults/{{ vault_group }}/vault.yml"
  tasks:
  ...
 ```
#### Assign a vault group to your hosts/groups/job template/etc.
 ```yaml
 vaultgroup: <group1>
 ```
