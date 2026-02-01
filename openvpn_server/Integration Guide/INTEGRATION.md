# OpenVPN Server - Home Assistant Integration Guide
**Version 1.0.15 | Updated February 2026**

---

## 📋 Overview

This integration provides **monitoring and control** for your OpenVPN Home Assistant Add-on:

- ✅ **Server Status Monitoring** - Real-time up/down status
- 👥 **Connected Clients Tracking** - See who's connected
- 📊 **Connection History** - Track connect/disconnect events
- 🔄 **Server Control** - Restart capability
- 🔔 **Smart Notifications** - Get alerted on connections

**Note:** Client creation is handled directly in the add-on **Configuration** tab. This integration focuses purely on monitoring and control.

---

## 🚀 Quick Start

### Step 1: Find Your Add-on Slug

**The slug in the configuration is UNIQUE to each installation!**

**To find YOUR slug:**
1. Go to: **Settings** → **Add-ons** → **OpenVPN Server**
2. Look at your browser **URL bar**
3. URL format: `/hassio/addon/YOUR_SLUG_HERE/info`
4. Copy `YOUR_SLUG_HERE`

**Example URLs:**
- `http://homeassistant.local:8123/hassio/addon/local_openvpn_server/info`
  - Slug: `local_openvpn_server`
- `http://homeassistant.local:8123/hassio/addon/0e7baaac_openvpn_server/info`
  - Slug: `0e9babbc_openvpn_server`

**Write down your slug - you'll need it in Step 2!**

---

### Step 2: Add Configuration

**Choose ONE method:**
- **Option A:** Add to `configuration.yaml` (simpler)
- **Option B:** Use packages folder (better organization)

---

#### **Option A: Add to configuration.yaml**

1. Open your `/config/configuration.yaml` file
2. Add the configuration below **at the end** of the file
3. **Replace `local_openvpn_server`** with YOUR slug (from Step 1)
4. Save the file

