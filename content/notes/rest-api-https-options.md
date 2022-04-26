---
title: "REST API HTTPs options"
date: 2022-04-27
description: "Options on how to secure REST API using HTTPs certificate"
---

# REST API HTTPs options

## PaaS (Heroku)

Heroku automatically provides a subdomain (`app.heroku.app`) with issues HTTPs certificate for all dynos. So no configuration is needed to manage (issue and renew) certificate.

## IaaS (DigitalOcean)

If you are running your backend server on the VPS instance you will be only presented with static *public IP address* (HTTP) for that machine.

These are the available proxy setup options:

- [nginx](https://nginx.org/en/) + [certbot](https://github.com/certbot/certbot)

- **[docker-nginx-certbot](https://github.com/JonasAlfredsson/docker-nginx-certbot)**

- [nginx-proxy](https://github.com/nginx-proxy/nginx-proxy) + [acme-companion](https://github.com/nginx-proxy/acme-companion)

- [nginx-proxy-manager](https://github.com/NginxProxyManager/nginx-proxy-manager)

- [traefik](https://github.com/traefik/traefik)



## Text

In the modern world the web application is split into backend and frontend. Not only the website client needs to securely fetch a frontend files, but also the frontend needs to **securely communicate with the backend** (pass passwords, etc.).

Depending on the cloud computing model (PaaS or IaaS) there are different options to achieve this.

For small projects PaaS is enough, but when it gets bigger and more complicated it might be easier to deploy on VPS instance instead. In this case Nginx will be your friend in managing certificates and a lot of other stuff instead of the main backend process.

There are many pre-configured setups for running proxy with Let's Encrypt. I would recommend [docker-nginx-certbot](https://github.com/JonasAlfredsson/docker-nginx-certbot) because it provides very *minimum abstraction* on top nginx and lets you configure it *as usual* without dealing with SSL/Let's Encrypt.
