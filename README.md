# users
Ansible role for managing system users.
## Examples
### inventory:
#### add users:
```yaml
variable_store_username: 'userName'
variable_store_uid: 2000
variable_store_gid: 2000

users_all:
  - user: 'userName'
    realname: 'realName'
    uid: 500
    gid: 500
    system: yes
  - user: '{{ variable_store_user_name }}'
    uid: '{{ variable_store_uid }}'
    gid: '{{ variable_store_gid}}'
```
#### change users for environment or host:
```yaml
users_env:
  - user: 'userName'
    groups: 'sudo'
    password: 'clearText'
```
#### add groups:
```yaml
users_groups_all:
  - group: 'groupName'
    gid: 500
    system: true
```
#### change groups for environment or host:
```yaml
users_groups_env:
  - group: 'groupName'
    system: no
```
### dependencies:
#### add users:
```yaml
dependencies:
  - role: users
    users:
      - user: 'userName'
        realname: 'realName'
        uid: 500
        gid: 500
        system: yes
        shell: /sbin/nologin
      - user: '{{ role_user }}'
        group: '{{ role_group }}'
        shell: '{{ role_shell }}'
        base_dir: '{{ role_work_dir }}'
```
#### add groups:
```yaml
dependencies:
  - role: users
    users_groups:
      - group: '{{ role_group }}'
        gid: 600
        system: true
```

## Users list format
```yaml
users:
  - user: 'userName or {{ variable_store_user_name }}'
    real_name: 'userComment'
    base_dir: 'userBaseDir'
    shell: 'userShell'
    uid: 'userID'
    gid: 'groupID'
    system: 'yes|no|true|false'
    sshkeys:
      - ssh-rsa public_key1 comment
      - ssh-rsa public_key2 comment
    group: 'groupName'
    groups: 'group1,group2'
    update_password: 'on_create'
```
## Groups list format
```yaml
users_groups:
  - group: 'groupName or {{ variable_store_group_name }}'
    gid: 'groupID'
    system: 'yes|no|true|false'
```
