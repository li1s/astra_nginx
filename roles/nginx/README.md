# Ansible role `astra.nginx.nginx`

Role for NGINX server deployment and managment.

## Requirements

All requirements for this role are the same as specified at the [collection level](../../README.md).

## Role variables

| Variable                              | Comment                                                                            | Required | Example/Default                    |
|:--------------------------------------|------------------------------------------------------------------------------------|:--------:|------------------------------------|
| `nginx_package_version`               | Required version of the `nginx` package to install                                 | no       | See example in `defaults/main.yml` |
| `nginx_vhosts`                        | List of vhost definitions (server blocks) for NGINX virtual hosts                  | no       | See example in `defaults/main.yml` |
| `nginx_remove_default_vhost`          | Whether to remove the 'default' virtual host configuration supplied by NGINX       | no       | `false`                            |
| `use_ssl`                             | Enables SSL                                                                        | no       | `false`                            |
| `nginx_ssl`                           | SSL configuration to be set in the `ssl.conf` file                                 | no       | See example in `defaults/main.yml` |
| `nginx_upstreams`                     | Defines one or more upstream sets                                                  | no       | See example in `defaults/main.yml` |
| `nginx_worker_processes`              | The number of CPU cores allocated to the worker processes                          | no       | `{{ ansible_processor_vcpus \|default(ansible_processor_count) }}` |
| `nginx_worker_connections`            | The number of connections per process                                              | no       | `1024`                             |
| `nginx_multi_accept`                  | Defines if you want NGINX to accept all connections immediately                    | no       | `off`                              |
| `nginx_error_log`                     | The path to the NGINX error log file and log level                                 | no       | `/var/log/nginx/error.log warn`    |
| `nginx_access_log`                    | The path to the NGINX access log file and log options                              | no       | `/var/log/nginx/access.log main buffer=16k` |
| `nginx_sendfile`                      | [See nginx_sendfile](https://nginx.org/en/docs/http/ngx_http_core_module.html#sendfile) | no       | `on`                          |
| `nginx_tcp_nopush`                    | [See nginx_tcp_nopush](https://nginx.org/en/docs/http/ngx_http_core_module.html#tcp_nopush) | no       | `on`                      |
| `nginx_tcp_nodelay`                   | [See nginx_tcp_nodelay](https://nginx.org/en/docs/http/ngx_http_core_module.html#tcp_nodelay) | no       | `on`                    |
| `nginx_keepalive_timeout`             | NGINX keepalive settings                                                           | no       | `65`                               |
| `nginx_keepalive_requests`            | NGINX keepalive settings                                                           | no       | `100`                              |
| `nginx_server_tokens`                 | Controls whether NGINX responds with its version in HTTP headers                   | no       | `on`                               |
| `nginx_client_max_body_size`          | Determines the largest possible file upload through NGINX before hitting a backend | no       | `64m`                              |
| `nginx_server_names_hash_bucket_size` | Used if you have many server names, or have very long server names                 | no       | `64`                               |
| `nginx_proxy_cache_path`              | Sets the path to the NGINX cache                                                   | no       | `/var/cache/nginx`                 |
| `nginx_extra_http_options`            | Extra HTTP options                                                                 | no       | See example in `defaults/main.yml` |
| `nginx_extra_conf_options`            | Extra NGINX configuration options                                                  | no       | See example in `defaults/main.yml` |
| `nginx_log_format`                    | Configures NGINX's log_format                                                      | no       | See example in `defaults/main.yml` |
| `nginx_charset`                       | Adds the specified charset to the “Content-Type” response header field             | no       | `utf-8` |
| `nginx_snippets`                      | Adds NGINX snippets                                                                | no       | See example in `defaults/main.yml` |
| `nginx_custom_srv_conf`               | Adds custom options for `nginx.service`                                            | no       | See example in `defaults/main.yml` |

## Examples

```YAML
---
    - hosts: servers
      roles:
        - nginx
      vars:
        nginx_package_version: 1.22.1-9 # You can specify version of the nginx directly
```

Configure vhost "example.com" with SSL enabled and create redirect from 80 port to 443:

```YAML
nginx_remove_default_vhost: true # delete default vhost
use_ssl: true #  enable ssl
nginx_vhosts:
  - server_name: "example.com"
    listen: "443 ssl http2" # default: "80"
    root: "/var/www/html"
    index: "index.php index.html index.htm"
    state: present # To remove the vhost configuration, type 'absent'
    ssl_certificate: "/etc/ssl/certs/ssl-cert-snakeoil.pem"
    ssl_certificate_key: "/etc/ssl/private/ssl-cert-snakeoil.key"
    ssl_protocols: "TLSv1.1 TLSv1.2"
    ssl_ciphers: "HIGH:!aNULL:!MD5"
```

Vhost with PHP support, with additional SSL configuration:

```YAML
nginx_remove_default_vhost: true
use_ssl: true
nginx_vhosts:
  - listen: "443 ssl http2"
    server_name: "example.com"
    root: "/var/www/example.com"
    index: "index.php index.html index.htm"
    error_page: ""
    access_log: ""
    error_log: ""
    state: "present"
    extra_parameters: |
      location ~ \.php$ {
          fastcgi_split_path_info ^(.+\.php)(/.+)$;
          fastcgi_pass unix:/var/run/php5-fpm.sock;
          fastcgi_index index.php;
          fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
          include fastcgi_params;
      }
    ssl_certificate: "/etc/ssl/certs/ssl-cert-snakeoil.pem"
    ssl_certificate_key: "/etc/ssl/private/ssl-cert-snakeoil.key"
    ssl_protocols: "TLSv1.1 TLSv1.2"
    ssl_ciphers: "HIGH:!aNULL:!MD5"
    ssl_session_cache: "shared:SSL:50m"
    ssl_session_timeout: "5m"
    ssl_session_tickets: "off"
    ssl_prefer_server_ciphers: "on"
    resolver: "8.8.8.8 8.8.4.4 valid=10m"
    resolver_timeout: "10s"
    http2_max_field_size: "16k"
    http2_max_header_size: "32k"
```

Create two vhosts without SSL:

```YAML
nginx_vhosts:
  - listen: "80 default_server"
    server_name: "site1.example.com"
    root: "/var/www/site1.example.com"
    index: "index.php index.html index.htm"
  - server_name: "site2.example.com"
    root: "/var/www/site2.example.com"
    index: "index.php index.html index.htm"
```

Install `fancyindex` module from repository:

```YAML
nginx_install_modules: true
nginx_modules:
  - http-fancyindex
```

Important note: if you set `nginx_install_modules` to `true` with installing `libnginx-mod-http-fancyindex` module, and even you provide required `nginx_package_version` (like 1.18.0-2~buster), role will install **latest** NGINX version anyway, as dependency.

Use styled error pages from a default directory:

```YAML
  error_page 403 /403.html;
  location /403.html {
      root /var/www/html;
  }
```

Enable styled underconstruction main page set main location in your config file:

```YAML
  location / {
          root /var/www/html;
          index underconstruction.html;
  }
```

## Author information

LLC "RusBITech-Astra"

## License

MIT / BSD
