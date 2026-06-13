# Contributing

The most useful thing you can add is a sample log line. Without one
there is no way to test whether a new decoder or rule actually matches
what RouterOS sends. Real output from `/log print` or your syslog
receiver beats a description every time. Anonymize IPs and MAC
addresses before pasting.

## What is missing

The obvious gaps right now:

- VPN / IPsec events
- CAPsMAN wireless AP management log messages
- BGP and routing topics
- RouterOS 6.x compatibility testing
- CEF format support (RouterOS 7.18+)

## How to contribute

Fork, make your changes, open a PR. A few things that make reviews
faster: include at least one raw sample log line that your change
matches, add a corresponding rule if you are adding a decoder, and
note the RouterOS version you tested on.

Rule IDs 110001–110010 are already used. New rules should start at 110011.

## Testing

```bash
/var/ossec/bin/wazuh-logtest
```

Paste a sample log line and check that your decoder fires and
extracts the fields you expect before opening a PR.
