---
layout: post
title: Demystifying Telco Connections: SIP, SIMs, and the Network in Between
author: atluxity
category: security
---

We at Hackeriet had the great pleasure of hosting a workshop by Harrison Sand. Harrison has been looking at telco security from the side of phishing groups, SIM farms, Wi-Fi Calling and packet captures.

Think of this post as a map. The point is to connect the pieces well enough that a responsible researcher that can read a pcap can recognize the trust boundaries, and know what needs authorization before touching anything live.

Modern mobile calling is several ordinary systems stacked together: DNS, IPsec, SIM-backed authentication, carrier profiles, SIP, and operator policy.

<!-- excerpt -->

## Start With Fraud

The research started with phishing groups. The infrastructure behind messages that look like Autopass, Posten, bank warnings, and delivery notices.

That led to SIM farms. A SIM farm is a rack of SIM cards attached to hardware that can send messages through a web interface or API. Europol has published examples from cybercrime-as-a-service cases. The hardware is modems, SIM slots, management software, and enough automation to send at scale.

SIM farms explain the attacker incentive. Phone numbers still matter. Even when scams move to iMessage or RCS, a phone number is often part of account creation, reputation, and delivery.

Wi-Fi Calling adds another question: if a phone can act like a normal subscriber over Wi-Fi, what does that path look like?

## Two Research Paths

There are two paths through the material. The first path teaches observation. The second teaches architecture. It can be smart to try to keep them separated mentaly.

One path starts with your phone. Plug in your own device, capture your own traffic, and look at what the handset and network exchange. On iPhone, Remote Virtual Interface capture can expose several interfaces. One of those views can show SIP traffic that would otherwise feel hidden behind the modem and IPsec. Android varies by device, firmware, modem, and access level.

The other path starts with the subscriber connection. A Wi-Fi Calling client has to find the operator's ePDG, authenticate using SIM-backed material, receive inner connectivity, and register to IMS over SIP.

## The Subscriber Path

The rough chain looks like this:

```text
SIM / USIM
   -> ePDG discovery
   -> IKEv2 / IPsec
   -> SIM-backed AKA authentication
   -> inner IPv6 connectivity
   -> P-CSCF / IMS SIP edge
   -> SIP REGISTER
   -> SIP AKA / digest challenge
   -> subscriber session
```

Terminology note: people often say "SIM" for the physical card and everything on it. More precisely, modern mobile networks use a USIM, the Universal Subscriber Identity Module, for subscriber identity and AKA, Authentication and Key Agreement. Some IMS setups also involve ISIM, the IP Multimedia Services Identity Module, for SIP/IMS identities.

The phone first needs an ePDG, the Evolved Packet Data Gateway. Many operators use a standardized DNS name based on MCC and MNC, the mobile country code and mobile network code. Some do not. Those operators may rely on carrier profiles shipped through iOS or Android.

After discovery, the device establishes IKEv2/IPsec to the operator. The SIM participates in AKA-style authentication. Think of the SIM as a small cryptographic oracle: software sends a challenge, the SIM returns a response, and the private key material stays on the card.

If the ePDG authentication succeeds, the client gets inner network connectivity, often IPv6, plus enough configuration to reach the IMS, IP Multimedia Subsystem, SIP edge. The endpoint you talk to is usually a P-CSCF, the Proxy-Call Session Control Function. Calling it "the SIP server" is close enough for a hallway conversation, but P-CSCF is the more precise term.

SIP registration comes next. The client sends `REGISTER`, receives a challenge, consults SIM/USIM-backed material, and sends an authenticated response. Once registration succeeds, the subscriber can maintain state for calls and messages under the operator's policy.

That is the architectural story Harrison walked through. ePDG gets you onto the operator path. SIM-backed AKA proves the subscription. SIP/IMS handles the call and messaging signaling.

## Carrier Profiles Count

Carrier profiles are part of the system.

They can define Wi-Fi Calling domains, IMS settings, radio options, display names, and operator-specific behavior. On iOS, firmware and carrier bundles can be inspected with tools such as `ipsw`. That does not require touching subscribers or sending traffic into the operator network.

