# ============================================================================
# HOME ASSISTANT PACKAGES SETUP - OpenVPN Integration
# ============================================================================

## What are Packages?

Packages allow you to organize Home Assistant configuration into separate files
instead of having everything in one large configuration.yaml file.

Benefits:
- Better organization
- Easier maintenance
- Can enable/disable entire integrations by renaming files
- Cleaner configuration.yaml

## Setup Instructions

### Step 1: Enable Packages in configuration.yaml

Add this to your `configuration.yaml` file:

```yaml
# Enable packages folder
homeassistant:
  packages: !include_dir_named packages
```

If you already have a `homeassistant:` section, add only the packages line to it.

### Step 2: Create Packages Folder

SSH to Home Assistant or use File Editor add-on:

```bash
# Navigate to config directory
cd /config

# Create packages folder
mkdir packages

# Verify it was created
ls -la
```

You should see a `packages` folder in `/config/`

### Step 3: Create OpenVPN Package File

Create file: `/config/packages/openvpn.yaml`

**Option A: Using File Editor Add-on (Easiest)**
1. Install "File Editor" add-on from Add-on Store
2. Open File Editor
3. Click folder icon → Navigate to config folder
4. Right-click → New file
5. Name it: `packages/openvpn.yaml`
6. Paste the configuration from openvpn_config.yaml
7. Save

**Option B: Using SSH/Terminal**

```bash
# Navigate to packages folder
cd /config/packages

# Create openvpn.yaml file
nano openvpn.yaml

# Paste the configuration, then:
# Ctrl+X to exit
# Y to save
# Enter to confirm
```

### Step 4: Update Add-on Slug

In your new `openvpn.yaml` file, find and replace `local_openvpn_server`:

**How to find your slug:**
1. Go to: Settings → Add-ons → OpenVPN Server
2. Look at browser URL
3. URL format: `/hassio/addon/YOUR_SLUG/info`
4. Copy YOUR_SLUG
5. Replace `local_openvpn_server` in openvpn.yaml with your slug

Example:
```yaml
# Before
shell_command:
  openvpn_restart_server: 'ha addons restart local_openvpn_server'

# After (your slug will be different)
shell_command:
  openvpn_restart_server: 'ha addons restart 0e7baaac_openvpn_server'
```

### Step 5: Check Configuration

Before restarting, validate your configuration:

1. **Developer Tools** → **YAML** tab
2. Click **"Check Configuration"**
3. Wait for validation
4. Should see: ✅ "Configuration valid!"

If errors appear, check:
- YAML indentation (use spaces, not tabs)
- Add-on slug is correct
- No typos in file

### Step 6: Restart Home Assistant

1. **Settings** → **System** → **Restart**
2. Wait 30-60 seconds for restart
3. Log back in

### Step 7: Verify Entities Created

**Developer Tools** → **States**, search for:
- ✅ `sensor.openvpn_server_status`
- ✅ `binary_sensor.openvpn_server_online`
- ✅ `input_boolean.openvpn_notifications`
- ✅ `script.openvpn_restart_server`

## File Structure

After setup, your structure should be:

```
/config/
├── configuration.yaml          # Main config with packages enabled
├── packages/                   # Packages folder
│   ├── openvpn.yaml           # OpenVPN integration
│   └── (other packages...)    # Future integrations
├── automations.yaml
├── scripts.yaml
└── ...
```

## Managing Packages

**Disable a package:** Rename the file
```bash
mv packages/openvpn.yaml packages/openvpn.yaml.disabled
```

**Enable it again:** Remove `.disabled`
```bash
mv packages/openvpn.yaml.disabled packages/openvpn.yaml
```

**Always restart Home Assistant after changes!**

## Alternative: Keep in configuration.yaml

If you prefer NOT to use packages, you can still add the configuration
directly to your `configuration.yaml` file:

1. Open `configuration.yaml`
2. Append the entire content of `openvpn_config.yaml` at the end
3. Update the add-on slug
4. Save and restart Home Assistant

This works fine but makes configuration.yaml larger and harder to manage.

## Troubleshooting

**Package not loading:**
- Check `homeassistant:` section has `packages: !include_dir_named packages`
- Verify folder name is exactly `packages` (lowercase)
- Ensure `openvpn.yaml` is inside packages folder
- Check YAML syntax (indentation with spaces, not tabs)

**Entities not appearing:**
- Verify add-on slug is correct
- Check if `sensor.openvpn_connected_clients` exists (from add-on)
- Restart Home Assistant after changes

**Configuration check fails:**
- Review error message carefully
- Check YAML indentation
- Ensure no duplicate IDs in automations
- Verify template syntax

## Additional Packages Examples

You can create separate packages for different integrations:

```
packages/
├── openvpn.yaml          # VPN monitoring
├── weather.yaml          # Weather automations
├── security.yaml         # Security system
├── lighting.yaml         # Light automations
└── media.yaml            # Media player controls
```

Each package is completely independent and can be managed separately.

## Notes

- ✅ Packages are loaded automatically on restart
- ✅ Each package can contain any Home Assistant configuration
- ✅ No limit on number of packages
- ✅ Great for organizing large configurations
- ⚠️ Always validate before restarting
- ⚠️ Keep backups of configuration files

============================================================================
