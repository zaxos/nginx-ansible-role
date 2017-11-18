[![Build Status](https://travis-ci.org/zaxos/nginx-ansible-role.svg?branch=master)](https://travis-ci.org/zaxos/nginx-ansible-role)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-_zaxos.nginx--ansible--role-blue.svg)](https://galaxy.ansible.com/zaxos/nginx-ansible-role/)

nginx-ansible-role
==================

Ansible role to install and configure Nginx on CentOS/RHEL.

Requirements
------------
* CentOS/RHEL 7
* SELinux disabled

Installation
------------
```
$ ansible-galaxy install zaxos.nginx-ansible-role
```

Example Playbook (simple)
-------------------------
```yaml
- hosts: servers
  vars:
    nginx_vhosts:
      - listen: "80"
        server_name: "www.example.com example.com"
        locations:
          - location: "/"
            proxy_pass: "http://127.0.0.1:8080"

  roles:
    - role: zaxos.nginx-ansible-role
```

Example Playbook (thorough)
-------------------------
```yaml
- hosts: servers
  vars:
    nginx_client_max_body_size: "64m"
    nginx_worker_connections: "2048"
    nginx_multi_accept: "on"

    nginx_conf_events_extra_parameters:
      use: "epoll"
    
    nginx_conf_http_extra_parameters:
      client_body_buffer_size: "10k"
      client_header_buffer_size: "1k"

    nginx_conf_http_extra_parameters_raw: |
      large_client_header_buffers 2 1k;

    nginx_conf_extra_parameters:
      worker_rlimit_nofile: "40000"

    nginx_gzip: "on"
    nginx_conf_gzip_extra_parameters:
      gzip_comp_level: 5
      gzip_disable: "msie6"
      gzip_vary: "on"
      gzip_proxied: "any"

    nginx_upstreams:
      - name: "backend"
        lb_method: "ip_hash"
        servers:
          - "10.10.10.10:80"
          - "10.10.10.11:80"

    nginx_vhosts:
      - listen: "443 ssl"
        server_name: "www.example.com"
        server_name_redirect: "example.com"
        server_name_redirect_https: "www.example.com example.com"
        filename: "www.example.com.conf"
        access_log: "/var/log/nginx/www.example.com.log"
        error_log: "/var/log/nginx/www.example.com.error.log"
        ssl:
          ssl_certificate: "/etc/nginx/cert.crt"
          ssl_certificate_key: "/etc/nginx/cert.key"
          ssl_protocols: "TLSv1 TLSv1.1 TLSv1.2"
          ssl_prefer_server_ciphers: "on"
          ssl_dhparam: "/etc/nginx/dhparam.pem"
          resolver: "8.8.8.8 8.8.4.4"
          ssl_stapling: "on"
          ssl_stapling_verify: "on"          
          raw: |
            ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
            add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";
        extra_parameters:
          proxy_redirect: "off"
          proxy_buffer_size: "4k"
          proxy_buffers: "4 32k"
          proxy_busy_buffers_size: "64k"
          proxy_temp_file_write_size: "64k"
        locations:
          - location: "/"
            proxy_pass: "http://backend"
            proxy_connect_timeout: "600"
            proxy_send_timeout: "600"
            proxy_read_timeout: "600"
            send_timeout: "600"
            raw: |
              proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
              proxy_set_header Host $http_host;
              proxy_set_header X-Real-IP $remote_addr;
              proxy_set_header X-Forwarded-Ssl on;
              proxy_set_header X-Forwarded-Proto https;
          - location: "/api"
            proxy_pass: "http://10.10.10.12:8080"
            rewrites:
              ^/api(.*): "/$1 break"
            raw: |
              set $new_request_uri "";
              set $subdomain "";
              
  roles:
    - role: zaxos.nginx-ansible-role
```
