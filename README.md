# Home Assistant Add-on: OpenVPN Server

![Supports aarch64 Architecture][aarch64-shield]
![Supports amd64 Architecture][amd64-shield]
![Supports armv7 Architecture][armv7-shield]

Full-featured OpenVPN server for Home Assistant with complete PKI management, allowing secure remote access to your Home Assistant instance and local network.

## About

This add-on provides a production-ready OpenVPN server with:

- 🔐 Complete PKI (Public Key Infrastructure) with EasyRSA
- 📱 Easy client certificate generation and management
- 🔄 Certificate revocation support (CRL)
- 🌐 Automatic client configuration file generation
- 🛡️ Industry-standard encryption (AES-256-GCM)
- 📊 Connection monitoring and logging
- ⚙️ Fully configurable through Home Assistant UI

## Installation

1. **Add Repository to Home Assistant:**
   - Navigate to **Supervisor** → **Add-on Store** → **⋮** (menu) → **Repositories**
   - Add: `https://github.com/laurentboutet/ha-openvpn-server`
   - Click **Add**

2. **Install Add-on:**
   - Find "OpenVPN Server" in the add-on store
   - Click **Install**
   - Wait for installation to complete

3. **Configure:**
   - Go to **Configuration** tab
   - Adjust settings (see Configuration section)
   - **Save** configuration

4. **Start:**
   - Go to **Info** tab
   - Click **Start**
   - Enable **Start on boot** for automatic startup

## Configuration

### Basic Configuration

```yaml
proto: udp                        # Protocol: udp or tcp
port: 1194                        # OpenVPN port
server_subnet: 10.8.0.0          # VPN subnet
server_netmask: 255.255.255.0    # VPN netmask
dns_servers:
  - 1.1.1.1                       # DNS servers pushed to clients
  - 1.0.0.1
routes:
  - 192.168.1.0 255.255.255.0    # Routes pushed to clients (your LAN)
client_to_client: true            # Allow clients to communicate
duplicate_cn: false               # Allow duplicate client connections
max_clients: 10                   # Maximum simultaneous clients
log_level: 3                      # Verbosity level (0-9)
cipher: AES-256-GCM              # Encryption cipher
auth: SHA256                      # Authentication algorithm
```

## Network Setup

### Port Forwarding (Required)

- Log into your router
- Forward UDP **port 1194** → Your Home Assistant IP
- Save settings

### DDNS (Recommended)

- Use a DDNS service (DuckDNS, No-IP, etc.) if you have a dynamic IP
- Update client .ovpn files with your DDNS hostname

## Usage

### Creating Client Certificates

#### Via Terminal/SSH

Access add-on terminal (Terminal icon in add-on page), then:

```bash
# Generate new client
openvpn-client-add phone

# Generate another client
openvpn-client-add laptop
```

#### Download Configuration

- Client .ovpn files are saved to `/share/` directory
- Access via:
  - **File Editor** add-on
  - **Samba** share
  - **SSH/SFTP**

### Managing Clients

#### List all clients

```bash
openvpn-client-list
```

#### Revoke a client certificate

```bash
openvpn-client-revoke old-phone
```

> **Note:** Restart add-on after revocation

## Connecting Clients

### Android/iOS

- Install **OpenVPN Connect** from app store
- Import `.ovpn` file
- Connect

### Windows

- Install OpenVPN GUI
- Copy `.ovpn` to `C:\Program Files\OpenVPN\config\`
- Right-click tray icon → Connect

### macOS

- Install **Tunnelblick** or **OpenVPN Connect**
- Import `.ovpn` file
- Connect

### Linux

```bash
sudo openvpn --config client.ovpn
```

## Troubleshooting

### Cannot connect to VPN

- **Check port forwarding:** Ensure UDP 1194 is forwarded to HA
- **Verify public IP:** Update remote line in .ovpn with correct IP/DDNS
- **Check firewall:** Ensure firewall allows UDP 1194
- **Review logs:** Check add-on logs for errors

### Can connect but no LAN access

- **Check routes:** Ensure routes in config matches your LAN subnet
- **Verify IP:** Confirm Home Assistant is on the same network
- **Firewall rules:** Check if HA host blocks forwarded traffic

### Certificate issues

```bash
# Reinitialize PKI (WARNING: Deletes all certificates!)
rm -rf /data/openvpn/pki
# Restart add-on
```

## Security Best Practices

- ✅ Use strong, unique client names
- ✅ Revoke certificates for lost devices immediately
- ✅ Regularly backup `/data/openvpn/pki` directory
- ✅ Use DDNS with HTTPS for additional security
- ✅ Keep add-on updated
- ❌ Don't share .ovpn files insecurely
- ❌ Don't enable duplicate_cn unless necessary

## Backup & Restore

### Backup PKI

```bash
tar -czf openvpn-backup.tar.gz -C /data openvpn/
```

### Restore PKI

```bash
tar -xzf openvpn-backup.tar.gz -C /data/
```

## Support

- 🐛 [Report Issues](https://github.com/laurentboutet/ha-openvpn-server/issues)
- 💬 [Community Forum](https://community.home-assistant.io/)
- 📖 [Full Documentation](https://openvpn.net/community/)

## License

MIT License - See LICENSE file
