# Security Policy

## Reporting a vulnerability

Please use GitHub's private vulnerability reporting to flag security
issues. Go to the Security tab of this repository and click
"Report a vulnerability" — this keeps the report private until a fix
is in place.

I will respond within a few days and coordinate disclosure from there.

## Scope

This project consists of Wazuh decoder and rule XML files. There is
no executable code. The main security consideration is correct
handling of log data — a misconfigured decoder could miss attacks
or generate excessive noise, but cannot itself be exploited.

## Network exposure

The syslog listener (UDP 514) should never be reachable from the
internet. Restrict `allowed-ips` in your Wazuh config to your
internal network only.
