---
layout: post
title: "Perl HTTP::Tiny has insecure TLS default, affecting CPAN.pm and other modules"
author: sgo
category: security
---

**UPDATE 2023-06-12:** [v0.083-TRIAL](https://metacpan.org/release/DAGOLDEN/HTTP-Tiny-0.083-TRIAL/) has been released with a fix.

\[CVE-2023-31486\] [HTTP::Tiny](https://metacpan.org/pod/HTTP::Tiny) v0.082, is a http client included in
Perl (since v5.13.9) and also a standalone CPAN module. It [does not verify TLS
certificates by default](https://metacpan.org/pod/HTTP::Tiny#SSL-SUPPORT) requiring users to opt-in with the `verify_SSL=>1` flag to verify the identity of the HTTPS server they are communicating with.

The module is used by
[many](https://hackeriet.github.io/cpan-http-tiny-overview/) distributions on
CPAN, and likely other open source and proprietary software.

> **NOTE:** This post summarizes security problems caused by the insecure default
> and how it affects code relying on it for https. For a
> discussion on how this is being addressed upstream, please see
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

- \[CVE-2023-31484\] [CPAN.pm](https://metacpan.org/pod/CPAN) v2.34 downloads and executes code from
  `https://cpan.org` without verifying server certs. Fixed in [v2.35-TRIAL](https://metacpan.org/release/ANDK/CPAN-2.35-TRIAL).
  ([patch](https://github.com/andk/cpanpm/commit/9c98370287f4e709924aee7c58ef21c85289a7f0))
- \[CVE-2023-31485\] [GitLab::API::v4](https://metacpan.org/dist/GitLab-API-v4) v0.26 exposes API
  secrets to a network attacker. Fixed in [v0.27](https://metacpan.org/release/BLUEFEET/GitLab-API-v4-0.27)
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

~~Upstream for `HTTP::Tiny` has not provided a patch or mitigation. Suggestions to change the insecure default has been turned down several times over the years due to backwards compatibility concerns~~.

Upstream for `HTTP::Tiny` has [merged a patch](https://github.com/chansen/p5-http-tiny/pull/153) that changes the insecure default from `0` to `1`. It is available in [HTTP::Tiny v0.083-TRIAL](https://metacpan.org/release/DAGOLDEN/HTTP-Tiny-0.083-TRIAL/) on CPAN, and will also be included in perl `v5.38.0`. 

> An escape hatch environment variable `PERL_HTTP_TINY_SSL_INSECURE_BY_DEFAULT=1` has been provided for users who need to restore the previous insecure default after updating.

For additional information, please see the upstream discussion in [RFC: Making SSL_verify safer](https://github.com/chansen/p5-http-tiny/issues/152).

To mitigate the risk caused by the [CWE-1188: Insecure Default Initialization of Resource](https://cwe.mitre.org/data/definitions/1188.html) weakness, you have some options:

- Ensure that `HTTP::Tiny` in your include path is [v0.083-TRIAL](https://metacpan.org/release/DAGOLDEN/HTTP-Tiny-0.083-TRIAL/) or newer.

- Modify affected code using `HTTP::Tiny` and set `verify_SSL=>1`.

- Modify affected code to use a http client with secure defaults, like
  `Mojo::UserAgent` or `LWP::UserAgent`.

- Patch `HTTP::Tiny` on your system with [a patch](https://github.com/chansen/p5-http-tiny/commit/77f557ef84698efeb6eed04e4a9704eaf85b741d.patch) that changes the default to `verify_SSL=>1`.

> It's recommended that users update [Mozilla::CA](https://metacpan.org/pod/Mozilla::CA) as well, since `HTTP::Tiny` defaults to trusting certificates provided by that module. Alternatively the environment variable `SSL_CERT_FILE` can be set to point to the system trust store.

## Links
- [chansen/p5-http-tiny: Change verify_SSL default to 1, add ENV var to enable insecure default \[CVE-2023-31486\] #153](https://github.com/chansen/p5-http-tiny/pull/153)
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

## Changes
- 2023-04-18: Add reference to fixed CPAN.pm v2.35-TRIAL
- 2023-04-29: Add CVE identifiers CVE-2023-31484, CVE-2023-31485, CVE-2023-31486
- 2023-06-12: Add references to fix applied to HTTP::Tiny, GitLab::API::v4, and suggestion to update Mozilla::CA

