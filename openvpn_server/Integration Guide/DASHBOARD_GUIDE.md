# Dashboard Cards - Complete Guide

## What is a Dashboard Card?

A **dashboard card** is a visual widget in your Home Assistant **Lovelace dashboard** (the main UI you see when you open Home Assistant).

Cards display information and controls for your entities (sensors, switches, etc.).

**The "Modern Entities Card"** shows your OpenVPN server status, connected clients, and control buttons in a clean, organized panel.

---

## What Will You See?

The Modern Entities Card will display:

```
┌─────────────────────────────────────┐
│ 🔒 OpenVPN Server                   │
├─────────────────────────────────────┤
│ 🖥️  Server Status      ●  ON        │
│ 👥  Active Connections     2        │
│ ℹ️   Details           Running (2.. │
├─────────────────────────────────────┤
│ 🔔  Connection Alerts  ●  ON        │
├─────────────────────────────────────┤
│ 🔄  Restart Server                  │
│ ⚙️   Manage Clients                 │
├─────────────────────────────────────┤
│ Connection History (24h)            │
│ ▂▃▅▆▇█▆▅▃▂▁▂▃▅▆▇█▆▅▃▂│
└─────────────────────────────────────┘
```

**Features:**
- Real-time server status (green dot = online)
- Number of connected VPN clients
- Toggle notification alerts on/off
- Button to restart server (with confirmation)
- Button to see client management instructions
- 24-hour connection history graph

---

## Where to Add Dashboard Cards?

Dashboard cards are added in your **Home Assistant Lovelace dashboard**.

**Two places to add cards:**

### **1. Overview Tab (Main Dashboard)**
The default home screen you see when opening Home Assistant

### **2. Custom Dashboard/Tab**
Create a dedicated VPN monitoring dashboard (optional)

---

## How to Add the Card - Step by Step

### **Method 1: Visual Editor (Easiest)**

#### Step 1: Enter Edit Mode

1. Open Home Assistant in your browser
2. Click on **Overview** (or any dashboard tab)
3. Click **three dots** ⋮ in top right corner
4. Click **"Edit Dashboard"**

You should see "Edit mode active" at the top.

#### Step 2: Add New Card

1. Click **"+ ADD CARD"** button (bottom right)
2. In the search box, type: **"Entities"**
3. Click **"Entities"** card type

#### Step 3: Configure the Card

You'll see a preview and configuration panel on the right.

**Switch to YAML mode:**
1. Click **"Show Code Editor"** at the bottom of the config panel
2. **Delete everything** in the code editor
3. **Copy and paste** this complete code:

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

#### Step 4: Save

1. Click **"Save"** (bottom right of config panel)
2. Card should appear on your dashboard
3. Click **"Done"** (top right) to exit edit mode

---

### **Method 2: YAML Editor (Advanced)**

If your dashboard is in YAML mode:

1. Click **three dots** ⋮ → **"Edit Dashboard"**
2. Click **three dots** ⋮ again → **"Raw configuration editor"**
3. Scroll to `views:` section
4. Find the view where you want the card (or create new view)
5. Add the card code under `cards:` section
6. Click **"Save"**

**Example:**
```yaml
views:
  - title: Home
    cards:
      # Your existing cards...

      # Add OpenVPN card here:
      - type: entities
        title: 🔒 OpenVPN Server
        icon: mdi:vpn
        # ... rest of card config
```

---

## Creating a Dedicated VPN Dashboard Tab (Optional)

**For better organization, create a separate tab for VPN monitoring:**

### Step 1: Enter Edit Mode
- Click **three dots** ⋮ → **"Edit Dashboard"**

### Step 2: Add New View/Tab
1. Click **"+ ADD VIEW"** (top of screen)
2. Configure:
   - **Title:** VPN Monitor
   - **Icon:** `mdi:vpn`
   - **URL:** `vpn` (optional)
3. Click **"Save"**

### Step 3: Add OpenVPN Card
1. New empty tab appears
2. Click **"+ ADD CARD"**
3. Follow Method 1 steps above to add the Entities card

---

## All Dashboard Card Options

You have **4 card options** in INTEGRATION.md:

### **Option 1: Modern Entities Card** ⭐ Recommended
- Complete monitoring dashboard
- All controls in one card
- Includes 24h history graph
- Best for primary VPN monitoring

### **Option 2: Compact Status Card**
- Small 3-column view
- Quick status glance
- No controls, just status
- Good for overview dashboards

### **Option 3: History Card**
- Only shows connection timeline
- 24-hour graph
- Good alongside other cards

### **Option 4: Full Dashboard Panel**
- Multiple cards stacked vertically
- Most comprehensive view
- Includes status header with alerts
- Best for dedicated VPN dashboard tab

---

## Troubleshooting Dashboard Cards

### Card Shows "Entity not available"

**Check entities exist:**
1. **Developer Tools** → **States**
2. Search for the entity name (e.g., `sensor.openvpn_server_status`)
3. If not found, verify:
   - Configuration is in `configuration.yaml`
   - Home Assistant was restarted
   - No errors in logs

### Card Shows Blank/Empty

**Verify YAML formatting:**
- Check indentation (use spaces, not tabs)
- Ensure entity names are correct
- Try "Show Code Editor" and check for errors

### "Unknown type: entities"

**Card type not supported:**
- Update Home Assistant to latest version
- Try standard card types (entities should always work)

### Graph Footer Not Showing

**Check entity has history:**
- Entity must have been active for some time
- Check: **History** → Search for entity
- If no data, wait for entity to collect history

---

## Card Customization Tips

### Change Card Order
In edit mode, **drag cards** to reorder them

### Resize Card
In edit mode, **drag corners** to resize

### Change Icon
```yaml
icon: mdi:shield-lock  # or mdi:vpn, mdi:security, etc.
```

Find icons at: https://pictogrammers.com/library/mdi/

### Change Colors
```yaml
state_color: true  # Shows green/red based on state
```

### Remove Graph Footer
Delete the `footer:` section if you don't want the graph

### Add More Entities
Add any OpenVPN-related entity to the card:
```yaml
entities:
  - entity: sensor.your_entity
    name: Your Label
    icon: mdi:icon-name
```

---

## Example: Simple Mobile View

**For mobile, use the compact card:**

```yaml
type: glance
title: VPN
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

Small, fits perfectly on phone screens!

---

## Where Are Dashboard Configurations Stored?

**UI-configured dashboards:**
- Stored in Home Assistant database
- Edited via UI

**YAML-configured dashboards:**
- Stored in `/config/.storage/lovelace` (UI mode)
- Or in `/config/ui-lovelace.yaml` (YAML mode)
- Or in separate files if using `lovelace:` in configuration.yaml

**Don't edit `.storage` files directly - use the UI!**

---

## Summary - Quick Start

1. ✅ Add OpenVPN integration to `configuration.yaml`
2. ✅ Restart Home Assistant
3. ✅ Verify entities exist (Developer Tools → States)
4. ✅ Open dashboard → Edit mode
5. ✅ Add Card → Choose "Entities"
6. ✅ Show Code Editor → Paste card YAML
7. ✅ Save → Done
8. ✅ See your OpenVPN monitoring card! 🎉

---

## Need Help?

**Check these:**
- Entities exist: **Developer Tools** → **States**
- Configuration valid: **Developer Tools** → **YAML** → Check Configuration
- Dashboard mode: **Edit Dashboard** → Works in both UI and YAML mode
- Browser cache: Hard refresh (Ctrl+F5 or Cmd+Shift+R)

---

*Dashboard cards are the visual interface for your Home Assistant entities!*
