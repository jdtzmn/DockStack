
# DockStack – DevStack on Docker

> This project installs DevStack inside a Docker container.

[![Build Status](https://travis-ci.org/jdtzmn/DockStack.svg?branch=master)](https://travis-ci.org/jdtzmn/DockStack)

## **NOTICE**

This version of DockStack is based on the [original](https://github.com/janmattfeld/DockStack) by janmattfeld.

It updates the devstack to the stein version as well as removing some excessive features.

## Why

A DevStack setup should

1. Not alter the host system
2. Restart clean and fast
3. Allow snapshots
4. Be lightweight
5. Run guest applications fast

### Not invented here

Running Docker on DevStack actually has been done before [[1]]. We add the following:

1. Ubuntu 16.04 LTS base image
2. systemd [[2]]
3. OpenStack Ocata and Pike
4. libvirt/QEMU instance support
5. Zun instead of the deprecated Nova Docker
6. container-adjusted DevStack configuration
7. Network configuration

[1]: https://github.com/ewindisch/dockenstack
[2]: https://docs.openstack.org/devstack/latest/systemd.html

## Quickstart

The `Makefile` includes a complete Docker lifecycle. Image build and DevStack installation are simply started with

```console
$ git clone https://github.com/jdtzmn/DockStack.git
$ cd DockStack
$ make

This is your host IP address: 172.17.0.2
Horizon is now available at http://172.17.0.2/dashboard
Keystone is serving at http://172.17.0.2/identity/
The default users are: admin and demo
The password: secret

Services are running under systemd unit files.

DevStack Version: stein
OS Version: Ubuntu 18.04 xenial
```

The first run can take up to 50 minutes, downloading all Ubuntu and Python packages. Subsequent container starts are much faster because of the Docker cache.

### Usage

- Enter the main DevStack container directly with `make bash`.
- Check your installation via Horizon at the displayed address.

### Network

Internet access for OpenStack instances

- Edit the `public-subnet` and enable DHCP with a custom DNS server i. e. `8.8.8.8`.

Reaching an OpenStack instance from your host through Docker

1. Add custom rules to the default security group

```text
Ping: Ingress, IPv4, ICMP, Any, 0.0.0.0/0
```

2. On your host: Route to instances through docker instead of the (here unusable) Open vSwitch/Neutron interface br-ex

```console
$ ip route
172.24.4.0/24 dev br-ex proto kernel scope link src 172.24.4.1

$ sudo ip route del 172.24.4.0/24

$ sudo ip route add 172.24.4.0/24 via 172.17.0.2

$ ip route
172.24.4.0/24 via 172.17.0.2 dev docker0
```

### Configuration

Feel free to adjust the file `local.conf` for your needs [[3]].

[3]: https://docs.openstack.org/devstack/latest/configuration.html#local-conf

### Snapshots

Although a container restart is faster than a complete build, it still takes a few minutes. So for experimenting use

- `docker commit` to save your running DevStack into the image
- Docker checkpoints [[4]] (experimental)
- the classic workflow of `/devstack/unstack.sh` and `/devstack/stack.sh`

If you really messed it up, `make clean` followed by `make run` will set up a fresh DevStack.

[4]: https://criu.org/Docker

### Requirements

- Recent Linux (tested on Ubuntu 18.04 LTS)
- 4 GB of RAM available for the container
- Docker (tested on 17.06.0-ce)