```yaml
# ============================================================================
# OpenVPN Server Integration - Monitoring & Control
# ============================================================================
# Version: 1.0.12
# Compatible: Home Assistant 2026.1+
# ============================================================================

# Shell Commands for Server Control
# ----------------------------------
shell_command:
  # ⚠️ IMPORTANT: Replace 'local_openvpn_server' with YOUR slug!
  openvpn_restart_server: 'ha addons restart local_openvpn_server'

# Input Boolean for Notification Control
# ---------------------------------------
input_boolean:
  openvpn_notifications:
    name: "OpenVPN Connection Alerts"
    icon: mdi:bell-ring
    initial: true

# Template Sensors - Modern Syntax (2026+)
# -----------------------------------------
template:
  - sensor:
      # Main status sensor with client count
      - name: "OpenVPN Server Status"
        unique_id: openvpn_server_status
        icon: >
          {% if states('sensor.openvpn_connected_clients') not in ['unavailable', 'unknown'] %}
            mdi:shield-check
          {% else %}
            mdi:shield-off
          {% endif %}
        state: >
          {% set clients = states('sensor.openvpn_connected_clients') %}
          {% if clients not in ['unavailable', 'unknown'] %}
            {% set count = clients | int(0) %}
            {% if count == 0 %}
              Running (no clients)
            {% elif count == 1 %}
              Running (1 client)
            {% else %}
              Running ({{ count }} clients)
            {% endif %}
          {% else %}
            Offline
          {% endif %}
        attributes:
          client_count: >
            {{ states('sensor.openvpn_connected_clients') | int(0) }}
          uptime: >
            {{ relative_time(states.sensor.openvpn_connected_clients.last_changed) }}
          last_updated: >
            {{ as_timestamp(states.sensor.openvpn_connected_clients.last_changed) | timestamp_custom('%Y-%m-%d %H:%M:%S') }}

  - binary_sensor:
      # Binary sensor for easier automations
      - name: "OpenVPN Server Online"
        unique_id: openvpn_server_online
        device_class: connectivity
        icon: >
          {% if this.state == 'on' %}
            mdi:vpn
          {% else %}
            mdi:vpn-off
          {% endif %}
        state: >
          {{ states('sensor.openvpn_connected_clients') not in ['unavailable', 'unknown'] }}

# Scripts for Dashboard Actions
# ------------------------------
script:
  openvpn_restart_server:
    alias: "Restart OpenVPN Server"
    icon: mdi:restart
    mode: single
    sequence:
      - service: shell_command.openvpn_restart_server
      - service: persistent_notification.create
        data:
          title: "🔄 OpenVPN Server Restarting"
          message: >
            Server restart initiated. Please wait 10-15 seconds for service to come back online.

            Check: Settings → Add-ons → OpenVPN Server → Logs
      - delay: "00:00:10"

  openvpn_open_addon_page:
    alias: "Open OpenVPN Add-on Page"
    icon: mdi:open-in-app
    mode: single
    sequence:
      - service: persistent_notification.create
        data:
          title: "📋 OpenVPN Add-on"
          message: >
            **Client Management:**
            Settings → Add-ons → OpenVPN Server → Configuration

            **View Connected Clients:**
            Settings → Add-ons → OpenVPN Server → Logs

            **Actions Available:**
            - Create new client certificates
            - Recreate lost .ovpn files
            - Revoke client access
            - List all clients

# Automations for Connection Monitoring
# --------------------------------------
automation:
  # Alert when client connects
  - id: openvpn_client_connected_alert
    alias: "OpenVPN - Client Connected"
    description: "Notify when a VPN client connects"
    mode: queued
    trigger:
      - platform: state
        entity_id: sensor.openvpn_connected_clients
    condition:
      - condition: template
        value_template: >
          {{ trigger.to_state.state not in ['unavailable', 'unknown'] and
             trigger.from_state.state not in ['unavailable', 'unknown'] and
             trigger.to_state.state | int(0) > trigger.from_state.state | int(0) }}
      - condition: state
        entity_id: input_boolean.openvpn_notifications
        state: "on"
    action:
      - service: persistent_notification.create
        data:
          title: "🔒 VPN Client Connected"
          message: >
            **Status:** {{ states('sensor.openvpn_connected_clients') }} client(s) now connected

            **Time:** {{ now().strftime('%H:%M:%S') }}

            {% if trigger.to_state.attributes.clients %}
            **Clients:** {{ trigger.to_state.attributes.clients }}
            {% endif %}

  # Alert when client disconnects
  - id: openvpn_client_disconnected_alert
    alias: "OpenVPN - Client Disconnected"
    description: "Notify when a VPN client disconnects"
    mode: queued
    trigger:
      - platform: state
        entity_id: sensor.openvpn_connected_clients
    condition:
      - condition: template
        value_template: >
          {{ trigger.to_state.state not in ['unavailable', 'unknown'] and
             trigger.from_state.state not in ['unavailable', 'unknown'] and
             trigger.to_state.state | int(0) < trigger.from_state.state | int(0) }}
      - condition: state
        entity_id: input_boolean.openvpn_notifications
        state: "on"
    action:
      - service: persistent_notification.create
        data:
          title: "🔓 VPN Client Disconnected"
          message: >
            **Status:** {{ states('sensor.openvpn_connected_clients') }} client(s) now connected

            **Time:** {{ now().strftime('%H:%M:%S') }}

            **Previous:** {{ trigger.from_state.state }} clients

  # Alert when server goes offline
  - id: openvpn_server_offline_alert
    alias: "OpenVPN - Server Offline"
    description: "Alert when OpenVPN server stops"
    mode: single
    trigger:
      - platform: state
        entity_id: binary_sensor.openvpn_server_online
        to: "off"
        for: "00:00:30"
    condition:
      - condition: state
        entity_id: input_boolean.openvpn_notifications
        state: "on"
    action:
      - service: persistent_notification.create
        data:
          title: "⚠️ OpenVPN Server Offline"
          message: >
            OpenVPN server has stopped responding.

            **Check:** Settings → Add-ons → OpenVPN Server → Logs

            **Action:** Use dashboard restart button or manually restart add-on

  # Alert when server comes online
  - id: openvpn_server_online_alert
    alias: "OpenVPN - Server Online"
    description: "Confirm when OpenVPN server starts"
    mode: single
    trigger:
      - platform: state
        entity_id: binary_sensor.openvpn_server_online
        to: "on"
        for: "00:00:05"
    condition:
      - condition: state
        entity_id: input_boolean.openvpn_notifications
        state: "on"
    action:
      - service: persistent_notification.create
        data:
          title: "✅ OpenVPN Server Online"
          message: >
            OpenVPN server is now running and accepting connections.

            **Status:** {{ states('sensor.openvpn_server_status') }}
```

