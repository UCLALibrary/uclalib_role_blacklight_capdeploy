uclalib_role_blacklight_capdeploy &nbsp;[![Build Status](https://travis-ci.org/UCLALibrary/uclalib_role_samvera_capdeploy.svg?branch=master)](https://travis-ci.org/UCLALibrary/uclalib_role_samvera_capdeploy)
=========

Ansible role to perform an initial code deploy to a UCLA Library Blacklight system using Capistrano

Requirements
------------

This role handles the __first deployment__ of Blacklight project code from a git repository to the rails application server.

Please take note of the following assumptions:
* the rails application server uses Red Hat Enterprise Linux 7 for the OS
* a Solr 7.X server is available with the index core created and project-specific configuration files installed in the core
* a MySQL database server is available with the project-specific database created and user account/privileges established
* a git repository is available containing the project's code
* project-specific variables for this role can be defined in a __vars__ file with a name following the format of `projectname_envname.yml`
    * an example vars file is available in `vars/exampleproj_test.yml`
    * this vars file will contain sensitive information and should be encrypted with ansible-vault
    * NOTE: if you choose not to use the vars file for including the variable definitions, they should be defined in the playbook file

Role Variables
--------------
Variables the need to be defined in the **play file** or the **host inventory file** - please note these should match the naming used for the vars file:
* `project_name` - defines the name of the rails application project - there is no default value
* `env_name` - defines the name of the deploy environment (e.g. test, stage, prod) - there is no default value

Variables with default values that **do not** need to be defined in the project vars file:
* `capistrano_user` - defines the user account that will perform the code deploy - default is `deploy`
* `capistrano_base` - defines the top-level directory where the project will be deployed - default is `/opt`
* `logrotate_files` - defies the list of files to include in the rails application logrotate configuration file - defaults are `production.log` and `sidekiq.log`

Variables that **do** need to be defined in the project vars file:
* `rails_host_fqdn` - defines the fully qualified domain name of the rails app server
* `rails_db_adapter` - defines the type of database back-end (`mysql2` or `postgresql`) - please note that currently this role only supports a db backend of `mysql2`
* `rails_db_pool` - defines the number of connections to maintain to the database
* `rails_db_host` - defines the hostname of the database server
* `rails_db_name` - defines the name of the project's database
* `rails_db_user` - defines the user account with access to the database
* `rails_db_pass` - defines the password for the database user
* `project_default_admin_password` - defines the rails app admin password that will be created at first deploy
* `contact_email_addr` - defines the email address for a project contact
* `solr_index_url` - defines the HTTP url to the project's Solr core
* `redis_host` - defines the hostname of the server running Redis
* `redis_port` - defines the port number where Redis is reachable
* `sidekiq_num_threads` - defines the number of Sidekiq threads to maintain
* `thumbnail_base_url` - defines the base URL path where Ursus should retrieve thumbnail image data (e.g. https://californica.example.com/)
* `feature_flag` - defines OPTIONAL feature flag to use during the deployment process
* `external_iiif_server_url` - defines the URL path where Ursus should retrieve IIIF server data (e.g. https://californica.example.edu/concern/works/)
* `google_tag_manager_id` - (optional) - defines a Google Tag Manager key to be used within the server environment
* `rollbar_api_token` - (optional) - defines the api token to use with the Rollbar error reporting tool
* `recaptcha_site_key` - (optional) defines the site key to use with the recaptcha service
* `recaptcha_secret_key` - (optional) defines the secret key to use with the recaptcha service
* `email_from_address` - defines the email address used in conjunction with the contact form where email will be sent from
* `email_to_address` - defines the email address used in conjunction with the contact form where email will be sent to
* `email_server_url` - the url of the SMTP server email should be sent
* `email_port` - the port number of the SMTP email server
* `git_repo_url` - defines the HTTP url to the project's git repository
* `git_repo_branch` - defines the name of the project's git branch to deploy
* `allow_web_crawl` - defines if search engines are allowed to crawl this site (yes or no)
* `ssh_pub_keys` - defines the list of ssh public keys to install in the deploy user's authorized_keys file; this allows for other users to perform deployments
* `use_deflate_module` - defines if we should use the deflate_module in the Apache HTTPD vhost configuration (default is `yes`)
* `sinai_id_url` - (optional) Sinai Auth URI
* `sinai_id_key` - (optional) Sinai Auth API Token
* `sinai_cipher` - (optional) Sinai Auth Password
* `sinai_cookie_domain` - (optional) Sinai Auth Cookie Domain

An example vars file is available as a part of this role, named `exampleproj_test.yml`

Variables with default values that define if this deployment should use SSL
For a local dev deployment, default values enable SSL and install self-signed certificates
For a production install, defaults values are overriden by variables defined in host_vars
* `enable_ssl` - defines if this deloyment should use SSL (`yes` or `no` - default is `yes`)
* `ssl_cert_base_path` - defines the base path to the SSL certs and key
* `ssl_cert_file_path` - defines the path to the SSL certs
* `ssl_key_file_path` - defines the path to the SSL private key
* `ssl_files`
    * `self_signed` - defines if the certificates are self-signed (`yes` or `no` - default is `yes`)
    * `crt` - contains the contents of the SSL public certificate
    * `interm` - contains the contents of the SSL intermediate chain certificate (only needed if using a trusted cert)
    * `key` - contains the contents of the SSL private key

Dependencies
------------

The following roles must be run on the rails application server prior to executing this deploy role:

```
uclalib_role_rhel7repos
uclalib_role_epel
uclalib_role_uclalibrepo
uclalib_role_samvera_basepackages
uclalib_role_pip
uclalib_role_ruby
uclalib_role_apache
uclalib_role_passenger
uclalib_role_nodejs
uclalib_role_yarn
uclalib_role_redis
```

Example Playbook
----------------
```
---

- name: uclalib_capdeploy.yml
  become: yes
  become_method: sudo
  hosts: test
  user: ansible
  vars:
    project_name: "ursus"
    env_name: "test"

  roles:
    - { role: uclalib_role_rhel7repos }
    - { role: uclalib_role_epel }
    - { role: uclalib_role_uclalibrepo }
    - { role: uclalib_role_samvera_basepackages }
    - { role: uclalib_role_pip }
    - { role: uclalib_role_ruby, ruby_version: '2.5.1' }
    - { role: uclalib_role_apache }
    - { role: uclalib_role_passenger, passenger_version: '5.3.3' }
    - { role: uclalib_role_nodejs, nodejs_version: '8' }
    - { role: uclalib_role_yarn }
    - { role: uclalib_role_redis }
    - { role: uclalib_role_blacklight_capdeploy }
```

Developers' Box
-------------------------------

The `uclalib_role_blacklight_capdeploy` role can also be used to build a developers' box that mimics the environment used in production. To do this, slight changes are needed in the way that the role is run. The need for these changes is a result of having tagged tasks in the `ansible_user_env_setup.yml`, `capistrano_deploy.yml`, and `dotenv_setup.yml` files, which are specific to a developers' box build. These are ignored for production builds.

To trigger the running of tasks that are only needed for a developers' box, supply the following two arguments when you run your Ansible playbook:

    --skip-tags "always" --tags "untagged,development"

For a production build, you would omit these arguments and the tasks that are intended only for the developers' box would be, by default, skipped.

Running the build with the above `skip-tags` and `tags` arguments ensures that the project .env files for a developers' box are created, that the test database is created, and that the GitHub source is kept on the machine for the developer to use.
