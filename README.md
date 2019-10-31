![Consul](https://www.consul.io/assets/images/mega-nav/logo-consul-c77f6844.svg) <h1>Consul</h1>
[![Build Status](https://travis-ci.org/cmacrae/ansible-consul.svg?branch=master)](https://travis-ci.org/cmacrae/ansible-consul)  
A no nonsense Ansible role to deploy and configure [Consul](https://consul.io)

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

### Multiple configuration file deployments
As shown in the above example, the `consul_configs` variable has a `main` dict. As you might have guessed; you can define multiple dicts of configuration. These are then deployed to files named after their parent key in the `conf.d` directory.  

### KISS
Not really sure this should be listed as a feature... but I deem it valuable.  
Consul, for what it does, is fantastically simple to configure and spin up - as such, it deserves an equally simple Ansible role.  
The main tasks file is only just over 70 lines, including whitespace. And the main configuration template? 1 line.  
Keeping this role simple allows it to be very flexible and easy to integrate with anything else you might want to.

### Packer provisioning support
A very simple, but handy feature of this role is the ability to set `consul_packer_provision` to `true` (`false` by default). When this is `true`, during the Ansible run, it won't start the Consul service. This exists so that you can place values into your configuration that may not be valid, intended to be used when producing machine images of some sort with Packer, intended to be substituted later on by way of some sort of launch configuration/user script/user data.  

## Requirements
This role has only been tested on Ubuntu 16.04, but should be expected to work on any Linux distro which runs `systemd` and has an `unzip` package available.

## Default Role Variables

```yaml
consul_group_name: consul
consul_group_gid: 3000
consul_user_name: consul
consul_user_uid: 3000
consul_user_home: /opt/consul
consul_config_dir: "{{ consul_user_home }}/conf.d"
consul_data_dir: "{{ consul_user_home }}/data"
consul_version: 0.8.4
consul_server: false
consul_uri: "https://releases.hashicorp.com/consul/{{ consul_version }}/consul_{{ consul_version }}_linux_amd64.zip"
consul_config_src: main.json.j2
consul_service_file:
  src: consul.service.j2
  dest: /etc/systemd/system/consul.service

consul_config:
  datacenter: dc-1
  data_dir: "{{ consul_data_dir }}"
  log_level: INFO
  node_name: node-1
  server: "{{ consul_server }}"
```

## Example Playbook

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
      - cmacrae.consul
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
Created by Calum MacRae

Feel free to:  
- Contact me: [@calumacrae](https://twitter.com/calumacrae), [calum0macrae@gmail.com](mailto:calum0macrae@gmail.com)  
- [Raise an issue](https://github.com/cmacrae/ansible-consul/issues)  
- [Contribute](https://github.com/cmacrae/ansible-consul/pulls)  
