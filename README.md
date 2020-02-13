# Ansible Role for deploy Consul

![Consul](https://www.consul.io/assets/images/mega-nav/logo-consul-c77f6844.svg) <h1>Consul</h1>
[![Build Status](https://api.travis-ci.org/automium/ansible-consul.svg?branch=master)](https://travis-ci.org/entercloudsuite/ansible-consul)
[![Galaxy](https://img.shields.io/badge/galaxy-automium.consul-blue.svg?style=flat-square)](https://galaxy.ansible.com/automium/consul)

## Features
### Configure Consul with YAML
Configuration for the Consul service is done using YAML to JSON conversion, so you can express your Consul configuration(s) like so:  
```yaml
consul_master_token: myToken
consul_server: true
consul_configs:
  main:
    acl_datacenter: pantheon
    acl_master_token: "{{ consul_master_token | to_uuid }}"
    bootstrap: true
    bind_addr: 0.0.0.0
    client_addr: 0.0.0.0
    datacenter: pantheon
    data_dir: "{{ consul_data_dir }}"
    log_level: INFO
    node_name: master
    server: "{{ consul_server }}"
    ui: true
```
This is done using Jinja2 filters. This role implements no preconfigured entries for configuration, so rather than write your Consul's configuration in JSON; you simply express it in YAML's equivilent syntax, which means it can sit anywhere in your Ansible configuration.  
As seen in the above example, this can be quite powerful, as it allows you to leverage other filters available in Ansible to construct arbitrary data and use it in the configuration. Anything you can express with Ansible's templating (including fetching data/host information from the inventory, etc.) can be used in the configuration.  

The above example configuration shows simple string key/value pairs, but you can of course define any valid type in YAML, such as dictionaries, and lists.  
If you don't know how to express your Consul's JSON configuration as YAML, see [here for a handy converter](https://www.json2yaml.com/).  
  

## Requirements
This role has only been tested on Ubuntu 16.04, but should be expected to work on any Linux distro.
It requred
  - `systemd`  
  - `unzip`  

## Default Role Variables

```yaml
---
consul_packer_provision: false
consul_group_name: consul
consul_group_gid: 3000
consul_user_name: consul
consul_user_uid: 3000
consul_user_home: /opt/consul
consul_config_dir: "{{ consul_user_home }}/conf.d"
consul_data_dir: "{{ consul_user_home }}/data"
consul_version: 1.6.3
# if consul is configure to be a server systemd set cap_net_bind_service for bind port 53
consul_cap_net_bind_service: "{{ consul_configs.main.server | default('false') }}"
consul_server: false
consul_uri: "https://releases.hashicorp.com/consul/{{ consul_version }}/consul_{{ consul_version }}_linux_amd64.zip"
consul_config_src: main.json.j2
consul_config_validate: "{{ consul_user_home }}/bin/consul validate -config-format=json %s"
consul_extra_args: []
consul_service_file:
  src: consul.service.j2
  dest: /etc/systemd/system/consul.service
consul_service_status: started
# reloaded
# restarted
# started
# stopped

enable_on_boot: yes
# yes
# no

# configure this var for don't run configuration step
# no_configure

# configure this var for don't run configuration step
# no_install

consul_config:
  datacenter: dc-1
  data_dir: "{{ consul_data_dir }}"
  log_level: INFO
  node_name: node-1
  server: "{{ consul_server }}"
```

## Example Playbook

#### Basic Role configuration
```yaml
- hosts: consul_servers
  vars:
    consul_master_token: myToken
    consul_server: true
    consul_config:
      acl_datacenter: pantheon
      acl_master_token: "{{ consul_master_token | to_uuid }}"
      bootstrap: true
      bind_addr: 0.0.0.0
      client_addr: 0.0.0.0
      datacenter: pantheon
      data_dir: "{{ consul_data_dir }}"
      log_level: INFO
      node_name: master
      server: "{{ consul_server }}"
      ui: true
  roles:
      - entercloudsuite.consul
```
#### Don't configure just install
```yaml
---
- name: run the main rolet
  hosts: all
  roles:
    - role: entercloudsuite.consul
      configure: false
      install: true
      consul_service_status: "stopped"
      consul_master_token: myToken
      consul_server: true
      consul_configs:
        main:
          acl_datacenter: pantheon
          acl_master_token: "{{ consul_master_token | to_uuid }}"
          bootstrap: true
          bind_addr: 0.0.0.0
          client_addr: 0.0.0.0
          datacenter: pantheon
          data_dir: "{{ consul_data_dir }}"
          log_level: INFO
          node_name: master
          server: "{{ consul_server }}"
          ui: true
```
#### Don't install just configure
```yaml
---
- name: run the main rolet
  hosts: all
  roles:
    - role: entercloudsuite.consul
      configure: true
      install: false
      consul_service_status: "started"
      consul_master_token: myToken
      consul_server: true
      consul_configs:
        main:
          acl_datacenter: pantheon
          acl_master_token: "{{ consul_master_token | to_uuid }}"
          bootstrap: true
          bind_addr: 0.0.0.0
          client_addr: 0.0.0.0
          datacenter: pantheon
          data_dir: "{{ consul_data_dir }}"
          log_level: INFO
          node_name: master
          server: "{{ consul_server }}"
          ui: true
```

# Consul Agent configurations
Agent example configurations, it join to server in groups['server']

```yaml
    - role: ansible-consul
      configure: true
      install: true
      consul_service_status: "started"
      consul_version: 1.6.3
      consul_configs:
        main:
          bind_addr: "{{ ansible_default_ipv4['address'] }}"
          client_addr: 0.0.0.0
          node_name: "{{ ansible_hostname }}"
          data_dir: "{{ consul_data_dir }}"
          datacenter: "pantheon"
          enable_syslog: true
          server: false
          ui: true
          enable_script_checks: true
          rejoin_after_leave: true
          retry_join: "{{ groups['server'] | map('extract', hostvars, ['ansible_default_ipv4', 'address']) | list }}"
```
# Consul Server example
Server example configurations, host in groups['server'] create a new consul cluster.
```yaml
    - role: ansible-consul
      configure: true
      install: true
      consul_service_status: "started"
      consul_version: 1.6.3
      consul_configs:
        main:
          bind_addr: "{{ ansible_default_ipv4['address'] }}"
          client_addr: 0.0.0.0
          node_name: "{{ ansible_hostname }}"
          data_dir: "{{ consul_data_dir }}"
          datacenter: "pantheon"
          enable_syslog: true
          server: true
          ui: true
          enable_script_checks: true
          rejoin_after_leave: true
          retry_join: "{{ groups['server'] | map('extract', hostvars, ['ansible_default_ipv4', 'address']) | list }}"
          ports:
            dns: 53
          dns_config:
            udp_answer_limit: 64
          bootstrap_expect: "{{ groups['server'] | length | int }}"
          recursors:
            - 1.1.1.1
            - 8.8.8.8
```
## Testing

Tests are performed using [Molecule](http://molecule.readthedocs.org/en/latest/).

Install Molecule or use `docker-compose run --rm molecule` to run a local Docker container, based on the [enterclousuite/molecule](https://hub.docker.com/r/fminzoni/molecule/) project, from where you can use `molecule`.

1. Run `molecule create` to start the target Docker container on your local engine.
2. Use `molecule login` to log in to the running container.
3. Edit the role files.
4. Add other required roles (external) in the molecule/default/requirements.yml file.
5. Edit the molecule/default/playbook.yml.
6. Define infra tests under the molecule/default/tests folder using the goos verifier.
7. When ready, use `molecule converge` to run the Ansible Playbook and `molecule verify` to execute the test suite.
Note that the converge process starts performing a syntax check of the role.
Destroy the Docker container with the command `molecule destroy`.

To run all the steps with just one command, run `molecule test`.

In order to run the role targeting a VM, use the playbook_deploy.yml file for example with the following command: `ansible-playbook ansible-docker/molecule/default/playbook_deploy.yml -i VM_IP_OR_FQDN, -u ubuntu --private-key private.pem`


## License
MIT

## Author Information
Created by:
 - Calum MacRae
 - Jacopo Secchiero
 - Attilio Greco
