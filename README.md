
## Ansible Nginx

### A. Purpose

The task is to write an Ansible role to install and manage Nginx stable version (1.14.2)

### B. Specifications

1. Input
    - Provide each nginx config file in this repo
    - Apply one by one for each config files into directory `/etc/nginx/conf.d/`
    - Only listen on `tcp:80` as default, not `tcp:443`
    - No SSL/HTTPs cert because we should manage it on AWS ALB (with ACM)

2. Prerequisites
    - An EC2 instance with a static IP mapped to a hostname
    - Security group for this EC2 with opened port `tcp:80`

3. Output
  - **Connectivity**:
      - _external_:
          - receive request from port `tcp:443` on ALB Listeners. 301 Redirection is assumed to be handled in ALB.
          - then routing from ALB to `tcp:80` of EC2 vm, nginx handling
      - _internal_: within AWS private subnet:
          - port: `tcp:80` as nginx
          - proxy pass from nginx to other services inside EC2 vm (backend api, admin, ...)
  - **Configuration**:
      - _default_:
          - all default configurations as the original of nginx
          - apply auto tuning for nginx.conf (worker_processes, worker_cpu_affinity, ...)
          - apply APM log format
      - _advanced_:
          - Flexible configure by editing config files, default variables on this repo
          - Automatically backing up old configurations before applying new ones at `/etc/nginx/.backup/` directory (hidden)
          - Support checking nginx status by using `/nginx_status`
          ```
          curl 127.0.0.1:80/nginx_status
          ```
          - Support checking other status/ping like php-fpm by using `/php_fpm_ping` & `/php_fpm_status`
          ```
          curl 127.0.0.1:80/php_fpm_status
          ```
  - **Monitoring**:
      - _default_: discuss in monitoring parts
      - _optional_: discuss in monitoring parts

---
### Defaults

### Required variables
* `nginx_user` - user of nginx service, default is `www-data`.

### Optional variables
* `static_site_enabled` - used to config vhost file for static site (ie: react frontend), default is `true`.
    - List variables required for vhost statis site:

    ```yaml
    static_site_server_name:
    static_site_root:
    static_site_index:
    static_site_access_log:
    static_site_error_log:
    static_site_cors_enabled: true

    ```
* `reverse_proxy_enabled` -  used to config vhost file for reverse proxy to port tcp:xxx (ie: nodejs backend), default is `true`.
    - List variables required for vhost reverse proxy:

    ```yaml
    reverse_proxy_server_name:
    reverse_proxy_pass:
    reverse_proxy_access_log:
    reverse_proxy_error_log:
    reverse_proxy_cors_enabled: true
    reverse_proxy_nocache_enabled: true

    ```

* `php_fpm_enabled` - used to config vhost file php-fpm to port tcp:9000, default is `true`.
	- List variables required for vhost php-fpm:

    ```yaml
    php_fpm_server_name:
    php_fpm_root:
    php_fpm_pass:
    php_fpm_index:
    php_fpm_access_log:
    php_fpm_error_log:
    php_fpm_status_enabled: true

    ```
### Tags
* `install` - install Nginx service.
* `configure` - config Nginx service.
### Notes

* Ansible-role-nginx just support creating only one vhost file for each types (static file, reverse proxy and php-fpm).
