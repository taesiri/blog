---
layout: post
title: "Handling Disconnexion!"
author: "Mohammad Reza Taesiri"
categories: random
tags: [Random, Freedom, Disconnexion]
image: freenet/heading.jpg
---

<link rel="stylesheet" href="../assets/css/table.css">

<link rel="stylesheet" href="../assets/css/redact.css">

There are times that your internet connection is heavily monitored, restricted, and firewalled. This could happen when you are behind a corporate firewall, some regions with limited internet connectivity, or mandatory censorship. In these scenarios, there is at least one layer between the actual internet and you. These extra layers perform monitoring and "*purifying*" the net. They don't allow you to visit a certain part on the internet by dropping packets from your connection. In the old times, these layers were just a set of hard-coded predefined rules, and they were easy to bypass. Nowadays, with the advance of Machine Learning, the hard-coded rules have changed and become far more sophisticated. They perform pattern matching on your traffic, and if they think you are trying to bypass the restriction, they will block your network's traffic. In this short post, I am going to briefly explain some of the handy methods to overcome such restrictions.

----

## SSH ProxyJump

{% include images.html url="../assets/img/freenet/ssh-bridge.png" description="SSH ProxyJump Scenarios" %}

SSH is by far one of the most interesting pieces of software I have ever used. It has many capabilities to create secure tunnels between computers, but there is a technique called ``ProxyJump`` which I needed for a long time and I didn't know it was there. Assume you want to use a middle computer as a bridge to access another computer, which is not accessible directly. You can create nested SSH tunnels or simply use ``ProxyJump``.

For example, if ``remote.host`` is only accessible via ``bridge.host``, you can connect to it like this:

```bash
ssh -J bridge.host remote.host
```

To Transfer file using ``scp`` do this:

```bash
scp -o 'ProxyJump bridge.host' file remote.host:/file
```

To create a ``socks5`` proxy server which re-routes traffic to ``remote.host``, just use ``-D`` option (dynamic port forwarding), like when you use it with a regular SSH connection.

```bash
ssh -J bridge.host remote.host -D 3000
```

### Learn More About SSH

