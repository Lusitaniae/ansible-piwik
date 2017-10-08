# Ansible Role to install Piwik

[![Build Status](https://travis-ci.org/Lusitaniae/ansible-piwik.svg?branch=master)](https://travis-ci.org/Lusitaniae/ansible-piwik)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/Lusitaniae/ansible-piwik/master/LICENSE)

This role will install Piwik for a cloud environment.

Install Ansible on its development version, for best python3 compatibility.


## Infrastructure:
1. EC2 instance - instance to be installed Piwik.

2. RDS instance - managed Mysql (or compatible) instance.

3. Elastic Cache - managed Redis instance.

Setup the appropriate Security Group so that EC and RDS are able to receive connections from EC2.

You can use the finished EC2 instance to make an AMI and use it in a Auto Scaling group.

## Installation
1. Clone this repository into your **roles** directory of your Ansible installation via:

   ```git clone https://github.com/hicknhack-software/ansible-piwik.git piwik```

2. Download the ssh key that will be used for connections.
3. Create a host file with the following details:

  ```
  [piwik]
  # Setup an Elastic IP and replace with your own.
  # Make sure that the private key name matches.
  80.80.80.80 ansible_ssh_user=ubuntu ansible_ssh_private_key_file=my_private_key.pem


  [piwik:vars]
 ansible_python_interpreter=/usr/bin/python3

  # change to your domain where piwik is hosted
  piwik_webpage_url=stats.example.com

  # change to the name of the webpage you'd like to track
  # you can add another later in the dashboard of piwik
  piwik_webpage_name=exammple.com

  # change to the domain or ip address of your piwik installation
  piwik_trusted_host=stats.example.com

  [mysql:children]
  piwik
  ```

4. Add a folder named `group_vars` as sibling to `roles`. Add a file named `piwik.yml` with the following example content:

  ```
  # Installation Details
  app_name: piwik
  app_user: deploy
  app_path: /var/www

  # Piwik Host
  server_name: stats.example.com
  piwik_admin_user: maestro

  # Remote Mysql
  mysql_host: my-piwik-cluster-123456789.eu-west-1.rds.amazonaws.com
  mysql_admin_user: maestro
  mysql_piwik_user: piwik
  mysql_piwik_db: piwik_db
  mysql_service: mysql

  # Remote Redis
  redis_host: my-piwik-cluster-123456789.euw1.cache.amazonaws.com

  ```
5. Create an encrypted Ansible Vault with `ansible-vault edit group_vars/secrets.yml ` and add the credentials for your piwik setup, like the following example:

    ```
    # Piwik Credentials
    # Generate a new hash with:
    # php -r 'echo password_hash(md5("my-password"), PASSWORD_DEFAULT) . PHP_EOL;'
    piwik_admin_password: $2y$10$Qcw6wAYjfVbHczecTQ7zVOmPb5c5NlLqhhgqnv0RO9iEu/gZILfnS
    piwik_admin_token: c449fb552e7d39fe6ae22d8a02f30e1a

    # Mysql root and user credentials
    mysql_admin_pass: rEMcrn$Dg6Cs
    mysql_piwik_pass: s3cR37p1w1k
    ```

6. Add an Ansible playbook with the following example content:

  ```
  - hosts: piwik
    sudo: true
    sudo_user: root
    vars:
      provisioned: yes
    roles:
      - role: piwik/prepare/user
        user_authorized_key: "{{ lookup('file', 'my_private_key.pub') }}"
      - piwik/prepare/environment
      - piwik/prepare/mysql
      - piwik/prepare/php
      - piwik/prepare/nginx

  - hosts: piwik
    sudo: true
    sudo_user: "{{ app_user }}"
    vars:
      gather_facts: true
      was_provisioned: "{{ provisioned is defined }}"
    roles:
      - role: piwik/install
  ```

6. Start your installation with: `ansible-playbook site.yml --ask-vault-pass`

## License

The MIT License (MIT)

Copyright (c) 2015 HicknHack Software GmbH

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
