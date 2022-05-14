# Docktool (Forked from `praqma/Network-MultiTool`)

A (**multi-arch**) multitool for container/network testing and troubleshooting. The main docker image is based on Alpine Linux.

The container image contains lots of tools, as well as a `nginx` web server, which listens on port `80` and `443` by default. The web server helps to run this container-image in a straight-forward way, so you can simply `exec` into the container and use various tools.

The docker repository to pull this image is now: [https://hub.docker.com/r/ra55/docktool (https://hub.docker.com/r/ra55/docktool)

Or:

```
docker pull ra55/docktool
```


## Supported platforms: 
* linux/386
* linux/amd64
* linux/arm/v7
* linux/arm64

## Downloadable from Docker Hub: 
* [https://hub.docker.com/r/ra55/docktool](https://hub.docker.com/r/ra55/docktool)  (An automated multi-arch build)


## Tools included in "latest":
* apk package manager
* Nginx Web Server (port `80`, port `443`) - with customizable ports!
* awk, cut, diff, find, grep, sed, wc
* curl, wget
* dig, nslookup
* ip, ifconfig, route
* traceroute, tracepath, mtr, tcptraceroute (for layer 4 packet tracing)
* ping, arp, arping
* ps, netstat
* gzip, cpio, tar
* telnet client
* tcpdump
* jq
* bash
* openssh-client, nmap, rsync, netcat-openbsd, grep, tar, sed, lsof

**Size:** 56 MB compressed, 68 MB uncompressed

**Note:** The SSL certificates are generated for "localhost", are self signed, and placed in `/certs/` directory. During your testing, ignore the certificate warning/error. While using curl, you can use `-k` to ignore SSL certificate warnings/errors.

------

# How to use this image? 
## How to use this image in normal **container/pod network** ?

### Docker:
```
$ docker run  -d ra55/docktool
```

Then:

```
$ docker exec -it container-name /bin/bash
```


### Kubernetes:

Create single pod - without a deployment:
```
$ kubectl run docktool --image=ra55/docktool
```

Create a deployment:
```
$ kubectl create deployment docktool --image=ra55/docktool
```

Then:
```
$ kubectl exec -it pod-name /bin/bash
```

**Note:** You can pass additional parameter `--namespace=<your-desired-namespace>` to the above kubectl commands.


## How to use this image on **host network** ?

Sometimes you want to do testing using the **host network**.  This can be achieved by running the docktool using host networking. 


### Docker:
```
$ docker run --network host -d ra55/docktool
```

**Note:** If port 80 and/or 443 are already busy on the host, then use pass the extra arguments to docktool, so it can listen on a different port, as shown below:

```
$ docker run --network host -e HTTP_PORT=1180 -e HTTPS_PORT=11443 -d ra55/docktool
```

### Kubernetes:
For Kubernetes, there is YAML/manifest file `multitool-daemonset.yaml` in the `kubernetes` directory, that will run an instance of the docktool on all hosts in the cluster using host networking.

```
$ kubectl apply -f kubernetes/multitool-daemonset.yaml
```

**Notes:** 
* You can pass additional parameter `--namespace=<your-desired-namespace>` to the above kubectl command.
* Due to a possibility of something (some service) already listening on port 80 and 443 on the worker nodes, the `daemonset` is configured to run docktool on port `1180` and `11443`. You can change this in the YAML file if you want.


# Configurable HTTP and HTTPS ports:
There are times when one may want to join this (docktool) container to another container's IP namespace for troubleshooting, or on the host network. This is true for both Docker and Kubernetes platforms. During that time if the container in question is a web server (nginx, apache, etc), or a reverse-proxy (traefik, nginx, haproxy, etc), then network-multitool cannot join it in the same IP namespace on Docker, and similarly it cannot join the same pod on Kubernetes. This happens because network multitool also runs a web server on port 80 (and 443), and this results in port conflict on the same IP address. To help in this sort of troubleshooting, there are two environment variables **HTTP_PORT** and **HTTPS_PORT** , which you can use to provide the values of your choice instead of 80 and 443. When the container starts, it uses the values provided by you/user to listen for incoming connections. Below is an example:

```
$ docker run -e HTTP_PORT=1180 -e HTTPS_PORT=11443 \
    -p 1180:1180 -p 11443:11443 -d local/network-multitool
4636efd4660c2436b3089ab1a979e5ce3ae23055f9ca5dc9ffbab508f28dfa2a


$ docker ps
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                                                             NAMES
4636efd4660c        local/network-multitool   "/docker-entrypoint.…"   4 seconds ago       Up 3 seconds        80/tcp, 0.0.0.0:1180->1180/tcp, 443/tcp, 0.0.0.0:11443->11443/tcp   recursing_nobel
6e8b6ed8bfa6        nginx                     "nginx -g 'daemon of…"   56 minutes ago      Up 56 minutes       80/tcp                                                            nginx


$ curl http://localhost:1180
Praqma Network MultiTool (with NGINX) - 4636efd4660c - 172.17.0.3/16 - HTTP: 1180 , HTTPS: 11443


$ curl -k https://localhost:11443
Praqma Network MultiTool (with NGINX) - 4636efd4660c - 172.17.0.3/16 - HTTP: 1180 , HTTPS: 11443
```  

If these environment variables are absent/not-provided, the container will listen on normal/default ports 80 and 443.

------

# FAQs
## Why this multitool runs a web-server?
Well, normally, if a container does not run a daemon/service, then running it (the container) involves using *creative ways / hacks* to keep it alive. If you don't want to suddenly start browsing the internet for "those creative ways", then it is best to run a small web server in the container - as the default process. 

This helps you when you are using Docker. You simply execute:
```
$ docker run  -d wbitt/network-multitool
```

This also helps when you are using kubernetes. You simply execute:
```
$ kubectl run multitool --image=wbitt/network-multitool
```


The multitool container starts as web server - so it remains `UP`. Then, you simply connect to it using:
```
$ docker exec -it some-silly-container-name /bin/sh 
```

Or, on Kubernetes:
```
$ kubectl exec -it multitool-3822887632-pwlr1  -- /bin/sh
```

This is why it is good to have a web-server in this tool. Hope this answers the question! Besides, I believe that having a web server in a multitool is like having yet another tool! Personally, I think this is cool! [Henrik](https://www.linkedin.com/in/henrikrenehoegh/) thinks the same!


## I can't find a tool I need for my use-case?
We have tried to put in all the most commonly used tools, while keeping it small and practical. We can't have all the tools under the sun, otherwise it will end up as [something like this](https://www.amazon.ca/Wenger-16999-Swiss-Knife-Giant/dp/B001DZTJRQ).  

However, if you have a special need, for a special tool, for your special use-case, then I would recommend to simply build your own docker image using this one as base image, and expanding it with the tools you need.

## Why not use LetsEncrypt for SSL certificates instead of generating your own?
There is absolutely no need to use LetsEncrypt. This is a testing tool, and validity of SSL certificates does not matter.

## Why use a daemonset when troubleshooting host networking on Kubernetes?
One could argue that it is possible to simply install the tools on the hosts and get over with it. However, we should keep the infrastructure immutable and not install anything on the hosts. *Ideally* we should never `ssh` to our cluster worker nodes. Some of the reasons are:

* It is generally cumbersome to install the tools since they might be needed on several hosts.
* New packages may conflict with existing packages, and *may* break some functionality of the host.
* Removing the tools and dependencies after use could be difficult, as it *may* break some functionality of the host.
* By using a `daemonset`, it makes it easier to integrate with other resources. e.g. Use volumes for packet capture files, etc.
* Using the `daemonset` provides a *'cloud native'* approach to provision debugging/testing tools.
* You can `exec` into the `daemonset`, without needing to SSH into the node.


## How to contribute to this project?
Contributions are welcome for packages/tools considered **"absolutely necessary"**, of **"core"** nature, are **"minimal"** in size, and **"have large number of use-cases"**. Remember, the goal is not to create yet another Linux distribution! :)

