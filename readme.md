# Readme


This repository contains some several code snippets i've used to get vlan based networking working
with docker containers.

## Pre reqs

I'm using Ubuntu 20 running as virtual machine on vSphere. If you want to do the same, you will
need to make sure to have the following configured:

- A standard or distributed port group configured with vlan trunking.
- The virtual machine configured with a nic on the trunk port group.
- fully deployed docker-ce: you can reference my blog here for deployment instructions:

## Code Snippets

### `netplan.yaml`

This file is a `cloud-config` file that can be used for reference to the `netplan` configuration 
you might use if you were configuring ubuntu 20.

Example of a `netplan.yaml` configuration I've used:

```yaml
network:
  ethernets:
    ens160:
      addresses:
      - 10.1.110.50/24
      gateway4: 10.1.110.254
      nameservers:
        addresses:
        - 10.1.110.11
        search:
        - foggyclouds.net
    ens192: {}
  vlans:
    vlan111:
      id: 111
      link: ens192
  version: 2

```

 > Your `ens` numbers may be different. you will want to run `ip addr` or `ifconfig` and get the
 > interface ids.

---

### `docker-compose.yaml`

I've included a sample `docker-compose` file that has a couple specific sections I will note here:

#### `networks` section:

```yaml
networks:
  default:
  management:
    driver: ipvlan
    driver_opts:
      parent: vlan111
    ipam:
      config:
        - subnet: 10.1.111.0/24
          gateway: 10.1.111.254
```

The important parts here are the naming: `management` which is the network name we will be using.
We are also using the `vlan111` we configured in the `netplan.yaml` as the `parent` in the
`driver_opts` section of the `management` network. The final section we identify is the `ipam`
configuration for the `management` network. In the `ipam: config` seciton we identify the `subnet`
and the `gateway` that the containers should be configured to use.

#### `services` section:

The important parts to pay attention to for the specific containers is the `networks:` section.

``` yaml
    networks:
      management:
        ipv4_address: 10.1.111.10
```

Here we identify the network name, and the `ipv4_address` that should be assigned to the container.
