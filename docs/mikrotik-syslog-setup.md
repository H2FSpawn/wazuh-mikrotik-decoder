# MikroTik Syslog Setup

This decoder expects syslog in BSD format (RFC 3164), which is what
RouterOS sends by default. Here is how to get the right data flowing.

## RouterOS configuration

Open Winbox and go to System → Logging.

First, create a new logging action under the Actions tab:

| Field | Value |
|-------|-------|
| Name | wazuh |
| Type | remote |
| Remote Address | your Wazuh manager IP |
| Remote Port | 514 |
| Syslog Facility | daemon |
| BSD Syslog | yes |

Then add logging rules under the Rules tab, one per topic:

| Topics | Action |
|--------|--------|
| firewall | wazuh |
| dhcp | wazuh |
| system | wazuh |

The same via CLI:

```bash
/system logging action add name=wazuh target=remote \
  remote=<wazuh-manager-ip> remote-port=514 bsd-syslog=yes \
  syslog-facility=daemon

/system logging add topics=firewall action=wazuh
/system logging add topics=dhcp action=wazuh
/system logging add topics=system action=wazuh
```

## Wazuh manager configuration

Add a syslog listener to `/var/ossec/etc/ossec.conf` if you do not
have one already:

```xml
<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>udp</protocol>
  <allowed-ips>192.168.0.0/16</allowed-ips>
</remote>
```

Restrict `allowed-ips` to your own network. UDP 514 should never be
reachable from the internet.

Restart the manager after any config change:

```bash
systemctl restart wazuh-manager
```

## What the log messages look like

Firewall drop:
```
Mar 16 22:14:05 router firewall,info forward: in:ether1 out:bridge
src-mac aa:bb:cc:dd:ee:ff, proto TCP, 1.2.3.4:54312->192.168.10.5:22,
len 60
```

DHCP lease:
```
Mar 16 22:10:01 router dhcp,info dhcp1 assigned 192.168.10.55 to
aa:bb:cc:dd:ee:ff (mylaptop)
```

System login failure:
```
Mar 16 22:11:43 router system,info login failure for user admin from
1.2.3.4 via ssh
```

## Testing the decoder

The fastest way to verify that a log line matches your decoder:

```bash
/var/ossec/bin/wazuh-logtest
```

Paste a raw log line and check that the decoder name and extracted
fields look right. If the decoder does not fire, the prematch pattern
is usually where to look first.

## Known issues

RouterOS omits the year from BSD syslog timestamps. Wazuh handles
this correctly but may log a parsing warning on first startup. This
is cosmetic and does not affect functionality.

Alert rules 110005 and 110006 (login failures and brute force) may
fire on legitimate automation if you have scripts logging into the
router frequently. Adjust the `<frequency>` and `<timeframe>` values
in the rules file to match your environment.
