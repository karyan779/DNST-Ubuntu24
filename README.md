# DNST Ubuntu 24

Standalone Ubuntu 24.04 x86_64 installer for the SSHPlus modules with SlowDNS preparation and English menu text.

## Run

```bash
apt-get update -y && apt-get upgrade -y && wget -O DNST https://raw.githubusercontent.com/karyan779/DNST-Ubuntu24/main/DNST && chmod +x DNST && ./DNST
```

The installer checks that the host is **Ubuntu 24.04.x x86_64** and must be run as root. It is not a promise that every Ubuntu release beginning with `24` works: Ubuntu 24.10 was an interim release and is already end-of-life, while Ubuntu 24.04 LTS remains the supported choice. Use Ubuntu 24.04 LTS, preferably the latest point release.

## What it changes

The upstream module set is retained and installed from its original source. Before the menu translation runs, each installed menu file is backed up under `/etc/SSHPlus/original-*`. Only visible menu/display strings are translated; function names, menu numbers, commands, paths, URLs, variables, and data are not intentionally changed.

The installer downloads the SlowDNS server binary from the maintained `powermx/dnstt` source, stores it at `/etc/SSHPlus/dns/dns-server`, and generates these files before the menu is installed:

```text
/etc/SSHPlus/dns/server.key
/etc/SSHPlus/dns/server.pub
```

It also installs the Ubuntu 24 package names `python3` and `python3-pip` rather than the obsolete `python-pip` package.

The SlowDNS server is managed by the `dnstt-ssh.service` systemd unit with automatic restart and boot enablement. UDP/53 is redirected to the local DNSTT listener on UDP/5300, and the firewall rules are added idempotently. Check it with:

```bash
systemctl status dnstt-ssh --no-pager
journalctl -u dnstt-ssh -n 50 --no-pager
ss -lunp | grep -E ':53|:5300'
```

Use the Ubuntu 24 repository URL above. The older URL `https://raw.githubusercontent.com/karyan779/DNST/main/DNST` is the separate Ubuntu 20 installer and should not be used on Ubuntu 24.

## Moving to a new VPS without changing the client public key

`server.pub` is not an independent password. It is derived from the matching private key `server.key`. Copying only `server.pub` to a new VPS cannot work. Before deleting the old VPS, transfer both files securely:

```bash
scp /root/server.key /root/server.pub root@NEW_VPS_IP:/root/
```

On the new VPS, place both files at `/root/server.key` and `/root/server.pub` **before** running this installer. The installer detects that matching pair, preserves it under `/etc/SSHPlus/dns/`, and starts the service with the same identity. The client can continue using the old `server.pub`.

Never publish or send `server.key`; it is the server's private key. If only the public key remains, generate a new pair on the new VPS and replace the public key in the client configuration.

## Warning

This software changes SSH settings, firewall rules, system services, cron/autostart files, user-management data, and files under `/bin` and `/etc`. It downloads and runs third-party upstream modules. Take a full VPS snapshot or backup and review the source before running it. SlowDNS may conflict with existing DNS, firewall, SSH, OpenVPN, or SSL services.

The script was not executed during preparation. It passed `bash -n` syntax validation.
