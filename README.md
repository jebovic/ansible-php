php
===

[![Build Status](https://travis-ci.org/jebovic/ansible-php.svg?branch=master)](https://travis-ci.org/jebovic/ansible-php) [![Ansible Galaxy](https://img.shields.io/badge/galaxy-jebovic.php-blue.svg?style=flat)](https://galaxy.ansible.com/jebovic/php)

Install and configure php-fpm for lamp and lemp stack, add your own pools configurations, and customize all the php.ini config from yaml variables

**Bonus** : add extensions for blackfire and newrelic APM

This role is a part of my [OPS project](https://github.com/jebovic/ops), follow this link to see it in action. OPS provides a lot of stuff, like a vagrant file for development VMs, playbooks for roles orchestration, inventory files, examples for roles configuration, ansible configuration file, and many more.

Compatibility
-------------

Tested and approved on :

* Debian jessie (8+)
* Ubuntu Trusty (14.04 LTS)
* Ubuntu Xenial (16.04 LTS)

Role Variables
--------------

```yaml
# use custom repository
php_use_custom_repository: yes
php_custom_repositories_key_url: https://www.dotdeb.org/dotdeb.gpg
php_custom_repositories:
  - deb http://packages.dotdeb.org jessie all
  - deb-src http://packages.dotdeb.org jessie all
# Use a ppa REPO
php_custom_repositories:
  - ppa:ondrej/php

# Or Choose a version and set php_version variables
# Choose packages to install
php_version: 7.2
php_packages:
  - php{{php_version}}-fpm
  - php{{php_version}}-cli
  - php{{php_version}}-common
  - php{{php_version}}-dev
  - php{{php_version}}-opcache
  - php{{php_version}}-mbstring
  - php{{php_version}}-memcached
  - php{{php_version}}-mysql
  - php{{php_version}}-redis
  - php{{php_version}}-curl
  - php{{php_version}}-json
  - php{{php_version}}-xsl
  - php{{php_version}}-xml
  - php{{php_version}}-mongodb
  - php{{php_version}}-imagick

# Define your own pools
php_pools:
  www:
    user: "www-data"
    group: "www-data"
    listen: "/var/run/php-fpm.sock"
    listen.owner: "www-data"
    listen.group: "www-data"
    listen.allowed_clients: "127.0.0.1"
    pm: "dynamic"
    pm.max_children: "5"
    pm.start_servers: "2"
    pm.min_spare_servers: "1"
    pm.max_spare_servers: "3"
    pm.process_idle_timeout: "30s"
    pm.max_requests: "500"

# Custom php.ini configs
php_ini_custom:
  opcache:
    opcache.blacklist_filename: "{{ php_fpm_root_dir }}/fpm/opcache.blacklist"
  newrelic:
    newrelic.enabled: "true"
    newrelic.license: "REPLACE IT WITH YOUR KEY"
    newrelic.logfile: "/var/log/newrelic/php_agent.log"
    newrelic.appname: "{{ ansible_hostname|default('VM Test') }}"
    newrelic.daemon.logfile: "/var/log/newrelic/newrelic-daemon.log"
    newrelic.daemon.port: "/tmp/.newrelic.sock"
    newrelic.daemon.location: "/usr/bin/newrelic-daemon"
    newrelic.daemon.collector_host: "collector.newrelic.com"
    newrelic.analytics_events.enabled: "true"

# Opcache blacklist
php_opcache_blacklist: []

# Php FPM basic configuration
php_fpm_root_dir: /etc/php/7.0
php_fpm_daemon: php7.0-fpm
php_fpm_pid: /run/php-fpm.pid
php_fpm_error_log: /var/log/php-fpm.log
php_fpm_syslog_facility: daemon
php_fpm_syslog_ident: php-fpm
php_fpm_log_level: notice
php_fpm_emergency_restart_threshold: 0
php_fpm_emergency_restart_interval: 0
php_fpm_process_control_timeout: 0
php_fpm_process_max: 128
php_fpm_process_priority: -19
php_fpm_daemonize: yes
php_fpm_rlimit_files: 1024
php_fpm_rlimit_core: 0
php_fpm_events_mechanism: epoll
php_fpm_systemd_interval: 10
php_fpm_pools_include: "{{ php_fpm_root_dir }}/fpm/pool.d/*.conf"

# Enable extras
php_blackfire_enabled: true
php_composer_enabled: true
php_newrelic_enabled: true

# BLackfire.io configuration
php_blackfire_daemon: blackfire-agent
php_blackfire_test_mode: false
php_blackfire_key_url: https://packagecloud.io/gpg.key
php_blackfire_package_url: http://packages.blackfire.io/debian
php_blackfire_directory: /etc/blackfire
php_blackfire_packages:
    - blackfire-agent
    - blackfire-php

# Blackfire agent configuration
php_blackfire_cert: # Sets the PEM encoded certicates
php_blackfire_collector: https://blackfire.io                # Sets the URL of Blackfire's data collector
php_blackfire_http_proxy: # Sets the http proxy to use
php_blackfire_https_proxy: # Sets the https proxy to use
php_blackfire_log_file: stderr                               # Sets the path of the log file. Use stderr to log to stderr
php_blackfire_log_level: 1                                   # log verbosity level (4: debug, 3: info, 2: warning, 1: error)
php_blackfire_server_id: "REPLACE_IT_WITH_YOUR_KEY"          # Sets the server id used to authenticate with Blackfire API
php_blackfire_server_token: "REPLACE_IT_WITH_YOUR_KEY"       # Sets the server token used to authenticate with Blackfire API. It is unsafe to set this from the command line
php_blackfire_socket: "unix:///var/run/blackfire/agent.sock" # Sets the socket the agent should read traces from. Possible value can be a unix socket or a TCP address
php_blackfire_spec: # Sets the path to the json specifications file

# Composer configuration
php_composer_download_url: https://getcomposer.org/installer
php_composer_path: /usr/local/bin/composer
php_composer_version: ''

# Newrelic APM configuration
php_newrelic_apt_key_url: https://download.newrelic.com/548C16BF.gpg
php_newrelic_apt_url: http://apt.newrelic.com/debian/
php_newrelic_packages:
  - newrelic-php5

# Configure php session on memcache wit MCrouter
# default set to false
php_memcache_mcrouter_enabled: true

```

Example Playbook
----------------

```yaml
- hosts: servers
  roles:
     - { role: jebovic.php }
```

Example : config
----------------

```yaml
# Define your own pools
php_pools:
  my_pool:
    user: "me"
    group: "me"
    listen: "/var/run/php-fpm.sock"
    listen.owner: "www-data"
    listen.group: "www-data"
    listen.allowed_clients: "127.0.0.1"
    pm: "dynamic"
    pm.max_children: "10"
    pm.start_servers: "4"
    pm.min_spare_servers: "2"
    pm.max_spare_servers: "6"
    pm.process_idle_timeout: "300s"
    pm.max_requests: "300"
# Custom php config, link it with my other roles, mailhog (mail catcher), memcached, newrelic, blackfire...
php_ini_custom:
  Session:
    session.save_handler: memcached
    session.save_path: "{{ memcache_listen_address }}:{{ memcache_port }}"
  mail function:
    sendmail_from: me@example.com
    sendmail_path: "{{ mailhog_mhsendmail_path }}"
  opcache:
    opcache.enable_cli: Off
    opcache.blacklist_filename: /etc/php5/fpm/opcache.blacklist
  newrelic:
    newrelic.enabled: "true"
    newrelic.license: "{{ newrelic_license }}"
    newrelic.logfile: "/var/log/newrelic/php_agent.log"
    newrelic.appname: "{{ ansible_hostname|default('VM Test') }}"
    newrelic.daemon.logfile: "/var/log/newrelic/newrelic-daemon.log"
    newrelic.daemon.port: "/tmp/.newrelic.sock"
    newrelic.daemon.location: "/usr/bin/newrelic-daemon"
    newrelic.daemon.collector_host: "collector.newrelic.com"
    newrelic.analytics_events.enabled: "true"
```
Example2 : use ubuntu PPA and Memcache MCroute
--------------------
```yaml
php_blackfire_enabled: false
php_composer_enabled: false
php_newrelic_enabled: false
php_memcache_mcrouter_enabled: true
php_use_custom_repository: yes
php_version: 7.2
php_custom_repositories:
  - ppa:ondrej/php
php_packages:
  - php{{php_version}}-fpm
  - php{{php_version}}-cli
  - php{{php_version}}-common
  - php{{php_version}}-json
  - php{{php_version}}-mbstring
  - php{{php_version}}-mysql
  - php{{php_version}}-opcache
  - php{{php_version}}-readline
  - php{{php_version}}-xml
php_pools:
  www:
    user: "www-data"
    group: "www-data"
    listen: "/run/php/php{{php_version}}-fpm.sock"
    listen.owner: "www-data"
    listen.group: "www-data"
    listen.allowed_clients: "127.0.0.1"
    pm: "dynamic"
    pm.max_children: "10"
    pm.start_servers: "4"
    pm.min_spare_servers: "2"
    pm.max_spare_servers: "6"
    pm.process_idle_timeout: "300s"
    pm.max_requests: "300"
php_ini_custom:
  Session:
    session.save_handler: memcached
    session.save_path: "{{ memcache_listen_address }}:{{ memcache_port }}"
```

Tags
----

* php_blackfire_config : only configure blackfire and restart php and blackfire services
* php_config : only update config and restart service
* php_composer : only install coposer
* php_blackfire : only install, configure and start blackfire service
* php_newrelic : only add newrelic apm plugin for php

License
-------

MIT

Author Information
------------------

Jérémy Baumgarth https://github.com/jebovic
