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
  - **Monitoring**:
      - _default_: discuss in monitoring parts
      - _optional_: discuss in monitoring parts

---
### Defaults
### Required variables
### Optional variables
### Tags
### Notes
