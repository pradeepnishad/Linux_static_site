# Project 1: Nginx Web Server on a VirtualBox VM

Install Nginx on an Ubuntu VM, host a static site, configure virtual hosts, and enable HTTPS with Let's Encrypt.

**Status:** Phase 1 (VM + network + SSH) complete. Nginx, virtual hosts and HTTPS are in progress.

---

## Table of Contents

1. [Overview](#overview)
2. [Environment](#environment)
3. [Phase 1: VM Network Setup (Bridged Adapter)](#phase-1-vm-network-setup-bridged-adapter)
4. [Phase 2: SSH Access from Host to VM](#phase-2-ssh-access-from-host-to-vm)
5. [Phase 3: SSH Key-Based Authentication](#phase-3-ssh-key-based-authentication)
6. [Next Phases (Planned)](#next-phases-planned)
7. [Troubleshooting Log](#troubleshooting-log)
8. [Key Learnings](#key-learnings)

---

## Overview

| Item | Detail |
|------|--------|
| Goal | Host a static website on Nginx with virtual hosts and HTTPS |
| Hypervisor | Oracle VirtualBox |
| Guest OS | Ubuntu Server (hostname: `webserver`) |
| Host OS | Ubuntu Desktop |
| Admin user (VM) | `webadmin` |
| Network mode | Bridged Adapter |

---

## Environment

| Machine | Interface | IP Address | Notes |
|---------|-----------|------------|-------|
| Host | `enx607d09a4f5c7` (USB Ethernet) | `192.168.0.101/24` | Active connection to the router |
| VM (`webserver`) | `enp0s3` | `192.168.0.118/24` | Assigned by DHCP |
| Router / Gateway | n/a | `192.168.0.1` | Default route |

VM MAC address: `08:00:27:f7:e8:56`

---

## Phase 1: VM Network Setup (Bridged Adapter)

### Why Bridged?

By default VirtualBox uses **NAT**, which hides the VM behind the host (the VM gets `10.0.2.15`). A **Bridged Adapter** puts the VM directly on the home network with its own IP, so the host (and other devices) can reach it and SSH into it.

### Steps

1. Shut down the VM completely.
2. Open **Settings > Network > Adapter 1**.
3. Tick **Enable Network Adapter**.
4. Set **Attached to** to **Bridged Adapter**.
5. Find the host's active interface:
   ```bash
   ip route | grep default
   ```
   Output on the host:
   ```
   default via 192.168.0.1 dev enx607d09a4f5c7 proto dhcp src 192.168.0.101 metric 100
   ```
   The name after `dev` is the interface to bridge to.
6. In the **Name** dropdown choose `enx607d09a4f5c7`.
7. Under **Advanced**, set **Adapter Type** to **Intel PRO/1000 MT Desktop (82540EM)**.
   (Not `usbnet`; that option is a virtual card type, unrelated to the host's USB adapter.)
8. Click OK and start the VM.

### Verification

Inside the VM:

```bash
ip a
```

Result: `enp0s3` is `UP` with `inet 192.168.0.118/24`, in the same subnet as the host. This confirms the bridge works.

> 📸 Screenshot: `ip a` output on the VM
> 📸 Screenshot: `ip a` output on the host

---

## Phase 2: SSH Access from Host to VM

### Install and enable the SSH server (inside the VM)

```bash
sudo apt update
sudo apt install openssh-server -y
sudo systemctl enable --now ssh
systemctl status ssh
```

### Connect from the host

```bash
ssh webadmin@192.168.0.118
```

Accept the host fingerprint (`yes`) on first connection and enter the `webadmin` password.

### Connectivity checks

```bash
ping -c 3 192.168.0.118          # from the host
sudo ufw status                  # in the VM, if the firewall is enabled
sudo ufw allow OpenSSH           # in the VM, if SSH is blocked
```

---

## Phase 3: SSH Key-Based Authentication

All commands in this phase run on the **host**.

### 1. Generate a key pair

```bash
ssh-keygen -t ed25519
```

Press Enter to accept the default path (`~/.ssh/id_ed25519`). A passphrase is optional.

- `~/.ssh/id_ed25519` is the **private key** (stays on the host, never shared)
- `~/.ssh/id_ed25519.pub` is the **public key** (copied to the VM)

### 2. Copy the public key to the VM

```bash
ssh-copy-id webadmin@192.168.0.118
```

This appends the public key to `~/.ssh/authorized_keys` on the VM. The password is needed one last time here.

### 3. Test passwordless login

```bash
ssh webadmin@192.168.0.118
```

Expected: login without a password prompt.

### 4. Create an SSH shortcut (optional)

Edit `~/.ssh/config` on the host:

```
Host webvm
    HostName 192.168.0.118
    User webadmin
    IdentityFile ~/.ssh/id_ed25519
```

Now connect with:

```bash
ssh webvm
```

### Note on the VM's IP address

The VM's IP comes from DHCP and can change after a reboot or lease expiry. To keep it stable, either reserve `192.168.0.118` for MAC `08:00:27:f7:e8:56` in the router, or configure a static IP with netplan. If the IP changes, update `HostName` in `~/.ssh/config`.

---

## Next Phases (Planned)

These are the remaining steps for the project; update this section with commands and screenshots as each is completed.

### Phase 4: Install Nginx

```bash
sudo apt install nginx -y
sudo systemctl enable --now nginx
systemctl status nginx
curl -I http://localhost
```

### Phase 5: Static site and first virtual host

- Create `/var/www/<domain>/html/index.html`
- Create `/etc/nginx/sites-available/<domain>` with a `server` block
- Symlink into `sites-enabled`, then `sudo nginx -t` and `sudo systemctl reload nginx`

### Phase 6: Second virtual host

- Separate web root and server block (for example a `blog` subdomain)
- Confirm each hostname serves different content

### Phase 7: HTTPS with Let's Encrypt

```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d <domain> -d www.<domain>
sudo certbot renew --dry-run
```

> **Important:** Let's Encrypt must verify a real domain over the internet. A VM on a home network is not reachable by default, so choose one of:
> - Use the **DNS-01 challenge** with a real domain
> - Practice with a **self-signed certificate** locally, then repeat the Let's Encrypt step on a cloud VM

### Phase 8: Firewall and verification

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

Check logs in `/var/log/nginx/access.log` and `/var/log/nginx/error.log`.

---

## Troubleshooting Log

| Problem | Cause | Fix |
|---------|-------|-----|
| Unsure which dropdown to use for the Wi-Fi/Ethernet card | The visible list was **Adapter Type**, not the host adapter | Set **Attached to** to Bridged Adapter first; the **Name** dropdown then lists host interfaces |
| Unknown host interface name | USB Ethernet adapters appear as `enx<MAC>` | Found with `ip route \| grep default` |
| VM shows `10.0.2.15` | Still on NAT | Re-check **Attached to: Bridged Adapter** and restart the VM |
| SSH "Connection refused" | `openssh-server` not installed or not running | Install it and run `sudo systemctl enable --now ssh` |
| SSH "Connection timed out" | Firewall or wrong IP | Check `ufw status` and `ping` the VM |

Add new issues here as they come up, with the exact error message and the fix.

---

## Key Learnings

- NAT vs Bridged networking in VirtualBox, and why bridged is better for server labs
- Reading `ip a` and `ip route` output to identify interfaces, IPs and gateways
- The difference between a virtual adapter type and a host network interface
- SSH key pairs: private key stays local, public key goes to `authorized_keys`
- DHCP-assigned IPs can change, so servers need a reservation or static IP
- Let's Encrypt needs public DNS and port 80/443 reachability to issue certificates
