pure-ftpd
=========

This is a role for installing an FTP server. It can automatically create users and generate passwords.
Supported OS: SmartOS


Features:
---------
- The role is fully idempotent
- Fully modifiable pure-ftpd configuration settings
- Customisable config dir path
- Virtual users support
- User passwords are generated automatically if necessary
- Customisable generated passwords complexity
- User removal is fully supported (just remove the user from `pureftpd_users`)
- Nonexistent user homes are created after first login
- TLS support

Configuration variables:
------------------------
```
pureftpd_local_passdb: "~/.ans_pureftpd_passdb"        # a directory on the ansible control machine which stores the generated pureftpd passwords
pureftpd_pass_len: 10                                  # a length of generated passwords
pureftpd_pass_complexity: 'ascii_letters,digits'       # possible values: ascii_letters,digits,hexdigits,punctuation
pureftpd_default_homes: '/home'
pureftpd_etcdir: '/etc/pure-ftpd'
pureftpd_confvars:                                     # configuration overrides (values must be enclosed in "" to prevent converting to True/False)
pureftpd_ssl_certfile: ...                             # SSL certificate to use when TLS is enabled (file must contain KEY and CERT in PEM format)
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
      pureftpd_ssl_certfile: '/etc/openssl/mycert-combined.pem'
      pureftpd_confvars:
        TLS: 2        # '0' to disable TLS; '1' to make TLS optional; '2' to require TLS
        CreateHomeDir: "no"
        IPV4Only: "no"
        PAMAuthentication: "yes"
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
Multiple OS support can be added if someone is interested.
