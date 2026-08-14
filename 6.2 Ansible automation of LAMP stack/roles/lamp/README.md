# LAMP

`LAMP` (Apache+ Maven+ Php) is an ***Ansible*** role that deploys a ***LAMP*** stack on Debian/RedHat Linux system.


## Features 

- [x] installation of the required packages according to the system
- [x] support for Debian/Ubuntu and RedHat/Rocky
- [x] Apache configuration
- [x] using templates for apache configuration
- [x] using variables for service configurations
- [x] use of handlers to restart Apache when necessary
- [x] creation of User and Database of application
- [x] using Ansible vault for Database secret data
- [x] initialization and startup of MariaDB



## Directory Structure

- tasks: Main installation and configuration tasks
	- tasks/install.yml : package installation and directory creation
	- tasks/mariadb.yml : MariaDB initialization and configuration
	- tasks/website.yml : Website configuration and deployment
	- tasks/apache.yml : Apache configuration
- handlers: Service restart handlers
- vars: Role variables
- defaults: Default configurable variables
- templates: 
	- templates/apache.conf.j2 : Apache configuration
	- templates/db.php.j2 : php connection to database
	- templates/index.php.j2 : Apache php web template
- meta: Role metadata
- README.md: Role documentation


## Variables 

These variables are defined in `defautls/main.yml`

- `lamp_root` : root directory of LAMP site
- `site_name` : site name
- `lamp_db_name` : database name
- `lamp_db_user` : MariaDB user
- `lamp_db_host` : MariaDB host
- `lamp_db_port` : MariaDB port 
- `lamp_server_name` : Apache server name
- `lamp_apache_listen_port` : Apache listening port

Requirements
------------

- Ansible `>= 2.9`
- Python on the target server
- SSH access to the target server with priveleges
- Target server os: Debian based or RHEL



## How to use playbook

Deploy LAMP as a a role

```yaml
---
- name: Deploy lamp as role
  hosts: all
  become: true

  roles:
    - lamp
```

Deploy lamp with customized variables

```yaml
---
- name: Deploy lamp with customized variables
  hosts: all
  become: true
  roles:
    - role: lamp
      vars:
        lamp_db_name: test_db
        lamp_db_user: test_user
        lamp_db_password: "test123"
        lamp_root: /var/www/mysite
	site_name: "test"
```

for simple execute

```bash
ansible-playbook -i inventory ansible-playbook-lamp.yml -kK --ask-vault-pass 

```

License
-------

MIT

Author Information
------------------

Ruhollah Jahanafrooz - COPYRIGHT:copyright: 2026.

