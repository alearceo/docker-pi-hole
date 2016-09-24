A [Docker](https://www.docker.com/what-docker) project to make lightweight x86 and ARM continer with [pi-hole](https://pi-hole.net) functionality.  Why?  Maybe you don't have a Raspberry Pi lying around but you do have a Docker server.

Now with ARM support!  Just install docker on your Rasberry-Pi (the current install is simply `curl -sSL https://get.docker.com | sh`[[1](https://www.raspberrypi.org/blog/docker-comes-to-raspberry-pi/)] and run docker image `diginc/pi-hole:arm tag` (see below for full required command).

[![Build Status](https://travis-ci.org/diginc/docker-pi-hole.svg?branch=master)](https://travis-ci.org/diginc/docker-pi-hole) [![Docker Stars](https://img.shields.io/docker/stars/diginc/pi-hole.svg?maxAge=2592000)](https://hub.docker.com/r/diginc/pi-hole/) [![Docker Pulls](https://img.shields.io/docker/pulls/diginc/pi-hole.svg?maxAge=2592000)](https://hub.docker.com/r/diginc/pi-hole/)

[![Join the chat at https://gitter.im/diginc/docker-pi-hole](https://badges.gitter.im/diginc/docker-pi-hole.svg)](https://gitter.im/diginc/docker-pi-hole?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## Running Pi-Hole Docker

[Dockerhub](https://hub.docker.com/r/diginc/pi-hole/) automatically builds the latest docker-pi-hole changes into images which can easily be pulled and ran with a simple `docker run` command.

One crucial thing to know before starting is the docker-pi-hole container needs port 53 and port 80, 2 very popular ports that may conflict with existing applications.  If you have no other services or dockers using port 53/80 (if you do, keep reading below for a reverse proxy example), the minimum options required to run this container are in the script [docker_run.sh](https://github.com/diginc/docker-pi-hole/blob/master/docker_run.sh) or summarized here:

```
IMAGE='diginc/pi-hole'
NIC='eth0'
IP=$(ip addr show $NIC | grep "inet\b" | awk '{print $2}' | cut -d/ -f1)
docker run -p 53:53/tcp -p 53:53/udp -p 80:80 --cap-add=NET_ADMIN -e ServerIP="$IP" --name pihole -d $IMAGE

# Recommended auto ad list updates & log rotation:
wget -O- https://raw.githubusercontent.com/diginc/docker-pi-hole/master/docker-pi-hole.cron | sudo tee /etc/cron.d/docker-pi-hole.cron
```

This is just an example and might need changing.  For exmaple of you're running on a raspberry pi over wireless you'll probably want to change your NIC variable to `wlan0` and IMAGE to `diginc/pi-hole:arm`

**Automatic Ad List Updates** - [docker-pi-hole.cron](https://github.com/diginc/docker-pi-hole/blob/master/docker-pi-hole.cron) is a modified verion of upstream pi-hole's crontab entries using `docker exec` to run the same update scripts inside the docker container.  The cron automatically updates pi-hole ad lists and cleans up pi-hole logs nightly.  If you're not using the `docker run` with `--name pihole` from default contariner run command be sure to fill in your container's DOCKER_NAME into the variable in the cron file.

**Tips**
* To customize your upstream DNS servers you use docker environment varibales of *DNS1* and *DNS2* passed into docker at runtime.  The default servers are Google's 8.8.8.8 and 8.8.4.4.
* A good way to test things are working right is by loading this page: [http://pi-hole.isworking.ok/admin/](http://pi-hole.isworking.ok/admin/)
* Ubuntu users especially may need to shutoff dnsmasq on your docker server so it can run in the container on port 53
* If you have another site/service using port 80 by default then the ads may not transform into blank ads correctly.  To make sure docker-pi-hole plays nicely with an exising webserver you run you'll probably need a reverse proxy websever config if you don't have one already.  Pi-Hole has to be the default web app on said proxy e.g. if you goto your host by IP instead of domain pi-hole is served out instead of any other sites hosted by the proxy. This behavior is taken advantage of so any ad domain can be directed to your webserver and get blank html/images/videos instead of ads.
 * [Here is an example of running with jwilder/proxy](https://github.com/diginc/docker-pi-hole/blob/master/jwilder-proxy-example-doco.yml) (an nginx auto-configuring docker reverse proxy for docker) on my port 80 with pihole on another port.  Pi-hole needs to be `DEFAULT_HOST` env in jwilder/proxy and you need to set the matching `VIRTUAL_HOST` for the pihole's container.  Please read jwilder/proxy readme for more info if you have trouble.  I tested this basic exmaple which is based off what I run.
* ServerIP environment variable is required to override/prevent pi-hole's script from returning a unreachable docker IP instead of your server's IP for ads.
* dnsmasq requires NET_ADMIN capabilities to run correctly in docker.

**Volume Mounts**
Here are some useful volume mount options to persist your history of stats in the admin interface, or add custom whitelists/blacklists.  **Create these files on the docker host first or you'll get errors**:

* `docker run -v /var/log/pihole.log:/var/log/pihole.log ...` (plus all of the minimum options added)
* `docker run -v /etc/pihole/blacklist.txt:/etc/pihole/blacklist.txt ...` (plus all of the minimum options added)
* `docker run -v /etc/pihole/whitelist.txt:/etc/pihole/whitelist.txt ...` (plus all of the minimum options added)
 * if you use this you should probably read the Advanced Usage section

All of these options get really long when strung together in one command, which is why I'm not showing all the full `docker run` commands variations here.  This is where [docker-compose](https://docs.docker.com/compose/install/) yml files come in handy for representing [really long docker commands in a readable file format](https://github.com/diginc/docker-pi-hole/blob/master/doco-example.yml).


## Docker tags

| tag                 | architecture | description                                                             | Dockerfile |
| ---                 | ------------ | -----------                                                             | ---------- |
| `alpine` / `latest` | x86          | Alpine x86 image, small size container running nginx and dnsmasq        | [Dockerfile](https://github.com/diginc/docker-pi-hole/blob/master/alpine.docker) |
| `debian`            | x86          | Debian x86 image, container running lighttpd and dnsmasq                | [Dockerfile](https://github.com/diginc/docker-pi-hole/blob/master/debian.docker) |
| `arm`               | ARM          | Debian ARM image, container running lighttpd and dnsmasq built for ARM  | [Dockerfile](https://github.com/diginc/docker-pi-hole/blob/master/debian-armhf.docker) |

### `diginc/pi-hole:alpine` (default tag) [![](https://images.microbadger.com/badges/image/diginc/pi-hole:alpine.svg)](http://microbadger.com/images/diginc/pi-hole "Get your own image badge on microbadger.com") [![](https://images.microbadger.com/badges/version/diginc/pi-hole:alpine.svg)](http://microbadger.com/images/diginc/pi-hole "Get your own version badge on microbadger.com") [![](https://images.microbadger.com/badges/version/diginc/pi-hole:latest.svg)](http://microbadger.com/images/diginc/pi-hole "Get your own version badge on microbadger.com") 

### `diginc/pi-hole:debian` [![](https://images.microbadger.com/badges/image/diginc/pi-hole:debian.svg)](http://microbadger.com/images/diginc/pi-hole "Get your own image badge on microbadger.com") [![](https://images.microbadger.com/badges/version/diginc/pi-hole:debian.svg)](http://microbadger.com/images/diginc/pi-hole "Get your own version badge on microbadger.com")

This version of the docker aims to be as close to a standard pi-hole installation by using the same base OS and the exact configs and scripts (minimally modified to get them working).  This serves as a nice baseline for merging and testing upstream repository pi-hole changes.

### `diginc/pi-hole:arm` (Debian)

As close to the debian image as possible, but cross compiled for ARM architecture hardware through [resin.io's awesome Qemu wrapper](https://resin.io/blog/building-arm-containers-on-any-x86-machine-even-dockerhub/).

Alpine doesn't have an arm cross compileable image at this time.

## Persistence and Customizations

The standard pi-hole customization abilities apply to this docker, but with docker twists such as using docker volume mounts to map host stored file configurations over the container defaults.  Volumes are also important to persist the configuration incase you have remove the pi-hole container which is a typical docker upgrade pattern.

### Volumes

Here are some relevant wiki pages from pi-hole's documentation and example volume mappings to optionally add to the basic example:

* [Customizing sources for ad lists](https://github.com/pi-hole/pi-hole/wiki/Customising-sources-for-ad-lists)
 * `-v your-adlists.list:/etc/pihole/adlists.list` Your version should probably start with the existing defaults for this file.
* [Whitlisting and Blacklisting](https://github.com/pi-hole/pi-hole/wiki/Whitelisting-and-Blacklisting)
 * `-v your-whitelist:/etc/pihole/whitelist.txt` Your version should probably start with the existing defaults for this file.
 * `-v your-blacklist:/etc/pihole/blacklist.txt` This one is empty by default

### Scripts

The original pi-hole scripts are in the container so they should work via `docker exec <container> <command>` like so:

* `docker exec pihole_container_name pihole updateGravity`
* `docker exec pihole_container_name whitelist.sh some-good-domain.com`
* `docker exec pihole_container_name blacklist.sh some-bad-domain.com`

### Customizations

Any configuration files you volume mount into `/etc/dnsmasq.d/` will be loaded by dnsmasq when the container starts or restarts or if you need to modify the pi-hole config it is located at `/etc/dnsmasq.d/01-pihole.conf`.  The docker start scripts runs a config test prior to starting so it should tell you about any errors in the docker log.

Similarly for the webserver you can customize configs in /etc/nginx (*:alpine* tag) and /etc/lighttpd (*:debian* tag).
