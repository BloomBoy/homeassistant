# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant configuration repository running version 2025.8.3. The setup includes:

- Main Home Assistant configuration with YAML-based config
- Custom ZHA (Zigbee Home Automation) device quirks
- ESPHome device configurations for M5Stack Atom Echo devices
- Custom components (HACS-managed)
- Automation blueprints for various lighting and sensor scenarios

## Architecture

### Core Configuration Structure
- `configuration.yaml` - Main HA configuration with MQTT lights/sensors, ZHA setup, and notification groups
- `automations.yaml` - Automation definitions, primarily using blueprints from Blackshome
- `scripts.yaml` - Custom lighting scripts with sequential timing effects
- `scenes.yaml` - Scene definitions for different lighting states

### Custom Components
- `custom_components/family_safety/` - Microsoft Family Safety integration
- `custom_components/hacs/` - Home Assistant Community Store integration
- Custom components follow standard HA integration patterns with manifests, coordinators, and entity classes

### ZHA Quirks
- `custom_zha_quirks/hue_wall_switch_rdm004.py` - Custom device quirk for Philips/Signify RDM004 wall switches
- Implements custom clusters for handling button press events and device-specific behaviors

### ESPHome Configurations
- `esphome/` - Device configs for M5Stack Atom Echo voice assistants
- Uses packages from GitHub firmware repository
- Each device has unique encryption keys and WiFi credentials from secrets

### Blueprints
- `blueprints/automation/` - Community automation blueprints for motion lighting, wall switches, and notifications
- `blueprints/script/` - Script blueprints for device configuration and notifications
- `blueprints/template/` - Template blueprints for sensor inversions

## Development Commands

This is a configuration-based Home Assistant setup with no build/test/lint commands. Development involves:

- Edit YAML configuration files directly
- Use Home Assistant's built-in configuration validation
- Test automations and scripts through HA UI
- ESPHome configurations can be validated with `esphome config <file.yaml>`

## Git Workflow and Deployment

**CRITICAL REMINDER**: Claude Code should NEVER commit changes or update Home Assistant directly. The user handles all Git operations and Home Assistant updates.

⚠️ **IMPORTANT**: Claude CANNOT test changes against the live Home Assistant system until AFTER the user has committed and deployed the changes. All REST API calls and MCP operations only work AFTER user deployment.

**Claude's Role:**
- Make configuration file changes locally in the repository
- Validate configuration syntax when possible (YAML structure only)
- Provide recommendations and implementation
- **NEVER assume entity_IDs exist until user confirms after deployment**

**User's Role:**
- Git commit and push changes to repository
- Update Home Assistant from Git repository
- Test and verify changes in live Home Assistant environment
- Report back any validation errors to Claude for fixes

**Deployment Process:**
1. Claude makes local file changes
2. User commits and pushes to Git
3. User updates Home Assistant from Git
4. User tests configuration validation
5. User reports any errors back to Claude for fixes

This separation ensures proper change control and prevents Claude from making assumptions about live system state before deployment.

## MCP Integration

This repository has Model Context Protocol (MCP) integration set up for direct Home Assistant access:

- **Home Assistant MCP Server**: Installed and running at `http://192.168.1.191:8123/mcp_server/sse`
- **Access Token**: Long-lived token stored in `.env` file (HA_TOKEN)
- **Local Proxy**: mcp-proxy installed via `uv tool install git+https://github.com/sparfenyuk/mcp-proxy`
- **Claude Code Configuration**: MCP server configured in `~/.claude.json` for automatic integration

### MCP Server Status
✅ **WORKING** - MCP integration is functional and configured:
- **Server**: home-assistant version 1.5.0
- **Connection**: HTTP 200 OK via mcp-proxy
- **Capabilities**: Tools and Prompts available
- **Authentication**: Bearer token authentication working

### Testing MCP Connection
```bash
source ~/.local/bin/env && mcp-proxy -H Authorization "Bearer $HA_TOKEN" http://192.168.1.191:8123/mcp_server/sse --debug
```

### Available MCP Methods

Claude Code can interact directly with Home Assistant through these MCP tools:

#### Device Control
- **`mcp__home-assistant__HassTurnOn`** - Turn on devices (lights, switches, etc.)
  ```
  mcp__home-assistant__HassTurnOn(name="Lampnamn", area="Område", floor="Våning")
  ```
- **`mcp__home-assistant__HassTurnOff`** - Turn off devices
  ```
  mcp__home-assistant__HassTurnOff(name="Lampnamn", area="Område")
  ```

#### Lighting Control
- **`mcp__home-assistant__HassLightSet`** - Set light brightness, color, temperature
  ```
  mcp__home-assistant__HassLightSet(name="Lampnamn", brightness=75, color="red", temperature=3000)
  ```

#### Media Player Control
- **`mcp__home-assistant__HassMediaPause`** - Pause media players
- **`mcp__home-assistant__HassMediaUnpause`** - Resume media players
- **`mcp__home-assistant__HassMediaNext`** - Skip to next track
- **`mcp__home-assistant__HassMediaPrevious`** - Go to previous track
- **`mcp__home-assistant__HassSetVolume`** - Set volume level (0-100)
- **`mcp__home-assistant__HassMediaSearchAndPlay`** - Search and play media

#### Climate Control
- **`mcp__home-assistant__HassClimateSetTemperature`** - Set target temperature
  ```
  mcp__home-assistant__HassClimateSetTemperature(name="Värmepump", temperature=22)
  ```

#### Todo Lists
- **`mcp__home-assistant__HassListAddItem`** - Add item to todo list
- **`mcp__home-assistant__HassListCompleteItem`** - Mark todo item as complete
- **`mcp__home-assistant__todo_get_items`** - Get items from todo lists

#### System Information
- **`mcp__home-assistant__GetLiveContext`** - Get current state of all devices and areas
  ```
  mcp__home-assistant__GetLiveContext()
  ```

#### Communication
- **`mcp__home-assistant__HassBroadcast`** - Broadcast message through home speakers
- **`mcp__home-assistant__HassCancelAllTimers`** - Cancel all running timers

### MCP Usage Guidelines
1. **Always use MCP methods first** for device control and status checking
2. **Prefer specific device names** over areas when possible for precision
3. **Check GetLiveContext** to understand current device states before making changes
4. **Use Swedish device names** as they appear in Home Assistant
5. **Test device availability** by attempting control rather than relying on "unavailable" status

This enables Claude Code to:
- Read entity states and configurations in real-time
- Control all smart home devices directly
- Test automations and scripts immediately
- Diagnose connectivity issues by attempting device control
- Update configurations and verify functionality instantly

## Key Patterns

### Lighting Scripts
- Sequential lighting effects with delays (e.g., `grus_trip_trapp_trull`)
- Parallel execution for synchronized lighting (e.g., `toalett_belysning_on`)
- Consistent transition times and brightness levels

### MQTT Integration
- Custom LED controls for Pi GPIO and Pico W devices
- Standardized topic structure: `homeassistant/device/component/action`
- QoS 1 and retain flags for reliability

### Automation Structure
- Heavy use of blueprints for consistency
- Motion-based lighting with configurable delays
- Device-specific triggers with proper entity/device ID references

### ZHA Quirks Development
- Custom cluster implementations for unsupported device features
- Event-based button handling with proper press type detection
- Device signature matching for automatic quirk application