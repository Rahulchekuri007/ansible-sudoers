# wtcross.sudoers
An Ansible role for configuring the /etc/sudoers file and /etc/sudoers.d files.

This role makes it possible to completely define your sudoers configuration with Ansible. All of the following are configurable:
- defaults
- aliases
  * Users
  * Runas
  * Hosts
  * Commands
- specifications

*Tip:* Here's a [great document about sudoers configuration](https://help.ubuntu.com/community/Sudoers)

## About and Usage
The top level `/etc/sudoers` file can be kept as light as possible by specifying sudoer_separate_specs: True in either the defaults or your playbook. sudoer_separate_specs is set to True by default.

***Warning, this role will clean out /etc/sudoers.d/ if sudoer_separate_specs is set to false. You will lose any files stored there even if not generated by this role.***

If sudoer_separate_specs is set to true, it will  include all defaults and aliases in /etc/sudoers rather than breaking the specs out into their own files in /etc/sudoers.d/.

All sudoer specifications will each be placed in their own file within the `/etc/sudoers.d/` directory. A specification consists of the following:
- `name`: the name of the specification (file name in `/etc/sudoers.d/`)
- `users`: user list or user alias
- `hosts`: host list or host alias
- `operators`: operator list or runas alias
- `commands`: command list or

The following properties are optional:
- `tags`: list of tags (ex: NOPASSWD)
- `comment`: A comment you'd like to add to your spec for clarity

Valid sudoer tags are: NOPASSWD, PASSWD, NOEXEC, EXEC, SETENV, NOSETENV, LOG_INPUT, NOLOG_INPUT, LOG_OUTPUT and NOLOG_OUTPUT.

User/Group specific defaults can be added to the defaults list by a preceding ':' followed by the user/group whitespace then the option.  For example:

```yaml
---
sudoer_defaults:
  - :MONITOR_USER     !logfile
```

This will generate a line:

```
Defaults:MONITOR_USER    !logfile
```


## Example Playbook
```yaml
- hosts: all
  vars:
    sudoer_aliases:
      user:
            - name: ADMINS
              comment: Group of admin users
          users:
            - %admin
          runas:
            - name: ROOT
              comment: Root stuff
          users:
            - '#0'
      host:
            - name: SERVERS
              comment: XYZ servers
          hosts:
            - 192.168.0.1
            - 192.168.0.2
      command:
            - name: ADMIN_CMNDS
              comment: Stuff admins need
              commands:
            - /usr/sbin/passwd
            - /usr/sbin/useradd
            - /usr/sbin/userdel
            - /usr/sbin/usermod
            - /usr/sbin/visudo
    sudoer_specs:
      - name: administrators
            comment: Stuff for admins
            users: ADMIN
        hosts: SERVERS
        operators: ROOT
        tags:
          - NOPASSWD
        commands: ADMIN_CMNDS
        defaults:
          - '!requiretty'

  roles:
    - wtcross.sudoers
```

## Defaults:
```yaml
sudoer_aliases: {}
sudoer_specs: []
sudoer_defaults:
 #  - requiretty (disabled, just uncomment if required)
  - "!visiblepw"
  - always_set_home
  - env_reset
  - env_keep:
   - COLORS
   - DISPLAY
   - HOSTNAME
   - HISTSIZE
   - INPUTRC
   - KDEDIR
   - LS_COLORS
   - MAIL
   - PS1
   - PS2
   - QTDIR
   - USERNAME
   - LANG
   - LC_ADDRESS
   - LC_CTYPE
   - LC_COLLATE
   - LC_IDENTIFICATION
   - LC_MEASUREMENT
   - LC_MESSAGES
   - LC_MONETARY
   - LC_NAME
   - LC_NUMERIC
   - LC_PAPER
   - LC_TELEPHONE
   - LC_TIME
   - LC_ALL
   - LANGUAGE
   - LINGUAS
   - _XKB_CHARSET
   - XAUTHORITY
  - secure_path: /sbin:/bin:/usr/sbin:/usr/bin
sudoer_separate_specs: True
```

## Requirements
The host operating system must be a member of one of the following OS families:

- Debian
- RedHat
- SUSE

## Dependencies
None

## Variable Schemas
```yaml
# host alias
name: string
hosts: string|[hostnames]
comment: string #procedes the alias with a comment

# user alias
name: string
users: string|[username|%group]
comment: string #procedes the alias with a comment

# runas alias
name: string
users: string|[username|%group|#uid]
comment: string #procedes the alias with a comment

# cmnd alias
name: string
commands: string|[string]
comment: string #procedes the alias with a comment

# sudoer specification
name: string
users: string|[string]
hosts: string|[string]
operators: string|[string]
tags: string|[string]
comment: string #procedes the alias with a comment
defaults: string|[string]

## Role Variables
- `sudoer_aliases`: a dictionary that specifies which aliases to configure
  - `sudoer_aliases.host`: a list of host alias descriptions
  - `sudoer_aliases.user`: a list of host alias descriptions
  - `sudoer_aliases.runas`: a list of host alias descriptions
  - `sudoer_aliases.command`: a list of host alias descriptions
- `sudoer_specs`: a list of sudoer specifications
- `sudoer_defaults`: a list of default settings
  - can be any of the following types
    - `string`
    - `string: string`
    - `string: [string]`
```

## License
[MIT](LICENSE)

## Author Information
[Tyler Cross](https://github.com/wtcross)
[Andrew J. Huffman](https://github.com/ahuffman)