1. [Using ProxyJump with SSH and SCP - Tutorial](https://www.madboa.com/blog/2017/11/02/ssh-proxyjump/)
1. [More on ProxyJump](https://en.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts)
1. [SSH VPN](https://help.ubuntu.com/community/SSH_VPN)
1. [Mosh: The Mobile Shell](https://mosh.org/)

----

## IP over DNS

{% include images.html url="../assets/img/freenet/ip-over-dns.png" description="IP over DNS - When DNS queries are possible" %}

One of the coolest (and also slowest) things you can do is create a tunnel over DNS queries! yes, DNS queries. When our government killed the internet, we still could access the internal websites, so DNS servers were working, and DNS tunnels were possible. There are lots of tools to create such tunnels, my personal choice is *iodine*.

### Quick Walkthrough

1. Create a ``A`` Record called ``y`` on your DNS server which points to your remote server IP Address.
1. Create an ``NS`` Record called ``x`` which points to ``y.YOUR-DOMAIN.com``.
1. Compile *iodine* on both your server and client.
1. On the server run this: ``iodined -f 172.16.0.1 -P PASSWORD x.YOUR-DOMAIN.com``.
1. On (MacOS) Client run this: ``sudo ./iodine -d utunX -f -P PASSWORD x.YOUR-DOMAIN.com``.
    * Sometimes you should do this: ``sudo ./iodine -d utunX -f -P PASSWORD 1.1.1.1 x.YOUR-DOMAIN.com``.
1. Now your client and server are sharing a network, your server IP address is ``172.16.0.1`` and your client will be ``172.16.0.2`` (or the next available IP address).

### Learn More About IP Over DNS

1. [*iodine* Project Home Page](https://code.kryo.se/iodine/)
1. [*iodine* Github Page](https://github.com/yarrick/iodine)
1. [Tunneling network traffic over DNS with Iodine and a SSH SOCKS proxy - Tutorial](https://davidhamann.de/2019/05/12/tunnel-traffic-over-dns-ssh/)
1. [Tunneling all traffic over DNS with a SOCKS proxy - Tutorial](https://0day.work/tunneling-all-traffic-over-dns-with-a-socks-proxy/)
1. [PTunnel](https://www.mit.edu/afs.new/sipb/user/golem/tmp/ptunnel-0.61.orig/web/) - (IP over ICMP Ping - Not very useful but a cool project).
1. [SoftEther VPN Project](https://www.softether.org/1-features/1._Ultimate_Powerful_VPN_Connectivity) (IP Over HTTPS)

----

## V2Ray

V2Ray or [Project V](https://www.v2ray.com/en/index.html) is a set of tools to create proxies and tunnels. With a single *JSON* configuration file you can define multiple types of inbound and outbound and you also can define routing policies, and load balancers. For example, you can have an inbound of type *Shadowsocks* and redirect all incoming traffic to another V2Ray server using *vmess* protocol. You can find a list of supported protocols [here](https://www.v2ray.com/en/configuration/protocols.html). With WebSocket plugin of this project, you can hide your inbound behind a regular webserver, thus from the outside world, your server is just serving a website, thus making detection of it harder. Another interesting thing is a combination of ``Caddy`` or ``nginx`` and a CDN like a Cloudflare, which makes your server harder and harder to catch. Of course, detection is a matter of when, and given enough time, any of such services can be detected, but with these techniques, you are going to buy yourself time.

### V2Ray - Example 1

{% include images.html url="../assets/img/freenet/vmess-cdn-caddy.png" description="Cloudflare + Caddy + V2Ray -> Socks" %}

The first example is using Cloudflare + Caddy + V2Ray (using ``VMess + Websocket`` as an Inbound here) to receive incoming connections and forward it to a Socks proxy! It might be crazy but, I want to show you how to connect your outbound to another protocol. If you want to connect your outbound directly to the Internet, just use the ``freedom`` protocol as an outbound.

**Remember**, The usage of Cloudflare CDN in here is just because of hiding the outside VPS from compromisation. Without using CDN, your server might be not reachable after a short period of using Shadowoskcs, or even simple SSH Tunnel. Keep in mind, this setup might have high latency in some regions.

Simple ``Caddyfile`` configuration which redirects connections to ``YOUR-DOMAIN.com/RAY-PATH`` to V2Ray server.

```Caddyfile
YOUR-DOMAIN.com {
  log ./caddy.log
  root /root/static
  gzip

  proxy /RAY-PATH v2ray:10000 {
    websocket
    header_upstream -Origin
  }
}
```

Simple ``config.json`` for V2Ray, which listens to inbound of type ``vmess`` (with ``WebSocket`` plugin) and forwards it to a socks proxy server.

```json
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 10000,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "USER-UUID",
            "alterId": 32
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/RAY-PATH"
        }
      },
      "listen": "0.0.0.0"
    }
  ],
  "outbounds": [
    {
      "protocol": "socks",
      "settings": {
        "servers": [
          {
            "address": "Socks-Server-IP",
            "port": 1080
          }
        ]
      }
    }
  ]
}
```

Here is a ``docker-compose.yaml`` file that automatically creates all the necessary containers for you.

```yaml
version: '3'

services:
  v2ray:
    image: teddysun/v2ray
    volumes:
      - ./config.json:/etc/v2ray/config.json
    expose:
      - "10000"

  caddy:
    image: abiosoft/caddy
    volumes:
      - ./Caddyfile:/etc/Caddyfile:ro
      - ./caddyCertificates:/root/.caddy
      - ./data:/root/static
    environment:
      - ACME_AGREE=true
    ports:
      - "80:80"
      - "443:443"
```

### V2Ray - Example 2

{% include images.html url="../assets/img/freenet/vmess-cdn-loadbalancer.png" description="V2Ray Load Balancer" %}

You can create load balancer with V2Ray, here is a simple ``json`` config to do so:

```json
{
  "log": {
    "access": "/var/log/v2ray/access.log",
    "error": "/var/log/v2ray/error.log",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "tag": "in-0",
      "port": 10000,
      "protocol": "vmess",
      "settings": {
        "clients": [
          {
            "id": "USER-UUID",
            "alterId": 32
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/RAY-PATH"
        }
      },
      "listen": "0.0.0.0"
    }
  ],
  "outbounds": [
    {
      "tag": "internet-gate-0",
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "OTHER-VMESS-SERVER-ADDRESS",
            "port": 443,
            "users": [
              {
                "id": "UIUD",
                "alterId": 32,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/RAY-PATH-0"
        }
      }
    },
    {
      "tag": "internet-gate-1",
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "OTHER-VMESS-SERVER-ADDRESS",
            "port": 443,
            "users": [
              {
                "id": "UIUD",
                "alterId": 32,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/RAY-PATH-1"
        }
      }
    },
    {
      "tag": "internet-gate-2",
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "OTHER-VMESS-SERVER-ADDRESS",
            "port": 443,
            "users": [
              {
                "id": "UIUD",
                "alterId": 32,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/RAY-PATH-2"
        }
      }
    },
    {
      "tag": "internet-gate-3",
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "OTHER-VMESS-SERVER-ADDRESS",
            "port": 443,
            "users": [
              {
                "id": "UIUD",
                "alterId": 32,
                "security": "auto"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "wsSettings": {
          "path": "/RAY-PATH-3"
        }
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "blocked",
      "protocol": "blackhole",
      "settings": {}
    }
  ],
  "routing": {
    "domainStrategy": "IPOnDemand",
    "rules": [
      {
        "type": "field",
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "direct"
      },
      {
        "type": "field",
        "inboundTag": [
          "in-0"
        ],
        "balancerTag": "my-balance"
      }
    ],
    "balancers": [
      {
        "tag": "my-balance",
        "selector": [
          "internet-gate-"
        ]
      }
    ]
  },
  "policy": {},
  "reverse": {},
  "transport": {}
}
```

----

### More on V2Ray

1. [Official Homepage](https://www.v2ray.com/en/)
1. You can find lots of configuration [Here](https://github.com/veekxt/v2ray-template)

## Chisel

Chisel creates a TCP tunnel over HTTP and It is magically fast! I think faster than anything I ever have seen before. With a click of the purple button on the bellow, you can deploy a Chisel server on Heroku. You only have to set credentials in form of ``user:password`` and the ``URL`` for your Heroku app.

<a href="https://heroku.com/deploy?template=https://github.com/mrluanma/chisel-heroku" rel="nofollow"><img src="https://camo.githubusercontent.com/83b0e95b38892b49184e07ad572c94c8038323fb/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e737667" alt="Heroku Deploy" data-canonical-src="https://www.herokucdn.com/deploy/button.svg" style="max-width:100%;"></a>

On the client it is as simple as running the following command:

```bash
chisel client --auth USER:PASSWORDS URL_OF_HEROKU_APP 0.0.0.0:1080:socks
```

After connecting to the server, you have a Socks5 proxy on your client-side, completely free!

### Custom ``Dockerfile``

Here is a custom ``Dockerfile`` I am using on multiple platforms. For some reason, the official ARM docker images didn't work for me. For each platform, just change the ``CHISEL_BINARY_URL.gz`` with appropriate one ([official builds are here](https://github.com/jpillora/chisel/releases).)

```Dockerfile
FROM alpine
RUN apk update && apk add --no-cache ca-certificates
RUN apk --no-cache add curl  
RUN curl -sSL CHISEL_BINARY_URL.gz | gzip -d - > /bin/chisel
RUN chmod +x /bin/chisel
ENTRYPOINT ["/bin/chisel"]
```

### Learn More About Chisel

1. [Github Page](https://github.com/jpillora/chisel)
1. [Official Docker](https://hub.docker.com/r/jpillora/chisel)
1. [Setup Chisel on Heroku - Tutorial](https://medium.com/@spscream/socks5-server-using-heroku-docker-chisel-step-by-step-howto-8e6bb358f7ff)
1. [Deploy Chisel On Heroku](https://github.com/mrluanma/chisel-heroku)
1. [Crowbar](https://github.com/q3k/crowbar)

----

## Client side evasion with Geneva

Geneva is one of the client-side evasion techniques which tries to bypass restrictions without any help from outside (a server). This tool (and also other client-side methods) try to tamper, modify network packets to trick "Firewalls" into believing that packet is safe to travel in the network. The difference between Geneva and other client-side tools is that Geneva uses Evolutionary algorithms to automatically find a new set of strategies to evade restrictions.

A Naïve approach to use a genetic algorithm for tampering the network packets would be in Bit-level, that's it, consider each packet's bitstream and manipulate those. This would, in theory, find all the possible manipulation types and eventually will find all the possible working strategies, but It is slow as hell! Not very useful. The authors of Geneva came up with a higher level manipulation scheme which is super fast and also useful. For example, the following directive will first duplicate ``ACK`` packets, and for the first duplicate, it will change the ``TCP flags`` field to ``RST``. See [Here](https://github.com/Kkevsterrr/geneva#strategy-dna) for more details.

```bash
[TCP:flags:A]-duplicate(tamper{TCP:flags:replace:R},)
```

This is a very powerful way to express strategies and works well with a genetic algorithm. Kudos to the authors of this paper!

~~I did try Geneva last day but unfortunately, none of the strategies presented in [this file](https://github.com/Kkevsterrr/geneva/blob/master/strategies.md) worked for me! :-(~~

**Update (12/14/19)**:  In my previous attempts, I made a few mistakes and after [reaching out to project owners](https://github.com/Kkevsterrr/geneva/issues/6), now Geneva works like a charm:

{% include images.html url="../assets/img/freenet/70827438-82b4b100-1dfe-11ea-8ef6-e23f65087fbf.png" description="Browsing Twitter with Geneva" %}

I tested all strategies published in the paper and found the following ones that work here.

**What works with HTTP**

| Species    | Sub-Species  | Variant   | Genetic Code                                                                                |
|------------|--------------|-----------|---------------------------------------------------------------------------------------------|
| TCB Desync | Inc. Dataofs | Small TTL |   `[TCP:flags:PA]-duplicate(tamper{TCP:dataofs:replace:10}(tamper{IP:ttl:replace:10},),)-\|`   |
| TCB Desync | Inv. Payload | Small TTL | `[TCP:flags:PA]-duplicate(tamper{TCP:load:corrupt}(tamper{IP:ttl:replace:8},),)-\|`      |

**What works with HTTPS**

| Species      | Sub-Species | Variant     | Genetic Code                                                     |
|--------------|-------------|-------------|------------------------------------------------------------------|
| TCB Desync   | Simple      | Payload syn |   `[TCP:flags:S]-duplicate(,tamper{TCP:load:corrupt})-\|`  |
| Segmentation | With ack    | Offsets     |  `[TCP:flags:PA]-fragment{tcp:8:False}-\| [TCP:flags:A]-tamper{TCP:seq:corrupt}-\|`                    |
| Segmentation | Reassembly  | Offsets     | `[TCP:flags:PA]-fragment{tcp:8:True}(,fragment{tcp:4:True})-\|`  |
| Segmentation | Simple      | In-Order    | `[TCP:flags:PA]-fragment{tcp:-1:True}-\| `                       |

Certain websites don't quite work well just now, (like [BBC.CO.UK](https://www.bbc.co.uk/), and [Youtube](https://www.youtube.com/)), but that's not a problem, I am sure more strategies will fix this. (I also have to do more tests to find out what's wrong here, Clearly, Geneva works, that is the fact! I think these websites can't handle the traffic Geneva produces. Maybe!)

I probably write a more in-depth blog post about Geneva, and why I am so excited about this particular project in the future.

### Learn more about this techniques and Geneva

1. [Geneva Project Paper](https://geneva.cs.umd.edu/papers/geneva_ccs19.pdf)
1. [Geneva Project Homepage](https://geneva.cs.umd.edu/)
1. [Geneva Github Page](https://github.com/Kkevsterrr/geneva)
1. [A Cool writeup](http://blog.zorinaq.com/my-experience-with-the-great-firewall-of-china/) by Marc Bevand about his experience in China. I highly recommend this to read for everyone that wants to have a better understanding.
1. [GoodbyeDPI](https://github.com/ValdikSS/GoodbyeDPI) - Passive Deep Packet Inspection blocker and Active DPI circumvention utility
1. [Green Tunnel](https://github.com/SadeghHayeri/GreenTunnel) - Green Tunnel is an anti-censorship utility designed to bypass DPI system that are put in place by various ISPs to block access to certain websites.

## Image Credits

1. [Cover Image by  Jack Moreh](https://www.stockvault.net/photo/239787/face-off-challenge-between-businessmen-overcoming-barriers)
1. Diragrams made with [Draw.io](https://www.draw.io/)