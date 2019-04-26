# Ansible Nginx

## Description
The task is to write an Ansible role to install and manage Nginx, with the latest stable version on Ubuntu 18.04

## Prerequisites
* An EC2 instance with a static IP mapped to a hostname
* Security group for this EC2 with opened port `tcp:80`

## Required variables

Each vhosts is passed as an item in the array `vhosts` (see [test](tests/test.yml) for examples). Each item must have a `type` that is 1 in 3 types:

* `type: "static-site"`: Serving a static site, with all requests routed to the index file. Useful for ReactJS frontend.
* `type: "reverse-proxy"`: Will do a proxy-pass to another IP:Port. Useful to act as a reverse proxy for Node.JS.
* `type: "php-fpm"`: Act as FPM-FPM Gateway for a PHP-FPM backend.

The following options are common for each types:

* `access_log`, `error_log`: Log paths for the vhosts
* `root`: The webroot for the vhost. Does not do anything for type `reverse-proxy`
* `index`: Index file. Does not do anything for type `reverse-proxy`
* `cors`: Whether to enable CORS for *all* incoming requests. Note: This should be used in non-production only. Cross-Origin *should* be handled at the application level

### extra_config

Each item can take an `extra_config` variable, which is a list of extra files that will be automatically appended to the end of the `server { ... }``` block of that vhost.

### Examples
```yaml
vhosts:
  # Vhost - static site
  - server_name: "nfq.asia"
    type: "static-site"
    root: "/var/www/static-frontend/current"
    index: "/index.html"
    access_log: "/var/log/nginx/static-frontend.access.log"
    error_log: "/var/log/nginx/static-frontend.error.log"
    cors: true

  # Vhost with an extra config files
  - server_name: "staging.nfq.asia"
    type: "static-site"
    root: "/var/www/static-a-frontend/current"
    index: "/index.html"
    access_log: "/var/log/nginx/static-frontend-2.access.log"
    error_log: "/var/log/nginx/static-frontend-2.error.log"
    extra_config:
      - "files/service-worker.j2"

  # Vhost - reverse proxy to port 9000
  - server_name: "api.nfq.asia"
    type: "reverse-proxy"
    access_log: "/var/log/nginx/nodejs-backend.access.log"
    error_log: "/var/log/nginx/nodejs-backend.error.log"
    cors_enabled: true
    reverse_proxy_pass: "http://127.0.0.1:9000"
    nocache: true

  # 3. Vhost - php-fpm to port tcp:9000
  - server_name: "backend.nfq.asia"
    type: "php-fpm"
    root: "/var/www/backend-symfony/current/web"
    index: "app.php"
    access_log: "/var/log/nginx/php-fpm-backend.access.log"
    error_log: "/var/log/nginx/php-fpm-backend.error.log"
    cors: true
    php_fpm_pass: "127.0.0.1:9000"
    php_fpm_status: true
```

## Defaults

* List of default parameters, mostly for tuning:

```yaml
nginx_user: "www-data"
nginx_worker_processes: "auto"
nginx_worker_cpu_affinity: "auto"
nginx_worker_rlimit_nofile: "100000"
nginx_worker_connections: "4096"

nginx_keepalive_timeout: "100"
nginx_client_body_timeout: "120"
nginx_reset_timedout_connection: "on"
nginx_types_hash_max_size: "2048"
nginx_client_max_body_size: "50M"
```

* `nginx_server_tokens: "off"`: Hide nginx version number
* `nginx_ssl_protocols: "TLSv1 TLSv1.1 TLSv1.2"`: Dropping SSLv3, ref: POODLE
* `nginx_apm_log_enabled: true`: Enable apm log format by default

## Optional variables
* `backup_old_config: true`: Automatically backing up old configurations before applying new ones at `/etc/nginx/.backup/` directory

## Tags
* `install` - install Nginx service.
* `configure` - config Nginx service.

## Notes

* The role does not handle TLS and HTTP->HTTPS Redirection by default since these are assumed to be handled by ALB.
* There is some tuning on nginx done (worker_processes, worker_cpu_affinity, etc.) as per the configuration [here](templates/nginx.conf.j2)
* The access log are in [APM format](https://www.nginx.com/blog/using-nginx-logging-for-application-performance-monitoring/)
* Nginx status can be checked with the endpoint `/nginx_status`:

```
curl 127.0.0.1:80/nginx_status
```
