# Adding ESPHome Smart Doorbell Project to Home Assistant

## Method 1: ESPHome Add-on (Recommended)

### Prerequisites
- Home Assistant with Supervisor (Home Assistant OS or Supervised)
- ESPHome Add-on installed

### Step-by-Step Instructions

#### 1. Install ESPHome Add-on
1. Go to **Settings** → **Add-ons** → **Add-on Store**
2. Search for "ESPHome" 
3. Click **Install** on the official ESPHome add-on
4. Start the add-on and enable **Start on boot**

#### 2. Add Your Device
1. Open ESPHome add-on web UI
2. Click **+ NEW DEVICE**
3. Choose **Continue** 
4. Enter device details:
   - **Name**: `smart-doorbell-project`
   - **Device Type**: Choose ESP32
5. Skip the wireless credentials (we'll use existing config)

#### 3. Upload Your Configuration
1. In ESPHome dashboard, click **EDIT** on your new device
2. Replace the generated content with your `smart-doorbell-project.yaml` content
3. Create a `secrets.yaml` file with:
   ```yaml
   wifi_ssid: "Your_WiFi_Name"
   wifi_password: "Your_WiFi_Password"
   api_encryption_key_sdb: "your-32-character-base64-encryption-key"
   ota_password_sdb: "your-secure-ota-password"
   ap_wifi_password_sdb: "your-hotspot-password"
   ```
4. Click **SAVE**

#### 4. Compile and Flash
1. Click **INSTALL** 
2. Choose **Wirelessly** (if device is already flashed) or **Plug into this computer**
3. Wait for compilation and upload

#### 5. Auto-Discovery in Home Assistant
- Once online, the device should appear in **Settings** → **Devices & Services**
- Look for "ESPHome" integration with your device
- Click **CONFIGURE** if prompted

## Method 2: Standalone ESPHome Installation

### If you're running ESPHome separately (not as HA add-on):

#### 1. Ensure API Configuration
Your configuration already has the API enabled:
```yaml
api:
  encryption:
    key: !secret api_encryption_key_sdb
```

#### 2. Manual Integration
1. Go to **Settings** → **Devices & Services**
2. Click **+ ADD INTEGRATION**
3. Search for "ESPHome"
4. Enter your ESP32's IP address
5. Enter the encryption key from your ESPHome config

## Method 3: MQTT Integration (Alternative)

### If you prefer MQTT, add this to your ESPHome config:

```yaml
mqtt:
  broker: your_mqtt_broker_ip
  username: !secret mqtt_username
  password: !secret mqtt_password
  discovery: true
  discovery_retain: false
  topic_prefix: esphome/smart_doorbell_project
```

## Device Configuration for Better HA Integration

### Add Device Information
Add this to your ESPHome configuration for better device identification:

```yaml
esphome:
  name: smart-doorbell-project
  friendly_name: Smart Doorbell Project
  comment: "Smart doorbell with chime control system"
  project:
    name: "custom.smart_doorbell"
    version: "1.0"

# Device information for Home Assistant
device_info:
  manufacturer: "Custom Build"
  model: "ESP32 Smart Doorbell"
  sw_version: "1.0"
```

## Expected Home Assistant Entities

After successful integration, you'll see these entities:

### 🔔 **Doorbell Controls**
- `switch.doorbell_chime_active` - Enable/disable the chime
- `switch.doorbell_restart` - Remotely restart the device

### � **Sensors**
- `binary_sensor.doorbell` - Shows when doorbell button is pressed
- `sensor.doorbell_uptime` - Device uptime
- `sensor.doorbell_wifi_signal` - WiFi signal strength

### 🏠 **Device Status**
- `binary_sensor.smart_doorbell_project_status` (online/offline)

## Creating a Dashboard Card

### Example Lovelace Card Configuration:

```yaml
type: entities
title: Smart Doorbell System
entities:
  - entity: binary_sensor.doorbell
    name: Doorbell Button
  - entity: switch.doorbell_chime_active
    name: Chime Active
  - type: divider
  - entity: sensor.doorbell_uptime
    name: System Uptime
  - entity: sensor.doorbell_wifi_signal
    name: WiFi Signal
  - type: divider
  - entity: switch.doorbell_restart
    name: Restart Device
```

## Automation Example

### Create automations in Home Assistant:

```yaml
# Example: Notification when doorbell is pressed
automation:
  - alias: "Doorbell Notification"
    trigger:
      - platform: state
        entity_id: binary_sensor.doorbell
        to: 'on'
    action:
      - service: notify.mobile_app_your_phone
        data:
          title: "Doorbell"
          message: "Someone is at the door!"

# Example: Disable chime at night
  - alias: "Disable doorbell chime at night"
    trigger:
      - platform: time
        at: "22:00:00"
    action:
      - service: switch.turn_off
        target:
          entity_id: switch.doorbell_chime_active

# Example: Enable chime in morning
  - alias: "Enable doorbell chime in morning"
    trigger:
      - platform: time
        at: "07:00:00"
    action:
      - service: switch.turn_on
        target:
          entity_id: switch.doorbell_chime_active
```

## Troubleshooting

### Device Not Appearing
1. **Check Network**: Ensure ESP32 and HA are on same network
2. **Verify API**: Confirm API encryption key matches
3. **Check Logs**: Look in HA logs for connection errors
4. **Manual Add**: Try adding via IP address manually

### Connection Issues
1. **Firewall**: Ensure port 6053 is open for ESPHome API
2. **WiFi Signal**: Check `sensor.doorbell_wifi_signal` strength
3. **Restart**: Reboot both ESP32 and Home Assistant

### Entity Names
- Entity names will be prefixed with your device name
- Use **Settings** → **Devices & Services** → **ESPHome** → your device to customize entity names

## Advanced Features

### Group Configuration
Create groups for easier management:

```yaml
# In configuration.yaml
group:
  doorbell_system:
    name: "Smart Doorbell System"
    entities:
      - switch.doorbell_chime_active
      - binary_sensor.doorbell
      - sensor.doorbell_uptime
      - sensor.doorbell_wifi_signal
```

### Template Sensors
Create calculated sensors:

```yaml
# Doorbell activity counter
template:
  - sensor:
      - name: "Daily Doorbell Presses"
        state: >
          {{ states.binary_sensor.doorbell.attributes.get('count', 0) }}
        unit_of_measurement: "presses"
```

Once you follow these steps, your ESPHome smart doorbell system will be fully integrated into Home Assistant with all entities available for monitoring, control, and automation!
