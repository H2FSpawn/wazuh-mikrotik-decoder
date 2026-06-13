# Security Policy

## Reporting a vulnerability

If you find a security issue in this project, please do not open a
public GitHub issue. Send a mail to sandro@puccios.de with a
description of the problem. I will respond within a few days and
coordinate a fix before any public disclosure.

## Scope

This project consists of Wazuh decoder and rule XML files. There is
no executable code. The main security consideration is correct
handling of log data — a misconfigured decoder could miss attacks
or generate excessive noise, but cannot itself be exploited.

## Network exposure

The syslog listener (UDP 514) should never be reachable from the
internet. Restrict `allowed-ips` in your Wazuh config to your
internal network only.