---

#### **Option B: Use Packages Folder (Recommended)**

**Benefits:** Better organization, easier to manage, can enable/disable easily

**Setup:**

1. **Enable packages in `configuration.yaml`:**

```yaml
# Add this to your configuration.yaml
homeassistant:
  packages: !include_dir_named packages
```

2. **Create packages folder:**

```bash
# Using SSH or Terminal
mkdir /config/packages
```

Or use **File Editor** add-on:
- Open File Editor
- Create new folder called `packages` in `/config/`

3. **Create `/config/packages/openvpn.yaml`:**

- Copy the ENTIRE configuration from Option A above
- Paste into `/config/packages/openvpn.yaml`
- **Replace `local_openvpn_server`** with YOUR slug
- Save

4. **Restart Home Assistant**

**Full packages setup guide:** See `PACKAGE_SETUP.md` in repository for detailed instructions.

---

### Step 3: Validate Configuration

Before restarting:

1. **Developer Tools** → **YAML** tab
2. Click **"Check Configuration"**
3. Wait for validation
4. Should see: ✅ **"Configuration valid!"**

If errors appear:
- Check YAML indentation (use spaces, not tabs)
- Verify your add-on slug is correct
- Ensure you replaced ALL occurrences of `local_openvpn_server`

---

### Step 4: Restart Home Assistant

1. **Settings** → **System** → **Restart**
2. Wait 30-60 seconds for restart
3. Log back in

---

### Step 5: Verify Entities Created

Go to **Developer Tools** → **States** and search for:

- ✅ `sensor.openvpn_server_status`
- ✅ `sensor.openvpn_connected_clients` (from add-on)
- ✅ `binary_sensor.openvpn_server_online`
- ✅ `input_boolean.openvpn_notifications`
- ✅ `script.openvpn_restart_server`
- ✅ `script.openvpn_open_addon_page`

---

## 🎨 Dashboard Cards

### Option 1: Modern Entities Card (Recommended)

```yaml
type: entities
title: 🔒 OpenVPN Server
icon: mdi:vpn
show_header_toggle: false
state_color: true
entities:
  # Server Status
  - entity: binary_sensor.openvpn_server_online
    name: Server Status
    icon: mdi:server-network

  # Connected Clients Count
  - entity: sensor.openvpn_connected_clients
    name: Active Connections
    icon: mdi:account-network

  # Full Status with Uptime
  - entity: sensor.openvpn_server_status
    name: Details
    icon: mdi:information-outline

  # Divider
  - type: divider

  # Notification Toggle
  - entity: input_boolean.openvpn_notifications
    name: Connection Alerts
    icon: mdi:bell-ring

  # Divider
  - type: divider

  # Actions
  - entity: script.openvpn_restart_server
    name: Restart Server
    icon: mdi:restart
    tap_action:
      action: call-service
      service: script.openvpn_restart_server
      confirmation:
        text: Restart OpenVPN server?

  - entity: script.openvpn_open_addon_page
    name: Manage Clients
    icon: mdi:account-cog
    tap_action:
      action: call-service
      service: script.openvpn_open_addon_page

footer:
  type: graph
  entity: sensor.openvpn_connected_clients
  hours_to_show: 24
  detail: 1
```

---

### Option 2: Compact Status Card

```yaml
type: glance
title: OpenVPN Monitor
show_name: true
show_icon: true
show_state: true
state_color: true
columns: 3
entities:
  - entity: binary_sensor.openvpn_server_online
    name: Server

  - entity: sensor.openvpn_connected_clients
    name: Clients

  - entity: input_boolean.openvpn_notifications
    name: Alerts
    tap_action:
      action: toggle
```

---

### Option 3: History Card (Connection Timeline)

