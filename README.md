# wazuh-mikrotik-decoder

Wazuh decoders and detection rules for MikroTik RouterOS syslog output.

RouterOS uses a syslog format that Wazuh's built-in parsers don't understand.
Firewall drops, DHCP leases, system events — all of it lands as unstructured
noise unless you have a decoder that knows what to look for. This is that decoder.

## What it covers

Three syslog topics, with structured field extraction:

- **Firewall** — source IP, source port, destination IP, and destination port; drop detection (rules 110001–110002)
- **DHCP** — assigned lease events with IP, MAC address, and client hostname (rule 110003); other DHCP events (offering, deassigned) match without field extraction
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

## Testing

Sample log lines for all supported event types are provided in `test-logs/test-logs.txt`.
Use them with `wazuh-logtest` to verify the decoder and rules are working correctly:

```bash
/var/ossec/bin/wazuh-logtest
```

Paste one line at a time and check Phase 2 for the expected decoded fields.

## Known limitations

**Firewall: TCP flag annotations prevent field extraction**
RouterOS can append TCP flags like `(SYN)` or `(ACK)` after the protocol
(e.g. `proto TCP (SYN), 1.2.3.4:54321`). When TCP flag annotations are present,
the firewall decoder does not extract IP fields. Events still match and fire
rules — only field extraction is affected.
Configure RouterOS firewall logging without TCP flag annotations to ensure
srcip, srcport, dstip, and dstport are always extracted.
See [docs/mikrotik-syslog-setup.md](docs/mikrotik-syslog-setup.md) for
the recommended RouterOS logging configuration.

**DHCP: field extraction only for assigned lease events**
The DHCP decoder extracts IP, MAC, and client hostname only from `assigned`
lease lines. Other DHCP events (`offering`, `deassigned`, etc.) match
rule 110003 without field extraction. The client hostname is stored in
`extra_data2` (not the `hostname` field, which is reserved by Wazuh for
the pre-decoding phase).

**Standalone decoders only**
Parent-child decoder structures are unreliable when the parent is defined only
via `prematch`. All decoders here are standalone for that reason.

**RouterOS syslog timestamp**
RouterOS omits the year from BSD syslog timestamps. Wazuh handles this correctly
but may log a parsing warning on first startup. This is cosmetic.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT — see [LICENSE](LICENSE).
