# CA

This is the distribution point for the DoubleU Labs Root Certificate Authority
(CA), a YubiKey-backed offline CA for my Homelab.

## Assets

- Certificate: [DoubleU_Root_CA_01.crt](/DoubleU_Root_CA_01.crt)
- Certificate Revocation List (CRL): [DoubleU_Root_CA_01.crl](/DoubleU_Root_CA_01.crl)

## Why and How

When setting up my Homelab, I was faced with a bit of a chicken-and-egg problem:
I needed issue certificates to servers and services before I was able to set up
a service to publish the CA certificates. Sure, I could just `Accept the Risk`
and continue, but that's no fun.

I wanted a starting point for true Trust-On-First-Use (TOFU) without exception
and starting cooking up a way to make that happen.

Github Pages allows repositories to occupy subdomains, so I tested publishing
my offline Root assets there for a little while to see how it worked while I
spent time figuring out how to automate the entire process.

If you're interested in doing something similar, or just wish to learn how this
was accomplished:

[DoubleU Labs Docs - Root CA](https://labs.doubleu.codes/docs/root_ca/)

## No HTTPS?

[RFC 5280](https://datatracker.ietf.org/doc/html/rfc5280) defines the x.509
standard and how certain assets are to be distributed.

- [&#167; 4.2.1.13](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.1.13) -
CRL Distribution Points (CDP)
- [&#167; 4.2.2.1](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.2.1) -
Authority Information Access (AIA)

The two assets that we're concerned with are certificates defined by `caIssuers`
AIA extensions, and Certificate Revocation Lists (CRL) defined by
`cRLDistributionPoints` CDP extensions.

Both `MUST` be DER encoded and accessible only from HTTP. Some PKI frameworks
(notably Windows' `CryptoAPI`) will silently fail if either of these entries
contain HTTPS URIs, though *most* applications will retrieve them from either.

Not having encryption isn't an issue since certificates and CRLs are
cryptographically signed and integrity can be verified independent of the
transport methodology.
