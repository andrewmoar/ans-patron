#Ansible postgresql-patroni


Install and configure patroni-cluster.

Requirements
------------

CentOS7


Role Variables
--------------
    consul1: "{{hostvars[groups['consul_server'][0]]['ansible_default_ipv4']['address']}}"
    consul2: "{{hostvars[groups['consul_server'][1]]['ansible_default_ipv4']['address']}}"
    consul3: "{{hostvars[groups['consul_server'][2]]['ansible_default_ipv4']['address']}}"
    CONSUL_VERSION: 1.7.2
    haproxy_pg_nodes:
      - { host:  "{{hostvars[groups['consul_patroni'][0]].inventory_hostname}}", ip: "{{hostvars[groups['consul_patroni'][0]]['ansible_default_ipv4']['address']}}" }
      - { host:  "{{hostvars[groups['consul_patroni'][1]].inventory_hostname}}", ip: "{{hostvars[groups['consul_patroni'][1]]['ansible_default_ipv4']['address']}}" }
      - { host:  "{{hostvars[groups['consul_patroni'][2]].inventory_hostname}}", ip: "{{hostvars[groups['consul_patroni'][2]]['ansible_default_ipv4']['address']}}" }
    
    keepalived_interface: ens192
    keepalived_virtual_address: 10.0.0.5/32
    keepalived_router_id: 130
    keepalived_priority: 100
    keepalived_role: MASTER
    keepalived_password: keepaliverpassword

    consul_dir:
      - /etc/consul.d/
      - /var/lib/consul/
    
    consul_system_user: consul
    consul_system_group: consul
    
    patroni_db_cluster: cluster-db
    
    patroni_postgresql_version: 11
    patroni_postgresql_exists: false
    
    patroni_config_dir: /etc/patroni
    patroni_system_user: postgres
    patroni_system_group: postgres
    
    
    patroni_dcs: consul
    
    patroni_replication_username: replicator #fill the replication user here
    patroni_replication_password: password #fill the replication user password here
    patroni_superuser_username: postgres #fill superuser
    patroni_superuser_password: supersecretpostgrespasswd #fill the superuser password
    
    patroni_postgresql_version: 11 #choice postgresql version
    patroni_postgresql_exists: false #choice patroni password
    #fill the new database , owner and his password, with a dict
    databases:
      1:
        name: prod-database-name
        user: prod-user
        owner: prod-user
        password: "{{ db_pass_1 }}"
        state: present
    db_pass_1: 'prod@password'
    
    # patroni config
    patroni_logformat: "%(asctime)s %(levelname)s: %(message)s"
    patroni_loglevel: INFO
    patroni_requests_loglevel: WARNING
    
    patroni_scope: "{{patroni_db_cluster}}"
    patroni_namespace: /service/
    patroni_name: "{{ inventory_hostname }}"
    
    patroni_restapi_listen: 0.0.0.0:8008
    patroni_restapi_connect_address: "{{ ansible_host }}:8008"
    patroni_restapi_certfile: ""
    patroni_restapi_keyfile: ""
    patroni_restapi_username: ""
    patroni_restapi_password: ""
    
    
    patroni_exhibitor_hosts: ""
    patroni_exhibitor_port: ""
    patroni_exhibitor_poll_interval: 300
    
    patroni_bootstrap_dcs_ttl: 30
    patroni_bootstrap_dcs_loop_wait: 10
    patroni_bootstrap_dcs_retry_timeout: 10
    patroni_bootstrap_dcs_maximum_lag_on_failover: 1048576
    patroni_bootstrap_dcs_master_start_timeout: 300
    patroni_bootstrap_dcs_synchronous_mode: false
    patroni_bootstrap_dcs_postgresql_use_pg_rewind: true
    patroni_bootstrap_dcs_postgresql_use_slots: true
    
    # patroni dynamic options, can be changed in runtime
    patroni_bootstrap_dcs_postgresql_parameters:
      - { option: "max_connections",           value: "300" }
      - { option: "max_locks_per_transaction", value: "64" }
      - { option: "max_worker_processes",      value: "8" }
      - { option: "max_prepared_transactions", value: "0" }
      - { option: "wal_level",                 value: "logical" }
      - { option: "archive_mode",              value: "on" }
      - { option: "track_commit_timestamp",    value: "off" }
      - { option: "max_wal_senders",           value: "20" }
      - { option: "max_replication_slots",     value: "20" }
      - { option: "wal_keep_segments",         value: "128" }
      - { option: "archive_command",           value: "'mkdir -p /opt/db/wal_archive && test ! -f /opt/db/wal_archive/%f && cp %p /opt/db/wal_archive/%f'" }
      - { option: "archive_timeout",           value: "60" }
    patroni_bootstrap_dcs_postgresql_recovery_conf:
      - { option: "standby_mode",    value: "on" }
      - { option: "restore_command", value: "'cp /opt/db/wal_archive/%f %p'" }
    
    patroni_bootstrap_method_name: ""
    patroni_bootstrap_method_command: ""
    patroni_bootstrap_method_keep_existing_recovery_conf: false
    patroni_bootstrap_method_recovery_conf: #[]
    
    patroni_bootstrap_initdb:
      - { option: "encoding", value: "UTF8" }
      - { option: "data-checksums" }
      - { option: "locale", value: "en_US.UTF-8" }
    #dynamic pg hba
    patroni_bootstrap_pg_hba: #[]
      - { type: "host", database: "all",         user: "all",                                address: "0.0.0.0/0", method: "md5" }
      - { type: "host", database: "replication", user: "{{ patroni_replication_username }}", address: "0.0.0.0/0", method: "md5" }
    #create users
    patroni_bootstrap_users:
      - { name: "{{ patroni_superuser_username }}",   password: "{{ patroni_superuser_password }}",   options: [] }
      - { name: "{{ patroni_replication_username }}", password: "{{ patroni_replication_password }}", options: ['replication'] }
    
    patroni_postgresql_use_unix_socket: true
    patroni_postgresql_listen: "0.0.0.0:5432"
    patroni_postgresql_connect_address: "{{ ansible_host }}:5432"
    
    patroni_postgresql_authentication:
      - { type: "superuser",   username: "{{ patroni_superuser_username }}",   password: "{{ patroni_superuser_password }}"   }
      - { type: "replication", username: "{{ patroni_replication_username }}", password: "{{ patroni_replication_password }}" }
    
        #TO FIND
    patroni_postgresql_create_replica_method: basebackup
    
    patroni_postgresql_basebackup: []
    #  - { option: "checkpoint", value: "fast" }
    
    patroni_postgresql_recovery_conf: []
    
    
    patroni_postgresql_custom_conf: "" #TODO
    #postgresql parameters
    patroni_postgresql_parameters:
      - { option: "unix_socket_directories",   value: "/var/run/postgresql" }
      - { option: "shared_buffers",            value: "2048" }
      - { option: "wal_log_hints",             value: "on" }
      - { option: "wal_log_hints",             value: "on" }
      - { option: "min_wal_size",             value: "2GB" }
      - { option: "max_wal_size",             value: "4GB" }
      - { option: "track_functions",           value: "pl" }
      - { option: "random_page_cost",            value: "1" }
      - { option: "unix_socket_directories", value: "/var/run/postgresql" }
      - { option: "synchronous_commit",           value: "on" }
      - { option: "synchronous_standby_names",           value: '"*"' }
      - { option: "work_mem",           value: "8MB" }
      - { option: "shared_buffers",           value: "4GB" }
      - { option: "temp_buffers",           value: "128MB" }
      - { option: "effective_cache_size",           value: "2GB" }
      - { option: "autovacuum_vacuum_threshold",           value: "100" }
      - { option: "vacuum_cost_delay",           value: "10" } #question
    
    
        #change pg_hba.conf
    patroni_postgresql_pg_hba:
     - { type: "host",  database: "replication",  user: "{{ patroni_replication_username }}", address: "0.0.0.0/0",        method: "md5" }
     - { type: "host",  database: "app_db",      user: "app_user",                           address: "10.0.0.0/8",        method: "md5" }
    
    
    
    patroni_postgresql_pg_ctl_timeout: 60
    patroni_postgresql_use_pg_rewind: true
    patroni_postgresql_remove_data_directory_on_rewind_failure: false
    
    patroni_watchdog_mode: 'off' # use quotes for 'off' value
    patroni_watchdog_device: /dev/watchdog
    patroni_watchdog_safety_margin: 5
    
    patroni_tags:
      - { name: "nofailover",    value: "false" }
      - { name: "noloadbalance", value: "false" }
      - { name: "clonefrom",     value: "false" }
      - { name: "nosync",        value: "false" }
    
    patroni_postgresql_data_dir: "/var/lib/pgsql/{{ patroni_postgresql_version }}/data"
    patroni_postgresql_config_dir: "/var/lib/pgsql/{{ patroni_postgresql_version }}/data"
    patroni_postgresql_bin_dir: "/usr/pgsql-{{ patroni_postgresql_version }}/bin"
    patroni_postgresql_pgpass: /var/lib/pgsql/.pgpass
    patroni_bin_dir: /usr/local/bin
    
    patroni_postgresql_packages:
      - { name: "postgresql11",         state: "present" }
      - { name: "postgresql11-server",  state: "present" }
      - { name: "postgresql11-contrib", state: "present" }
      - { name: "postgresql11-devel",   state: "present" }
    