```yaml
type: history-graph
title: VPN Connection History
hours_to_show: 24
refresh_interval: 60
entities:
  - entity: sensor.openvpn_connected_clients
    name: Connected Clients
```

---

### Option 4: Full Dashboard Panel (All-in-One)

```yaml
type: vertical-stack
cards:
  # Status Header
  - type: markdown
    content: >
      # 🔒 OpenVPN Server

      {% if is_state('binary_sensor.openvpn_server_online', 'on') %}
      <ha-alert alert-type="success">✅ **Server Online** - {{ states('sensor.openvpn_server_status') }}</ha-alert>
      {% else %}
      <ha-alert alert-type="error">❌ **Server Offline** - Check add-on logs</ha-alert>
      {% endif %}

  # Main Status
  - type: entities
    show_header_toggle: false
    state_color: true
    entities:
      - entity: binary_sensor.openvpn_server_online
        name: Server Status
      - entity: sensor.openvpn_connected_clients
        name: Active Connections
      - entity: sensor.openvpn_server_status
        name: Details
      - type: divider
      - entity: input_boolean.openvpn_notifications
        name: Enable Alerts

  # Control Buttons
  - type: horizontal-stack
    cards:
      - type: button
        name: Restart
        icon: mdi:restart
        tap_action:
          action: call-service
          service: script.openvpn_restart_server
          confirmation:
            text: Restart OpenVPN?

      - type: button
        name: Manage
        icon: mdi:cog
        tap_action:
          action: call-service
          service: script.openvpn_open_addon_page

  # Connection History
  - type: history-graph
    title: Connection History (24h)
    hours_to_show: 24
    refresh_interval: 60
    entities:
      - entity: sensor.openvpn_connected_clients
        name: Clients
```

---

## 📝 Client Management

**All client operations are in the add-on Configuration tab:**

### View Client List

**Method 1: Using Add-on Configuration**
1. **Settings** → **Add-ons** → **OpenVPN Server**
2. **Configuration** tab
3. Set **Client Action** to: `list`
4. Click **Save**
5. Go to **Logs** tab
6. See complete client list
7. **Important:** Set **Client Action** back to `none` and Save

**Method 2: Check Logs for Active Connections**
1. **Settings** → **Add-ons** → **OpenVPN Server**
2. **Logs** tab
3. Connected clients appear in real-time when they connect/disconnect

**Method 3: Use Dashboard Button**
- Click **"Manage Clients"** button in dashboard
- Follow instructions in notification

### Create New Client

1. **Settings** → **Add-ons** → **OpenVPN Server**
2. **Configuration** tab
3. Enter **Client Name** (e.g., `myphone`, `laptop`)
4. Set **Client Action** to: `create`
5. Click **Save**
6. Download from: `/share/openvpn/clients/clientname.ovpn`
7. **Important:** Set **Client Action** back to `none` and Save

### Recreate Lost .ovpn File

1. **Settings** → **Add-ons** → **OpenVPN Server**
2. **Configuration** tab
3. Enter existing **Client Name**
4. Set **Client Action** to: `recreate`
5. Click **Save**
6. New .ovpn file generated
7. **Important:** Set **Client Action** back to `none` and Save

### Revoke Client Access

1. **Settings** → **Add-ons** → **OpenVPN Server**
2. **Configuration** tab
3. Enter **Client Name** to revoke
4. Set **Client Action** to: `revoke`
5. Click **Save**
6. Restart add-on for revocation to take effect
7. **Important:** Set **Client Action** back to `none` and Save

---

## 📱 Mobile Notifications (Optional)

Replace `persistent_notification.create` with your mobile app notification service.

**To find your device:**
1. **Developer Tools** → **Services**
2. Search for `notify.mobile_app_`
3. Note your service name (e.g., `notify.mobile_app_iphone`)

**Update automations:**

```yaml
# Replace:
- service: persistent_notification.create
  data:
    title: "🔒 VPN Client Connected"
    message: "..."

# With:
- service: notify.mobile_app_your_phone  # Your device name
  data:
    title: "🔒 VPN Client Connected"
    message: "..."
```

---

## 🔧 Troubleshooting

### Entities Not Appearing

