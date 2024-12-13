---
layout: post
title: Adventures with Fail2Ban and AbuseIPDB
author: atluxity  
category: linux
---

Securing systems exposed to the internet is a moving target, especially when dealing with brute-force attacks on authentication services. I run a Zimbra mail server on an Ubuntu 18.04 server (yes, I know it’s time to upgrade), and I decided to tackle these never-ending login attempts. Along the way, I integrated AbuseIPDB for IP reporting, configured the recidive jail for persistent offenders, and encountered a few interesting bugs and quirks worth sharing.

Fail2Ban is an open-source intrusion prevention software that monitors system logs for signs of automated attacks, such as repeated failed login attempts. When it detects suspicious activity, Fail2Ban temporarily bans the offending IP address by updating the system's firewall rules. This helps mitigate brute-force attacks and other malicious behavior without requiring constant manual intervention. It is highly configurable and can be extended to monitor various services like SSH, mail servers, and web servers.

This post details what I did, the tricks I applied, and the lessons I learned to secure my server effectively.

# Setting Up Fail2Ban for SSH and Zimbra

When I started, the fail2ban configuration already included the following files in /etc/fail2ban/jail.d/:

> root@mail1:~# ls /etc/fail2ban/jail.d/
>
> defaults-debian.conf zimbra.local

- `defaults-debian.conf`: This file contained the default sshd jail.
- `zimbra.local`: Provided by the Zimbra package, this file included configurations for the jails zimbra-smtp and zimbra-web.

I verified the active jails using:

> root@mail1:~# fail2ban-client status
>
> Status
>
>|- Number of jail:      3
> 
>`- Jail list:   sshd, zimbra-smtp, zimbra-web

For Zimbra, this config came from the install-packages of Zimbra. At least I have no recolection of setting this up. Zimbra has its [own documentation regarding this](https://blog.zimbra.com/2022/08/configuring-fail2ban-on-zimbra/).
So that is my exposed authentications, sshd, webmail and smtp.

# Adding AbuseIPDB Integration

In order to contribute back to the community I wanted to report abusive IP addresses to AbuseIPDB. As a side-effect it also provides better visibility and enriched feedback.
AbuseIPDB is a public database of abusive IP addresses that allows users to report and share information about malicious activity, such as brute-force attacks, spam, and hacking attempts. It provides tools to query and analyze IP reputation, helping system administrators and security professionals identify and block bad actors.

## Fixing the abuseipdb.conf Action

AbuseIPDB has [its own documentation for integration with Fail2Ban](https://www.abuseipdb.com/fail2ban.html), but I noticed a problem with their documentation and it seem a bit outdated. For example it includes `--tlsv1.0`. After some testing, reading [some](https://github.com/fail2ban/fail2ban/issues/2334)(1) [issues](https://github.com/fail2ban/fail2ban/issues/2590)(2) I figured that using the latest version of Fail2Ban action-file from their main branch worked perfectly.

This showcased to me that Fail2Ban is just simply running linux-commands, and it is quite flexible in that regard, so I added some `sed`s to redact hostname and domain. I split them for reasons.

> actionban = lgm=$(printf '%%.1000s\n...' "<matches>" | \                   # Limit log message to 1000 characters
> 
> sed 's,REDACTED-HOSTNAME,host,g' | \                                   # Replace hostname with 'host'
> 
> sed 's,REDACTED-DOMAINNAME,domain.tld,g'); \   # Replace domain name with 'domain.tld'
> 
> curl -sSf "https://api.abuseipdb.com/api/v2/report" \
>
> -H "Accept: application/json" \
>
> -H "Key: <abuseipdb_apikey>" \
>
> --data-urlencode "comment=$lgm" \
>
> --data-urlencode "ip=<ip>" \
>
> --data "categories=<abuseipdb_category>"

By applying this trick, I ensured reports remain useful without revealing my server’s identity. It just feels better.

# Adding the Recidive Jail for Persistent Offenders

To deal with repeat offenders, I enabled the recidive jail. This jail monitors Fail2Ban’s own log file to identify IPs that are repeatedly banned and imposes harsher penalties.

## Recidive Jail Configuration

I created a recidive jail configuration in `/etc/fail2ban/jail.d/recidive.local`:

>[recidive]
>
> enabled = true
>
> logpath  = /var/log/fail2ban.log
>
> banaction = %(banaction_allports)s
>
> bantime  = 26w
>
> findtime = 7d

## Lesson Learned: Service Restart Is More Reliable

I encountered an issue where `fail2ban-client reload` sometimes failed to apply changes, particularly when modifying or adding new jails like recidive or changing actions. It was mighty frustrating. Restarting the Fail2Ban service using systemctl worked reliably:

> systemctl restart fail2ban

# Bugs and Lessons Learned

1. AbuseIPDB Placeholder Bug: Initially, the IP placeholder (<ip>) was not being replaced in the abuseipdb.conf action file. After reviewing the latest version from the Fail2Ban GitHub repository, I realized my version had a bug. Replacing my action file with the updated version fixed the issue.
2. Outdated TLS Flag: AbuseIPDB's documentation included --tlsv1.0 in the curl command. This is insecure and unnecessary; modern curl versions negotiate TLS securely by default. Removing it resolved the issue.
3. Service Reload vs Restart: fail2ban-client reload occasionally failed to apply configuration changes. Restarting the service with systemctl ensured consistency and reliability.
4. Hostname Redaction: Adding redaction to reports before submitting them to AbuseIPDB was a simple but effective way to prevent leaking sensitive server details.

# Final Thoughts

Deploying Fail2Ban to protect SSH and Zimbra services, integrating AbuseIPDB for info-sharing, and using the recidive jail for persistent offenders I feel significantly improved the security of my server and help mitigate some of my exposure. Along the way, I learned the importance of carefully reviewing documentation, checking for software bugs, and leveraging the flexibility of Fail2Ban actions. I am now considering collecting and sharing the bans done in the recidive-jail between all my servers, because why not. Well, why not, the risk here is if one does not have a way to remote control their server "out of band", like via VM management interface etc, one might lock themself out from contacting their server. Or if one has customers or users that manages to trigger the ban then it requires manual intervention. For me, these are managed risks. Is it for you?

[This is what my reports on AbuseIPDB](https://www.abuseipdb.com/user/23612) looks like.

As always, hack the planet.

kthxby
