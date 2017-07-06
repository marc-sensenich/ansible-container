# Supporting Multiple Containers Per Service in Kubernetes and OpenShift

## Table of Contents
- [Reasoning](#Reasoning)
- [Declaring Multiple Containers Per Service](#declaring-multiple-containers-per-service)

## Reasoning

Some services are composed of multiple containers that are coupled and share resources to for a single cohesive service. Common cases of this are sidecar and ambassador containers. Within Ansible Container currently it is a single pod per service, which allows for the generation of the images used within these services separately and deployed to a server. However, the container.yml file is not able to act as a single source-controlled piece of code to define the state of the deployable application. Therefore, I propose allowing multiple containers to be declared per service. 

## Declaring Multiple Containers Per Service

To achieve this the service section will accept a new property, containers, which will allow for the declaration of 1-N containers per service. The addition of this property will allow for the maintenance of existing applications by using an check when building and deploying the service; e.g. `if 'containers' in service`. A declaration may look like this:

```yaml
services:
  web:
    containers:
      - from: centos:7
        entrypoint: [/usr/bin/entrypoint.sh]
        working_dir: /
        user: apache
        command: [/usr/bin/dumb-init, httpd, -DFOREGROUND]
        ports:
          - 8000:8080
          - 4443:8443
        roles:
          - apache-container
        volumes:
          - static-content:/var/www/static
      - from: centos:7
        command: [/usr/bin/dumb-init, file-refresher, --out-dir, /var/www/static]
        entrypoint: [/usr/bin/entrypoint.sh]
        user: refresher
        roles:
          - apache-file-refresher
        volumes:
          - static-content:/var/www/static
volumes:
  static-content:
    docker: {}
    k8s:
      force: false
      state: present
      access_modes:
      - ReadWriteOnce
      requested_storage: 1Gi
      metadata:
        annotations: 'volume.beta.kubernetes.io/mount-options: "discard"'
```
