
# WAM

`WAM` (Wordpress+ Apache+ Maven) is an ***Ansible*** role that deploys a ***Wordpress*** site along a ***MariaDB*** database on an ***Apache*** webserver. The role is able to deploy this stack on ***Debian*** or ***RHEL*** systems.


## Features

- [x] installation of the required packages according to the system
- [x] support for Debian/Ubuntu and RedHat/Rocky
- [x] Apache/ Wordpress configuration template
- [x] download and installation of WordPress
- [x] using variables for service configurations
- [x] use of handlers to restart Apache when necessary
- [x] connection of Wordpress with MariaDB 

## Directory Structure

- tasks: Main installation and configuration tasks
    - tasks/install.yml : package installation and directory creation
    - tasks/mariadb.yml : MariaDB initialization and configuration
    - tasks/wordpress.yml : WordPress download and configuration
    - tasks/apache.yml : Apache configuration
- handlers: Service restart handlers
- vars: Role variables
- defaults: Default configurable variables
- templates: Jinja2 configuration templates
    - templates/wp-config.php.j2 : WordPress configuration
    - templates/wordpress.conf.j2 : Apache configuration
- meta: Role metadata
- README.md: Role documentation


## Variables

These variables are defined in `defautls/main.yml`

- `wordpress_site_name` : logical name of the site
- `wordpress_download_url` : WordPress download URL
- `wordpress_archive_path` : Local path of the downloaded archive
- `wordpress_install_parent` : parent installation directory
- `wordpress_install_dir` : final WordPress site directory
- `wordpress_db_name` : database name
- `wordpress_db_user` : MariaDB user
- `wordpress_db_password` : MariaDB password
- `wordpress_db_host` : MariaDB host
- `wordpress_db_user_host` : authorized host for the MariaDB user
- `wordpress_db_port` : MariaDB port
- `wordpress_server_name` : Apache server name
- `wordpress_apache_listen_port` : Apache listening port
- `wordpress_mariadb_data_dir` : MariaDB data directory


Requirements
------------

- Ansible `>= 2.9`
- Python on the target server
- SSH access to the target server with priveleges
- Target server os: Debian based or RHEL





## How to use playbook

Deploy wam as a a role

```yaml
---
- name: Deploy wam as role
  hosts: all
  become: true
  roles:
    - wam
```

Deploy wam with customized variables

```yaml
---
- name: Deploy wam with customized variables
  hosts: all
  become: true
  roles:
    - role: wam
      vars:
        wordpress_db_name: test
        wordpress_db_user: test_user
        wordpress_db_password: "123"
        wordpress_install_dir: /var/www/mysite
```

for simple execute

```bash
ansible-playbook -i inventory wam-playbook.yml -kK
```

License
-------

MIT

Author Information
------------------

Ruhollah Jahanafrooz - COPYRIGHT :copyright: 2026.

