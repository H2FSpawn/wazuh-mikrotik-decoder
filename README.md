# wazuh-mikrotik-decoder

Wazuh decoders and detection rules for MikroTik RouterOS syslog output.

RouterOS uses a syslog format that Wazuh's built-in parsers don't understand.
Firewall drops, DHCP leases, system events — all of it lands as unstructured
noise unless you have a decoder that knows what to look for. This is that decoder.

## What it covers

Three syslog topics, with structured field extraction:

- **Firewall** — source IP, destination IP, port, interface, drop detection (rules 110001–110002)
- **DHCP** — lease events with IP, MAC address, and client hostname (rule 110003)
- **System** — general RouterOS system messages, login failure detection, brute force detection, interface down events (rules 110004–110007)

Tested on RouterOS 7.x and Wazuh 4.12–4.14.

## Installation

**Wazuh manager:**

```bash
cp decoders/mikrotik_decoder.xml /var/ossec/etc/decoders/
cp rules/mikrotik_rules.xml /var/ossec/etc/rules/
systemctl restart wazuh-manager
```

Add a syslog listener to `/var/ossec/etc/ossec.conf` if you don't have one:

```xml
<remote>
  <connection>syslog</connection>
  <port>514</port>
  <protocol>udp</protocol>
  <allowed-ips>192.168.0.0/16</allowed-ips>
</remote>
```

Restrict `allowed-ips` to your own network. UDP 514 should never be reachable from the internet.

**MikroTik:** See [docs/mikrotik-syslog-setup.md](docs/mikrotik-syslog-setup.md) for the full RouterOS configuration.

## Known limitations

The `->` character sequence that RouterOS uses as a separator in certain log
messages is a reserved character in Wazuh's regex engine and cannot be escaped.
The workaround is `[^,]+` as a field separator — that's what this decoder uses.

Parent-child decoder structures are unreliable when the parent is defined only
via `prematch`. All decoders here are standalone for that reason.

RouterOS omits the year from BSD syslog timestamps. Wazuh handles this correctly
but may log a parsing warning on first startup. This is cosmetic.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
