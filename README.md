# CorePost Install

`corepost-install` is a Bash installer and provisioning component for the CorePost demo environment.

What it does:
- downloads `corepost-preboot` runtime artifacts from GitHub raw content;
- registers a device on the CorePost server;
- writes `/etc/corepost-preboot.conf` and installs initramfs-tools hooks/scripts;
- optionally prepares a 3FA USB factor (with explicit interactive confirmation);
- installs and enables a stub `corepost-agent` systemd unit (placeholder for the real agent);
- optionally runs a demo decrypt validation by creating a demo LUKS device and invoking the preboot script.

The installer avoids hardcoding server addresses: you enter the server URL interactively or pass it via environment variables.

## Quick Start

Run as root on the target machine (or inside the Debian VM):

```bash
sudo ./install.sh install
```

To remove everything it installed:

```bash
sudo ./install.sh uninstall
```

## Notes

- The installer stores the provisioning bundle under `/var/lib/corepost-install` with `0600` permissions.
- Preboot runtime is installed into `/etc/initramfs-tools/...` and requires `update-initramfs -u`.
- The demo LUKS device is created only if you explicitly confirm it during `install`.

