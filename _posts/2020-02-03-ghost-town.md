---
layout: post
title: "VMessy Gostly Trojanic Chiselling Town"
author: "Mohammad Reza Taesiri"
categories: random
tags: [Random, Freedom, Disconnection]
image: ghosts/heading.jpg
---

<link rel="stylesheet" href="../assets/css/table.css">
<link href="https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css" rel="stylesheet" integrity="sha384-wvfXpqpZZVQGK6TAh5PVlGOfQNHSoD2xbE+QkPxCAFlNEevoEH3Sl0sibVcOQVnN" crossorigin="anonymous">

There are countless methods for tunneling/proxying traffics from one endpoint to another. In this post, I am going to explain some of the more widespread techniques used in my neighborhood. (All guides obtained from the Net, and I am just gathering them in one place).

----

## TL;DR

| Method 	| Server 	| Server Params      	| Client         	|
|--------	|--------	|--------------------	|----------------	|
| <a href="#v2ray">V2Ray</a>  	| <a href="https://heroku.com/deploy?template=https://github.com/onplus/v2hero/tree/core-latest" rel="nofollow"><img src="https://camo.githubusercontent.com/c0824806f5221ebb7d25e559568582dd39dd1170/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e706e67" alt="" data-canonical-src="https://www.herokucdn.com/deploy/button.png" style="max-width:100%;"></a>  	| ``UUID``           	|    <a href="../assets/other-contents/configs/sample.json"> Sample JSON <i class="fa fa-download"></i> </a> 	|
| <a href="#gost">Gost</a>   	| <a href="https://heroku.com/deploy?template=https://github.com/xuiv/gost-heroku" rel="nofollow"><img src="https://camo.githubusercontent.com/c0824806f5221ebb7d25e559568582dd39dd1170/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e706e67" alt="" data-canonical-src="https://www.herokucdn.com/deploy/button.png" style="max-width:100%;"></a> 	|                    Nothing	| <a href="#gost-client" >Client Command</a>	|
| <a href="#chisel">Chisel</a> | <a href="https://heroku.com/deploy?template=https://github.com/mrluanma/chisel-heroku" rel="nofollow"><img src="https://camo.githubusercontent.com/83b0e95b38892b49184e07ad572c94c8038323fb/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e737667" alt="Heroku Deploy" data-canonical-src="https://www.herokucdn.com/deploy/button.svg" style="max-width:100%;"></a> 	| ``USER:PASSWORDS`` 	|      <a href="#chisel-client">Client Command</a>          	|
|  <a href="#trojan">Trojan</a>   	| [Installer Script](https://github.com/atrandys/trojan) 	|        Nothing            	|         Refer to installer script's repo      	|
| <a href="#cloak">Cloak</a>  	| [Installer Script](https://github.com/HirbodBehnam/Shadowsocks-Cloak-Installer)  	|          Nothing          	|     Refer to installer script's repo           	|



----

<div id="v2ray"></div>
## V2Ray on Heroku

### Server

One-click Deploy on Heroku using [v2Hero](https://github.com/onplus/v2hero). The only parameter you need to specify is ``UUID``. You can use [this site](https://www.uuidgenerator.net/) to generate UUIDs.

<a href="https://heroku.com/deploy?template=https://github.com/onplus/v2hero/tree/core-latest" rel="nofollow"><img src="https://camo.githubusercontent.com/c0824806f5221ebb7d25e559568582dd39dd1170/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e706e67" alt="" data-canonical-src="https://www.herokucdn.com/deploy/button.png" style="max-width:100%;"></a>

### Clients

Client configuration can be tricky cause most GUI-based clients have some limitation to fully configure the client. The best way is to edit a configuration is by text editor. Here is a valid sample ``JSON`` file for ``v2Hero``-based servers:

```json
{
  "routing" : {
    "name" : "all_to_main",
    "domainStrategy" : "AsIs",
    "rules" : [
      {
        "type" : "field",
        "outboundTag" : "SOME_TAG",
        "port" : "0-65535"
      }
    ]
  },
  "inbounds" : [
    {
      "listen" : "0.0.0.0",
      "protocol" : "socks",
      "settings" : {
        "ip" : "127.0.0.1",
        "auth" : "noauth",
        "udp" : true
      },
      "tag" : "socksinbound",
      "port" : 1081
    },
    {
      "listen" : "0.0.0.0",
      "protocol" : "http",
      "settings" : {
        "timeout" : 0
      },
      "tag" : "httpinbound",
      "port" : 8001
    }
  ],
  "dns" : {
    "servers" : [
      "1.1.1.1"
    ]
  },
  "outbounds" : [
    {
      "sendThrough" : "0.0.0.0",
      "mux" : {
        "enabled" : false,
        "concurrency" : 8
      },
      "protocol" : "vmess",
      "settings" : {
        "vnext" : [
          {
            "address" : "SERVER-URL-HERE",
            "users" : [
              {
                "id" : "UUID-HERE",
                "alterId" : 64,
                "security" : "auto",
                "level" : 1
              }
            ],
            "port" : 443
          }
        ]
      },
      "tag" : "V2HERO-HEROKU",
      "streamSettings": {
          "network": "ws",
          "security": "tls",
          "wsSettings": {
              "path": "/"
          }
      }
    }
  ]
}
```

1. [V2RayX - A Client for MacOS](https://github.com/Cenmrev/V2RayX)
1. [V2RayW - A Client for Windows](https://github.com/Cenmrev/V2RayW)
1. [ShadowRocket - An iOS Client](https://apps.apple.com/us/app/shadowrocket/id932747118)
1. [V2RayNG - An Android Client](https://play.google.com/store/apps/details?id=com.v2ray.ang)

### Learn More about ``v2Hero``

1. [v2Hero Github Repository](https://github.com/onplus/v2hero)

----

<div id="gost"></div>
## Gost on Heroku

Gost is branded as a simple security tunnel written in Golang, but trust me, it is everything except for simple! It offers multiple transport layers, protocols, load-balancing, and a lot more.

### Server

The easiest way to deploy Gost on Heroku is with the help of a project called ``gost-heroku``. No configuration is needed, everything is taken care of, just deploy a Heroku app using the button below:

<a href="https://heroku.com/deploy?template=https://github.com/xuiv/gost-heroku" rel="nofollow"><img src="https://camo.githubusercontent.com/c0824806f5221ebb7d25e559568582dd39dd1170/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e706e67" alt="" data-canonical-src="https://www.herokucdn.com/deploy/button.png" style="max-width:100%;"></a>

### Clients 

<div id="gost-client"></div>

With ``Gost`` binary on any platform:

```bash
./gost -L=:1080 -F=quick+wss://SERVER_URL.herokuapp.com:443
```

Or with official Docker image:

```bash
 docker run -d -p 1080:1080 --restart=always ginuerzh/gost -L=:1080 -F="quick+wss://SERVER_URL.herokuapp.com:443"
```

Other Clients:

1. [ShadowRocket - An iOS Client](https://apps.apple.com/us/app/shadowrocket/id932747118)
1. [ShadowSocksGost Plugin for Android](https://github.com/xausky/ShadowsocksGostPlugin)

### Learn More About ``Gost`` and ``gost-heroku``

1. [Gost Github Page](https://github.com/ginuerzh/gost)
1. [Official Documentation](https://docs.ginuerzh.xyz/gost/en/)
1. [Gost-Heroku Github Page](https://github.com/xuiv/gost-heroku)

----

<div id="chisel"></div>
## Chisel

I described Chisel in my previous post, So I just copy-paste whatever I wrote there, in here:

Chisel creates a TCP tunnel over HTTP and It is magically fast! I think faster than anything I ever have seen before. With a click of the purple button on the bellow, you can deploy a Chisel server on Heroku. You only have to set credentials in form of ``user:password`` and the ``URL`` for your Heroku app.

<a href="https://heroku.com/deploy?template=https://github.com/mrluanma/chisel-heroku" rel="nofollow"><img src="https://camo.githubusercontent.com/83b0e95b38892b49184e07ad572c94c8038323fb/68747470733a2f2f7777772e6865726f6b7563646e2e636f6d2f6465706c6f792f627574746f6e2e737667" alt="Heroku Deploy" data-canonical-src="https://www.herokucdn.com/deploy/button.svg" style="max-width:100%;"></a>

On the client it is as simple as running the following command:

<div id="chisel-client"></div>
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

<div id="trojan"></div>
## Trojan

Trojan is the latest addition to our list, It is extremely fast and hard to detect. There is no magic "Deploy to Heroku" button just yet, but there are some easy to use scripts to set up a server in [this repository](https://github.com/atrandys/trojan). Here are one-line installers for server:

*Docker-based Installer for CentOS 7:*

```bash
curl -O https://raw.githubusercontent.com/atrandys/trojan/master/trojan_install.sh && chmod +x trojan_install.sh && ./trojan_install.sh
```

*Normal Installer for all destros:*

```bash
curl -O https://raw.githubusercontent.com/atrandys/trojan/master/trojan_install.sh && chmod +x trojan_install.sh && ./trojan_install.sh
```

### Clients

1. [ShadowRocket - An iOS Client](https://apps.apple.com/us/app/shadowrocket/id932747118)
1. [Igniter - An Under Development Android Client](https://github.com/trojan-gfw/igniter)

### Learn More About Trojan

1. [Trojan Github Page](https://github.com/trojan-gfw/trojan)
1. [Useful Scripts](https://github.com/atrandys/trojan)

----

<div id="cloak"></div>
## Cloak

Cloak is just another tool that obfuscates any traffic as a legitimate HTTPS traffic. It can be used to hide other protocols traffic, like [ShadowSocks](https://github.com/cbeuw/Cloak/wiki/Underlying-proxy-configuration-guides#shadowsocks) and even [OpenVPN](https://github.com/cbeuw/Cloak/wiki/Underlying-proxy-configuration-guides#openvpn).

### Clients

1. [ShadowRocket - An iOS Client](https://apps.apple.com/us/app/shadowrocket/id932747118)
1. [Cloak-android - An Android Client](https://github.com/cbeuw/Cloak-android)

### Learn More About Cloak

1. [Cloak Github Page](https://github.com/cbeuw/Cloak)
1. [Useful Scripts](https://github.com/HirbodBehnam/Shadowsocks-Cloak-Installer)
1. [Cloak Wiki - Underlying proxy configuration guides](https://github.com/cbeuw/Cloak/wiki/Underlying-proxy-configuration-guides)

----

## Image Credits

1. Header Image taken from [Freepik](https://www.freepik.com/free-vector/hand-drawing-illustration-lifestyle-concept_2782473.htm). <a href="https://www.freepik.com/free-photos-vectors/icon">Icon vector created by rawpixel.com - www.freepik.com</a>