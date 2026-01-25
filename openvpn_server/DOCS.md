# OpenVPN Server Add-on - Complete Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [Installation Guide](#installation-guide)
3. [Configuration Options](#configuration-options)
4. [Certificate Management](#certificate-management)
5. [Client Setup](#client-setup)
6. [Advanced Usage](#advanced-usage)
7. [Troubleshooting](#troubleshooting)

---

## Introduction

This add-on transforms your Home Assistant into a full-featured OpenVPN server, enabling secure remote access to your home network and Home Assistant instance from anywhere in the world.

### Features

- **Complete PKI Management**: Built-in EasyRSA for certificate authority
- **Easy Client Generation**: Simple commands to create client configurations
- **Certificate Revocation**: Revoke compromised certificates via CRL
- **Production Ready**: Industry-standard security settings
- **Automatic Configuration**: Generates optimal server and client configs
- **Monitoring**: Real-time connection status and logging

---

## Installation Guide

### Step 1: Add Repository

1. Open Home Assistant
2. Navigate to **Supervisor** → **Add-on Store**
3. Click **⋮** (three dots menu) → **Repositories**
4. Add URL: `https://github.com/laurentboutet/ha-openvpn-server`
5. Click **Add**

### Step 2: Install Add-on

1. Refresh add-on store (pull down to refresh)
2. Find **OpenVPN Server** in the list
3. Click on it → Click **Install**
4. Wait for installation (may take 2-5 minutes)

### Step 3: Initial Configuration

Before starting, configure these essential settings:

```yaml
routes:
  - "192.168.1.0 255.255.255.0"  # CHANGE TO YOUR LAN SUBNET!
```

**How to find your LAN subnet:**

- Check your router settings (typically 192.168.0.0/24 or 192.168.1.0/24)
- Or run: `ip route | grep default`

### Step 4: Start Add-on

- Click **Start**
- Monitor **Log** tab for "PKI initialization complete"
- Enable **Start on boot**
- Enable **Watchdog** for auto-restart on failure

---

## Configuration Options

### Network Settings

| Option | Default | Description |
|--------|---------|-------------|
| proto | udp | Protocol (udp recommended for performance, tcp for restrictive networks) |
| port | 1194 | OpenVPN listening port (standard is 1194) |
| server_subnet | 10.8.0.0 | VPN IP pool starting address |
| server_netmask | 255.255.255.0 | VPN subnet mask (255.255.255.0 = /24 = 254 clients) |

### DNS Configuration

```yaml
dns_servers:
  - "1.1.1.1"      # Cloudflare DNS
  - "1.0.0.1"      # Cloudflare DNS backup
  - "192.168.1.1"  # Or your local DNS/router
```

**Options:**

- `1.1.1.1`, `1.0.0.1` - Cloudflare (fast, privacy-focused)
- `8.8.8.8`, `8.8.4.4` - Google DNS
- `192.168.1.1` - Your router (for local DNS resolution)

### Route Configuration

Routes define which networks clients can access:

```yaml
routes:
  - "192.168.1.0 255.255.255.0"    # Your home LAN
  - "192.168.2.0 255.255.255.0"    # Additional subnet (if any)
```

**Important:** Match your actual network topology!

### Client Settings

| Option | Default | Description |
|--------|---------|-------------|
| client_to_client | true | Allow VPN clients to communicate with each other |
| duplicate_cn | false | Allow same certificate to connect multiple times (NOT recommended) |
| max_clients | 10 | Maximum simultaneous connections |

### Security Settings

| Option | Default | Description |
|--------|---------|-------------|
| cipher | AES-256-GCM | Encryption algorithm (AES-256-GCM, AES-128-GCM, AES-256-CBC) |
| auth | SHA256 | HMAC authentication (SHA256, SHA384, SHA512) |
| log_level | 3 | Verbosity (0=silent, 3=normal, 9=maximum debug) |

**Recommended:** Keep defaults unless you have specific requirements.

---

## Certificate Management

### Understanding PKI

The add-on creates a complete PKI (Public Key Infrastructure):

```
/data/openvpn/pki/
├── ca.crt              # Certificate Authority (public)
├── private/
│   ├── ca.key          # CA private key
│   ├── server.key      # Server private key
│   └── client1.key     # Client private keys
├── issued/
│   ├── server.crt      # Server certificate
│   └── client1.crt     # Client certificates
├── dh.pem              # Diffie-Hellman parameters
├── ta.key              # TLS auth key
└── crl.pem             # Certificate Revocation List
```

### Creating Clients

**Access Add-on Terminal:**

1. Click on OpenVPN Server add-on
2. Click **Terminal** tab (top icon bar)

**Generate client certificate:**

```bash
openvpn-client-add clientname
```

**Naming conventions:**

- Use descriptive names: `john-phone`, `laptop-work`, `tablet-kids`
- Only letters, numbers, hyphens, underscores
- No spaces or special characters

**Output:**

```
✓ Client 'john-phone' created successfully!
✓ Configuration file: /share/john-phone.ovpn
```

### Downloading Client Files

#### Method 1: File Editor Add-on

1. Install "File Editor" add-on if not present
2. Navigate to `/share/` directory
3. Open `clientname.ovpn`
4. Click **Download** icon

#### Method 2: Samba Share

1. Install "Samba share" add-on
2. Access `\\homeassistant\share` from file explorer
3. Copy `.ovpn` files

#### Method 3: SSH/SFTP

```bash
scp root@homeassistant:/share/client.ovpn ./
```

### Editing Client Configuration

Before distributing, update the remote line with your public IP or DDNS:

```
# Change this line:
remote YOUR_HOME_ASSISTANT_IP 1194

# To your actual address:
remote myhome.duckdns.org 1194
# or
remote 203.0.113.45 1194
```

### Listing Clients

```bash
openvpn-client-list
```

**Output example:**

```
OpenVPN Clients:
================================================
  ✓ john-phone (Active)
  ✓ laptop-work (Active)
  ✗ old-tablet (Revoked)
================================================
Currently Connected Clients:
================================================
  → john-phone (10.8.0.2) - Connected since: 2026-01-24 14:30:15
================================================
```

### Revoking Certificates

If a device is lost/stolen or certificate compromised:

```bash
openvpn-client-revoke clientname
```

**Important:**

- Restart add-on after revocation
- Client will be immediately disconnected on next certificate check
- `.ovpn` file is automatically deleted from `/share/`

### Backup PKI

**Critical:** Backup your PKI regularly!

```bash
# Create backup
tar -czf /share/openvpn-backup-$(date +%Y%m%d).tar.gz -C /data openvpn/

# Restore backup
tar -xzf /share/openvpn-backup-YYYYMMDD.tar.gz -C /data/
```

**Automated backup:**

- Use Home Assistant's built-in backup (includes `/data`)
- Or setup external backup with Synology/rsync/rclone

---

## Client Setup

### Android & iOS

**OpenVPN Connect (Official):**

1. Install from [Google Play](https://play.google.com/store/apps/details?id=net.openvpn.openvpn) or [App Store](https://apps.apple.com/app/id590379981)
2. Transfer `.ovpn` file to device (email, cloud, USB)
3. Open file → Select "OpenVPN"
4. Click **Add** → **Import** → Select `.ovpn`
5. Toggle connection switch

**Tips:**

- Enable "Seamless tunnel" for auto-reconnect
- Set "Connection timeout" to 60+ seconds for mobile networks
- Enable battery optimization exceptions

### Windows

**OpenVPN GUI:**

1. Download from [OpenVPN.net](https://openvpn.net/community-downloads/)
2. Install with default settings
3. Copy `.ovpn` file to: `C:\Program Files\OpenVPN\config\`
4. Run OpenVPN GUI (may need admin rights)
5. Right-click tray icon → Select profile → Connect

**OpenVPN Connect (Alternative):**

1. Download from Microsoft Store
2. Import `.ovpn` through UI

### macOS

**Tunnelblick (Recommended):**

1. Download from [Tunnelblick.net](https://tunnelblick.net/)
2. Install and grant necessary permissions
3. Drag `.ovpn` file onto Tunnelblick icon
4. Click **Connect**

**OpenVPN Connect:**

1. Download from Mac App Store
2. Import `.ovpn` file

### Linux

**Command line:**

```bash
# Install OpenVPN
sudo apt install openvpn  # Debian/Ubuntu
sudo dnf install openvpn  # Fedora/RHEL

# Connect
sudo openvpn --config client.ovpn

# Or run as daemon
sudo openvpn --config client.ovpn --daemon
```

**NetworkManager (GUI):**

```bash
# Install plugin
sudo apt install network-manager-openvpn-gnome

# Import via Settings → Network → VPN → Import
```

### Verify Connection

Once connected, verify:

**Check IP assignment:**

```bash
ip addr show tun0  # Should show 10.8.0.x
```

**Ping Home Assistant:**

```bash
ping 192.168.1.100  # Your HA IP
```

**Access Home Assistant:**

1. Open browser: `http://192.168.1.100:8123`
2. Should load without external URL/Nabu Casa

**Check routing:**

```bash
ip route  # Should show routes via tun0
```

---

## Advanced Usage

### Split-Tunnel vs Full-Tunnel

**Current:** Full-tunnel (all traffic through VPN)

**Enable split-tunnel** (only home traffic through VPN):

Edit `/data/openvpn/server.conf`, remove:

```
push "redirect-gateway def1"
```

Clients will only route routes through VPN, keeping internet direct.

### Static IP for Clients

Create `/data/openvpn/ccd/clientname`:

```
ifconfig-push 10.8.0.100 255.255.255.0
```

### Custom DNS per Client

In `/data/openvpn/ccd/clientname`:

```
push "dhcp-option DNS 192.168.1.10"
```

### IPv6 Support

Add to `config.yaml`:

```yaml
server_subnet: "10.8.0.0"
server_subnet_v6: "fd00:8::/64"
```

Edit `/data/openvpn/server.conf`:

```
server-ipv6 fd00:8::/64
push "route-ipv6 2000::/3"
```

### Performance Tuning

For high-speed connections, edit `/data/openvpn/server.conf`:

```
# Larger buffers
sndbuf 524288
rcvbuf 524288
push "sndbuf 524288"
push "rcvbuf 524288"

# Faster cipher (if hardware supports AES-NI)
cipher AES-128-GCM

# Disable compression (better for modern CPUs)
compress lz4-v2
push "compress lz4-v2"
```

### Monitoring & Statistics

**Real-time status:**

```bash
cat /data/openvpn/openvpn-status.log
```

**Connection log:**

```bash
tail -f /data/openvpn/openvpn.log
```

**Connection history:**

```bash
cat /data/openvpn/ipp.txt  # IP persistence
```

---

## Troubleshooting

### Cannot Connect to VPN

**Symptom:** Client shows "connection timeout"

**Solutions:**

**Verify port forwarding:**

1. Log into router
2. Confirm UDP 1194 → HA IP
3. Test: `nmap -sU -p 1194 YOUR_PUBLIC_IP`

**Check public IP:**

1. Visit whatismyip.com
2. Update `.ovpn` remote line if changed

**Firewall:**

- Check router firewall rules
- Disable temporarily to test

**ISP restrictions:**

- Some ISPs block VPN ports
- Try `proto tcp` and `port 443` (HTTPS port)

### Connects but No LAN Access

**Symptom:** VPN connects (10.8.0.x IP) but cannot reach 192.168.x.x

**Solutions:**

**Verify routes:**

- Check config routes matches LAN
- Client should show route: `ip route` (Linux) or `route print` (Windows)

**IP forwarding:**

```bash
# On HA host
cat /proc/sys/net/ipv4/ip_forward  # Should be 1
```

**iptables rules:**

```bash
# Check NAT
iptables -t nat -L POSTROUTING -v
# Should show MASQUERADE rule
```

**Home Assistant firewall:**

- If running firewall on HA host, allow `tun0` interface

### Slow Performance

**Solutions:**

**Check CPU usage:**

- High CPU → Consider AES-128-GCM cipher
- Modern CPUs with AES-NI handle AES-256 well

**Network MTU:**

Add to client `.ovpn`:

```
mssfix 1400
```

**Connection quality:**

- Test base connection: `ping -c 100 YOUR_PUBLIC_IP`
- High jitter/packet loss → Try TCP protocol

### Certificate Errors

**Error:** "certificate verify failed"

**Solution:**

```bash
# Check certificate validity
openssl x509 -in /data/openvpn/pki/issued/client.crt -noout -dates

# If expired, regenerate:
openvpn-client-revoke oldclient
openvpn-client-add oldclient
```

**Error:** "TLS key negotiation failed"

**Solution:**

- Ensure `ta.key` matches between server and client
- Check key-direction (server: 0, client: 1)

### Add-on Won't Start

**Check logs:**

- Look for "Address already in use" → Something else using port 1194
- "Permission denied" → Restart Home Assistant host

**Reset PKI:**

```bash
rm -rf /data/openvpn/pki
# Restart add-on to reinitialize
```

### Lost/Corrupt Configuration

**Restore from backup:**

```bash
cd /data
rm -rf openvpn
tar -xzf /share/openvpn-backup-DATE.tar.gz
```

**Fresh start:**

```bash
rm -rf /data/openvpn
# Restart add-on
# Regenerate all client certificates
```

---

## FAQ

**Q: Can I run this with Nabu Casa Cloud?**
A: Yes, but unnecessary. Use if you want additional VPN access or need to access other LAN devices.

**Q: What's my public IP?**
A: Visit whatismyip.com or use: `curl ifconfig.me`

**Q: Should I use UDP or TCP?**
A: UDP (faster, recommended). Use TCP only if UDP is blocked.

**Q: How many clients can connect?**
A: Default 10, configurable up to 100. Depends on hardware resources.

**Q: Can I use with multiple HA instances?**
A: Yes, use different ports (e.g., 1194, 1195) and subnets (10.8.0.0, 10.9.0.0).

**Q: Is this secure?**
A: Yes. Uses AES-256-GCM encryption, 2048-bit RSA certificates, TLS 1.2+, and HMAC authentication—industry standards.

**Q: What if my IP changes?**
A: Use DDNS (DuckDNS add-on available for HA). Update `.ovpn` files when IP changes.

**Q: Can I access from multiple locations?**
A: Yes, same `.ovpn` works from anywhere with internet access.

---

## Support & Contributing

- **Issues:** [GitHub Issues](https://github.com/laurentboutet/ha-openvpn-server/issues)
- **Discussions:** [Home Assistant Community](https://community.home-assistant.io/)
- **Documentation:** This file
- **Updates:** Star the repo for notifications

**Contributing:**
Pull requests welcome! Please test thoroughly before submitting.

---

**Last Updated:** January 2026  
**Add-on Version:** 1.0.0