**Check if `sensor.openvpn_connected_clients` exists:**
1. **Developer Tools** → **States**
2. Search for `openvpn_connected_clients`
3. If missing, check add-on is running

**If still missing:**
- Verify add-on is started
- Check add-on logs for errors
- Restart add-on

### Server Status Shows "Offline"

1. Verify add-on is running: **Settings** → **Add-ons** → **OpenVPN Server**
2. Check **Logs** tab for errors
3. Restart add-on if needed
4. Verify `sensor.openvpn_connected_clients` exists

### Restart Button Not Working

**Check your slug is correct:**
1. Go to add-on page in browser
2. Check URL: `/hassio/addon/YOUR_SLUG/info`
3. Verify slug in configuration matches exactly

**Update if different:**
```yaml
shell_command:
  openvpn_restart_server: 'ha addons restart YOUR_ACTUAL_SLUG'
```

### Configuration Check Fails

Common issues:
- **Indentation:** Use spaces (not tabs), check alignment
- **Slug:** Verify add-on slug is correct
- **Duplicate IDs:** Automation IDs must be unique
- **Template syntax:** Check for missing `%}` or `}}`

**Fix and run Check Configuration again**

---

## 🎯 Features Summary

| Feature | Entity | Purpose |
|---------|--------|---------|
| **Server Status** | `sensor.openvpn_server_status` | Human-readable status with client count |
| **Online Check** | `binary_sensor.openvpn_server_online` | Simple on/off for automations |
| **Client Count** | `sensor.openvpn_connected_clients` | Number of active VPN connections (from add-on) |
| **Notifications** | `input_boolean.openvpn_notifications` | Toggle alerts on/off |
| **Restart** | `script.openvpn_restart_server` | Restart VPN server |
| **Manage Clients** | `script.openvpn_open_addon_page` | Show client management instructions |

---

## 📊 Automations Included

| Automation | Trigger | Action |
|------------|---------|--------|
| **Client Connected** | Client count increases | Notification with count and time |
| **Client Disconnected** | Client count decreases | Notification with previous/current count |
| **Server Offline** | Server down for 30s | Alert to check logs |
| **Server Online** | Server comes up | Confirmation notification |

All automations respect the `input_boolean.openvpn_notifications` toggle.

---

## 📊 Logbook & History

Entities automatically track in History/Logbook:
- Connection/disconnection events
- Server online/offline status changes
- Client count changes over time

**View:** **History** → Search for "OpenVPN"

---

## 🚀 Complete Setup Checklist

- [ ] Find your add-on slug from browser URL
- [ ] Add configuration (Option A or B)
- [ ] Replace `local_openvpn_server` with YOUR slug
- [ ] Check configuration is valid
- [ ] Restart Home Assistant
- [ ] Verify entities created
- [ ] Add dashboard card
- [ ] Test restart button
- [ ] Create VPN client in add-on Configuration
- [ ] Download .ovpn file from `/share/openvpn/clients/`
- [ ] Edit .ovpn file (replace `YOUR_PUBLIC_IP_OR_DDNS`)
- [ ] Configure router port forwarding (UDP 1194)
- [ ] Test VPN connection from phone/laptop
- [ ] Monitor in dashboard!

---

## 💡 Tips

**Finding Client Names:**
- Set Client Action to `list`, save, check Logs tab

**Lost .ovpn File:**
- Use `recreate` action with same client name

**Server Not Starting:**
- Check Logs tab for port conflicts or config errors

**Notifications Too Frequent:**
- Toggle `input_boolean.openvpn_notifications` off

**Multiple Locations:**
- Create package files for better organization
- Example: `packages/openvpn.yaml`, `packages/weather.yaml`

---

## 📦 Related Files

- **openvpn_config.yaml** - Ready-to-use configuration file
- **PACKAGE_SETUP.md** - Detailed packages folder setup guide
- **README.md** - Main add-on documentation

---

**Need Help?**

1. Check add-on logs: **Settings** → **Add-ons** → **OpenVPN Server** → **Logs**
2. Verify configuration: **Developer Tools** → **YAML** → **Check Configuration**
3. Review entity states: **Developer Tools** → **States**

---

*Last updated: February 2026 | Compatible with Home Assistant 2026.1+*
