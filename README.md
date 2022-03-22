---
permalink: index.html
layout: default
---

# CA

This is a work-in-progress proof-of-concept for hosting internal Root
Certificate Authority (CA) assets in Github Pages.

Once everything is ironed out, this page will be finalized and a documentation
entry will be created and linked from here.

## Assets

- [DoubleU Root CA Certificate](/DoubleU_Root_CA.crt)
- [DoubleU Root CA Certificate Revocation List (CRL)](/DoubleU_Root_CA.crl)

## Why?

When setting up an internal PKI for a [homelab](https://www.reddit.com/r/homelab/),
you might find yourself facing a chicken-and-egg problem when it comes to how to
host PKI assets, often needing them before you have the ability to host them
internally, or you may not want to dedicate any hardware to a simple file
server.

Github Pages allows repositories to occupy subdomains, so you can use one to
dedicate to your CA assets, leaving you free to implement the rest of your PKI
in any way you see fit without tying up hardware locally.

## No HTTPS?

[RFC 5280](https://datatracker.ietf.org/doc/html/rfc5280) defines the x.509
standard and how certain assets are to be distributed.

- [&#167; 4.2.1.13](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.1.13) -
CRL Distribution Points (CDP)
- [&#167; 4.2.2.1](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.2.1) -
Authority Information Access (AIA)

The two CA assets that I'm concerned about are CRLs and certificates defined by
`caIssuers` AIA entries.

Both `MUST` be DER encoded and accessible only from HTTP. Some PKI frameworks
(notably Windows' `CryptoAPI`) will silently fail if either of these entries
contain HTTPS URIs.

Not having encryption isn't an issue since certificates and CRLs are
cryptographically signed and integrity can be verified independent of the
transport methodology.

## The How: Github Pages

What you need:

- A public domain
- DNS services for it

This domain uses Cloudflare for DNS services. If you also use Cloudflare, you
likely have `Always Use HTTPS` enabled under `SSL/TLS` > `Edge Certificates`.
This applies Let's Encrypt certificates to each domain record ***that is
proxied***. To get around this, set your `CNAME` record pointing to your pages
to be `DNS only`. Cloudflare will warn you that `DNS only` records will leak
your IP address, but this is a non-issue for Github Pages since those IPs aren't
a secret.

![Cloudflare DNS Record](/assets/images/cname.png)

Place a `CNAME` file at the root of your repository and a simple `index.html`
file to serve if you want. You can probably get away with it being empty.

Then, place your root certificate and CRL in your preferred location in the
repository. I chose the root level since this repository will only contain these
files (plus this Jekyll site).

## How to Use

For those who are just getting started with PKI, the CDP and `caIssuers` AIA
extension will not be present on your root certificate, but on the certificates
that your root signs.

Certificates signed by your root will show something similar under the `x509v3
Extensions`:

```sh
X509v3 extensions:
    . . .
    X509v3 CRL Distribution Points:

        Full Name:
            URI:http://ca.doubleu.codes/DoubleU_Root_CA.crl

    Authority Information Access:
        CA Issuers - URI:http://ca.doubleu.codes/DoubleU_Root_CA.crt
```
