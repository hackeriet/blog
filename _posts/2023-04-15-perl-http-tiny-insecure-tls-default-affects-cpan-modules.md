---
layout: post
title: "Perl's HTTP::Tiny has insecure TLS default, affecting CPAN.pm and other modules"
author: sgo
category: security
---
  
[HTTP::Tiny](https://metacpan.org/pod/HTTP::Tiny), is a http client included both in
Perl (since v5.13.9) and as a standalone CPAN module. It [does not verify TLS
certificates by default](https://metacpan.org/pod/HTTP::Tiny#SSL-SUPPORT) requiring users to opt-in with the `verify_SSL=>1` flag to verify the identity of the HTTPS server they are communicating with.

The module is used by
[many](https://hackeriet.github.io/cpan-http-tiny-overview/) distributions on
CPAN, and likely other open source and proprietary software.

> **NOTE:** This post summarizes security problems caused by the insecure default
> and how it affects code relying on it for https. For a
> discussion on how to address this problem, please see
> [RFC: Making SSL_verify safer](https://github.com/chansen/p5-http-tiny/issues/152)


## Affected CPAN Modules

After 
[PSA: HTTP::Tiny disabled SSL verification by
default!](https://www.reddit.com/r/perl/comments/111tadi/psa_httptiny_disabled_ssl_verification_by_default/) was posted on Reddit, we were reminded that this might be a bigger problem than first thought. 

So we searched trough
[metacpan-cpan-extracted](https://github.com/metacpan/metacpan-cpan-extracted)
to find distributions using HTTP::Tiny without specifying the cert verification
behaviour. Distros using it without mentioning `verify_SSL` somewhere in the code was flagged. See [hackeriet.github.io/cpan-http-tiny-overview](https://hackeriet.github.io/cpan-http-tiny-overview/) for the full list.

Most distributions we found did not enable the certificate verification feature, potentially exposing users to [machine-in-the-middle](https://www.internetsociety.org/resources/doc/2020/fact-sheet-machine-in-the-middle-attacks/Machine-in-the-middle) attacks via a [CWE-295: Improper Certificate Validation](https://cwe.mitre.org/data/definitions/295.html) weakness.


- [CPAN.pm](https://metacpan.org/pod/CPAN) v2.34 downloads and executes code from
  `https://cpan.org` without verifying server certs.
  ([patch](https://github.com/andk/cpanpm/commit/9c98370287f4e709924aee7c58ef21c85289a7f0))
- [GitLab::API::v4](https://metacpan.org/dist/GitLab-API-v4) v0.26 exposes API
  secrets to a network attacker.
  ([patch](https://github.com/bluefeet/GitLab-API-v4/pull/57))
- [Finance::Robinhood](https://metacpan.org/dist/Finance-Robinhood) v0.21 is
  maybe exposing API secrets and financial information to a network
  attacker. ([patch](https://github.com/sanko/Finance-Robinhood/pull/6))
- [Paws (aws-sdk-perl)](https://metacpan.org/pod/Paws) v0.44 is
  maybe exposing API secrets to a network attacker. ([patch](https://github.com/pplu/aws-sdk-perl/pull/426))
- [CloudHealth::API](https://metacpan.org/pod/CloudHealth::API) v0.01 is maybe
  exposing API secrets to a network attacker.
  ([patch](https://github.com/pplu/cloudhealth-api-perl/pull/2))

... and more. We have done a search of CPAN and generated a list of [381 potentially problematic distributions](https://hackeriet.github.io/cpan-http-tiny-overview/).


## Mitigations

Upstream for `HTTP::Tiny` has not provided a patch or mitigation. Suggestions to change the insecure default has been turned down several times over the years due to backwards compatibility concerns.

To mitigate the risk caused by the [CWE-1188: Insecure Default Initialization of Resource](https://cwe.mitre.org/data/definitions/1188.html) weakness, you have some options:

- Modify affected code using `HTTP::Tiny` and set `verify_SSL=>1`.

- Modify affected code to use a http client with secure defaults, like
  `Mojo::UserAgent` or `LWP::UserAgent`.

- Patch `HTTP::Tiny` on your system with a [proposed patch](https://salsa.debian.org/perl-team/interpreter/perl/-/commit/1490431e40e22052f75a0b3449f1f53cbd27ba92.patch) from
  Debian.

## Links

- [chansen/p5-http-tiny: RFC: Making SSL_verify  safer #152](https://github.com/chansen/p5-http-tiny/issues/152)
- [chansen/p5-http-tiny: SSL environment controls, and warn about insecure connections #151](https://github.com/chansen/p5-http-tiny/pull/151)
- [hackeriet: cpan-http-tiny-overview](https://hackeriet.github.io/cpan-http-tiny-overview/)
- [reddit: PSA: HTTP::Tiny disabled SSL verification by default!](https://www.reddit.com/r/perl/comments/111tadi/psa_httptiny_disabled_ssl_verification_by_default/)
- [NixOS/nixpkgs: perl: verify_SSL=>1 by default in HTTP::Tiny #187480](https://github.com/NixOS/nixpkgs/pull/187480)
- [Debian Bug #962407: libhttp-tiny-perl: Default to verifying SSL certificates](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=962407)
- [Debian Bug #954089: perl: Default HTTP::Tiny to verifying SSL certificates](https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=954089)
- [Debian Proposed Patch: \[PATCH\] Enable SSL by default in HTTP::Tiny](https://salsa.debian.org/perl-team/interpreter/perl/-/commit/1490431e40e22052f75a0b3449f1f53cbd27ba92.patch)
- [chansen/p5-http-tiny: verify_SSL being true by default (redux) #134](https://github.com/chansen/p5-http-tiny/issues/134)
- [chansen/p5-http-tiny: Shouldn't verify_SSL be true by default? #68](https://github.com/chansen/p5-http-tiny/issues/68)

