---

title: "Object storage, custom domains, and HTTPs"
date: 2022-05-07
description: "Connecting custom domain name to object storage / CDN with managed HTTPs"

---

# Object storage, custom domains and HTTPs

Recently I wanted to deploy a static-generated site (Next.js) to the raw object storage (S3) and connect a CDN to it. But encountered an issue with the DNS provider that doesn't allow simply created an alias to other domain name with managed HTTPs.

## The case

You:

- want to store website in object storage (S3)
  
  - some subdomain will be given (`xyz.netlify.app`)

- want to use CDN for HTTPs certificate management + faster access
  
  - or use certificate manager to easily create/store/renew certificates

- want to have custom domain name (`mydomain.xyz`)

DNS record options:

- `A` - can only be created for public static IP address

- `CNAME` - can only be created for subdomain

- `ALIAS` - can be created for root domain (`google.com`) and sub domains (`www.google.com`) (not supported by all DNS providers)

Problem:

- **my DNS provider doesn't support `ALIAS` record**

Solution:

1. create a `CNAME` record for **`www`** subdomain (`www.google.com`) pointing at provided storage/CDN domain

2. make a redirect from root domain (`google.com`) to `www` subdomain using **DNS provider's web redirect service** (HTTP 301 permanent redirect)

Problems:

- web redirect (301) may charge additional money