Example Playbook
----------------
- `main.yml` to assign roles to your nodes, e.g.:
```Yaml

- hosts: consul
  become: yes
  vars_files:
    - vars/default.yml
    - vars/{{env}}.yml
  serial:
    - 3
    - 100%

  roles:
    - { role: consul_server, when: "inventory_hostname in groups ['consul_server']", tags: ['consul_server'] }

- hosts: consul
  become: yes

  vars_files:
    - vars/default.yml
    - vars/{{env}}.yml
  serial:
    - 3
    - 100%

  roles:
    - { role: patroni_consul, when: "inventory_hostname in groups ['consul_patroni']|default([])", tags: ['patroni_consul'] }
    - { role: consul_exporter, when: "inventory_hostname in groups ['consul_patroni']|default([])", tags: ['consul_exporter'] }
    - { role: create_db, when: "inventory_hostname in groups ['consul_patroni']|default([])", tags: ['create_db'],ansible_python_interpreter: "/usr/bin/python3"  }
    - { role: haproxy, when: "inventory_hostname in groups ['consul_patroni']|default([])", tags: ['haproxy']  }
    - { role: keepalived, when: "inventory_hostname in groups ['consul_patroni']|default([])", tags: ['keepalived']  }

```


inventory example ./example_inventory, 
where variable env is using for vars file 


    [consul:children]
    consul_server
    consul_patroni

    [consul_server]
    consul-server-1 ansible_host=10.1.216.231
    consul-server-2 ansible_host=10.1.216.232
    consul-server-3 ansible_host=10.1.216.233

    [consul_patroni]

    patroni001 ansible_host=10.1.216.231
    patroni002 ansible_host=10.1.216.232
    patroni003 ansible_host=10.1.216.233


    [consul:vars]
    env='example'
    ansible_ssh_common_args='-o StrictHostKeyChecking=no'
    ansible_python_interpreter=/usr/bin/python2.7




vars name  vars/example.yml


Execute example
```ShellSession
  > ansible-playbook main.yml -i example_inventory 
```









