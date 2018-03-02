pure-ftpd
=========

This is a role for installing an FTP server. It can automatically create users and generate passwords.
Supported OS: SmartOS

The role is fully idempotent.

Configuration variables:
------------------------
```
pureftpd_local_passdb: "~/.ans_pureftpd_passdb"        # a directory on the ansible control machine which stores the generated pureftpd passwords
pureftpd_pass_len: 10                                  # a length of generated passwords
pureftpd_pass_complexity: 'ascii_letters,digits'       # possible values: ascii_letters,digits,hexdigits,punctuation
pureftpd_default_homes: '/home'
pureftpd_users:
  user1:
    uid: 1234       # mandatory
    gid: 5678       # if not defined, defaults to uid
    dir: /ftp/user1 # if not defined, defaults to {{pureftpd_default_homes}}/{{user_name}}
    chrooted: true  # optional, default true
    passwd: 'mypwd' # optional, otherwise generated automatically according to config rules
```
Password specified in `pureftpd_users` take precedense before auto-generated ones. Generated passwords are stored on *ansible master host* (yes, the one that executed the ansible command) in `pureftpd_local_passdb` directory using following pattern:
```
{{ pureftpd_local_passdb }}/{{ machine_inventory_name }}/{{ user_name }}
Example:
> cat /root/.ans_pureftpd_passdb/mysrv1/user22
```

Example configuration:
----------------------
```
- name: install pureftpd
  hosts: purehost
  gather_facts: True
  any_errors_fatal: True
  vars:
      pureftpd_default_homes: '/ftpdata'
      pureftpd_users:
          myuser:
              uid: 1122
              chrooted: false
              passwd: 'aaa'
          youruser:
              uid: 1123
              gid: 2222
              dir: /tmp
          simpleuser:
              uid: 2000
  pre_tasks:
  roles:
    - pure-ftpd
```

Future work
-----------
Multiple OS support can be added if someone is interested. Also more configuration options (actually all of them) can be added as role variables.
Other plans:
* add TLS support with Let's Encrypt
* create and setup user homes