This is a good passive research target. You can learn which operators use standard ePDG DNS, which ship custom domains, and what assumptions the phone carries before it sends a packet.

## SIP Makes The Policy Visible

SIP looks familiar if you have spent time with HTTP. It has methods, headers, identity fields, user-agent-style strings, routing data, and provisional responses such as ringing.

IMS uses SIP with 3GPP extensions. Headers can carry access-network type, cell identity, Wi-Fi access information, preferred identity, and network-provided values. Some of those fields make sense inside a trusted operator domain. They become a problem when they cross to the wrong party.

The Telia case showed this clearly. During his research, Harrison found that Telia call signaling could expose the base station of the person being called. NRK later documented the issue and showed that affected users could be located to roughly 100 to 200 meters in Oslo. Telia fixed it before publication.

The Telia case matters because SIP headers sit on trust boundaries. A proxy has to decide what to preserve, strip, rewrite, or mark as network-provided. If that policy is wrong, a diagnostic field becomes location data.

## Caller Identity Needs Enforcement

SIP has several identity-like fields. A call can contain caller identity, preferred identity, authorization identity, subscriber identity, and network-provided identity. Some of this exists for legitimate reasons. A company may want employees to call out from the main switchboard number.

That only works safely if the network checks the requested identity against what the subscriber or organization is allowed to use.

A recent Telia press release about a closed spoofing weakness. The same boundary problem shows up there: domestic networks and inter-operator paths can trust each other in ways that anti-spoofing systems built for foreign ingress do not cover.

## Observing Your Own Phone

Device-side capture is the safer entry point.

With iPhone RVI-style capture, Wireshark can show SIP call setup from your own device. You may see access-network headers, provisional responses, user-agent behavior, and the difference between outer encrypted transport and inner signaling.

Remember that this is observing your phone, not the phone network.

It still changes how the system feels. A mobile call stops being a sealed black box. You can see what the handset says, what the network returns, and which metadata appears during call setup.

For a first lab, that is a good start.

## Why The Work Is Awkward

Telco research crosses too many layers for one neat mental model.

Some behavior is in the SIM. Some is in the modem. Some is in the OS. Some is in carrier profiles. Some is in IPsec negotiation. Some is in IMS/SIP. Some is in vendor equipment from Ericsson, Nokia, Cisco, Alcatel-Lucent, or whoever sold the operator a box years ago.

The disclosure path is also harder than in web security. Operators run emergency-call infrastructure. Lawful intercept exists. Regulations limit what can be shown to outside researchers. The operator may not control the code that implements the broken behavior.

Another point that should worry operators: AI lowers the cost of reading huge protocol documents and building enough glue code to test an idea. More people will be able to ask telco questions that previously required years of background.

The answer is to make responsible research possible before irresponsible research finds the same edges, not to hope nobody looks.

Thanks to Harrison Sand for the research and workshop this post is based on.

## Further Reading

- NRK on the Telia location issue: https://www.nrk.no/norge/sikkerhetshull-avslorte-telia-kunders-posisjon-1.17842282
- Telia / NTB on the closed spoofing issue: https://kommunikasjon.ntb.no/pressemelding/18924675/telia-har-lukket-spoofing-hull-i-mobilnettet
- Europol on cybercrime-as-a-service and SIM-farm infrastructure: https://www.europol.europa.eu/media-press/newsroom/news/cybercrime-service-takedown-7-arrested
- iOS Remote Virtual Interface capture tooling: https://github.com/gh2o/rvi_capture
- iOS firmware and carrier-profile tooling: https://github.com/blacktop/ipsw
- RFC 7315, SIP private header extensions for 3GPP: https://www.rfc-editor.org/rfc/rfc7315
- 3GPP TS 23.003, ePDG naming and operator identifiers.
- strongSwan documentation for IKEv2/IPsec and EAP-AKA-related pieces: https://docs.strongswan.org/
- Google Geolocation API docs, for context on why cell IDs can become location data: https://developers.google.com/maps/documentation/geolocation/requests-geolocation
