# Changelog

All notable changes to this project will be documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/).

## [1.1.0] - 2026-06-14

### Added

- `test-logs/test-logs.txt` — anonymized sample log lines for all supported
  event types, ready to paste into `wazuh-logtest` for verification

### Fixed

- Firewall decoder now extracts `dstip` and `dstport` in addition to `srcip`
  and `srcport`. The `->` separator between source and destination in RouterOS
  firewall logs was previously a blocker (reserved operator in Wazuh's regex
  engine). Resolved by switching to `pcre2` regex type, which supports matching
  the `-` and `>` characters individually via `\p\p`.
- DHCP decoder: client hostname is now stored in `extra_data2` instead of
  `hostname`. The `hostname` field is reserved by Wazuh's pre-decoding phase
  and was being overwritten with the router's hostname, making the field useless.
- Interface-down rule (110007) now uses `if_sid` to chain from the base system
  rule (110004) instead of a standalone `match`. The previous approach did not
  fire reliably because Wazuh stops rule evaluation at the first match.

### Changed

- Firewall decoder uses `type="pcre2"` for the `regex` element
- Rule descriptions for 110001 and 110002 no longer include IP field variables
  in the description text, as fields are not always present (TCP flag events)
- Removed the known limitation about `->` being unescapable — no longer applies

## [1.0.0] - 2026-06-13

### Added

- Initial release
- Firewall decoder: extracts `srcip` and `srcport` via `proto` keyword anchor
- DHCP decoder: extracts assigned IP, MAC address, and client hostname
- System decoder: matches general RouterOS system messages
- Rules 110001–110007: firewall events, DHCP lease, system events, login
  failure, brute force detection, interface down
- `docs/mikrotik-syslog-setup.md`: RouterOS syslog configuration guide
