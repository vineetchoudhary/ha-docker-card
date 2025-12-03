# Docker Card

A simple Lovelace card that lets you view and control your Docker containers from Home Assistant. When paired with the official Home Assistant Portainer integration, every entity shown below already exists (no templates or shell commands required). Drop the card into your dashboard and manage containers without leaving Home Assistant.

## Features

- Compact overview of your Docker host
- Auto-updating container list with live state badges and control actions
- Theme-aware styling with configurable running vs not-running accent colors
- Works out-of-the-box with entities provided by the Portainer integration; also supports any toggle-friendly domains (`switch`, `input_boolean`, `light`, etc.)
- Optional tap/hold actions per container row for quick navigation, service calls, or external links

## Requirements

- Home Assistant 2025.8 or newer
- Docker managed via the official Portainer integration (provides all referenced sensors, switches, and buttons)
- Optional: For non-Portainer environments, equivalent entities (sensors, binary_sensors, switches, scripts, etc.) that expose Docker data and operations

> ℹ️ This card **does not** fetch Docker data directly. It visualises data exposed through the standard Home Assistant entity model. Example helpers are included below for non-Portainer setups; if you already use the Home Assistant Portainer integration, you can plug its entities directly into the card.

## Installation

### 1. Via HACS (recommended)
1. In Home Assistant, open **HACS (Community Store) → ⋮ → Custom repositories**.
2. Add this repository as a **Dashboard** type and click **Add**.
3. Locate **Docker Card** under **Frontend** and install it.
4. Reload Lovelace resources (or restart Home Assistant) so the module is served.


### 2. Manual install

1. Copy `docker-card.js` to your Home Assistant `/config/www/docker-card/` folder.
2. Add the resource through **Settings → Dashboards → Resources → +**:
   ```yaml
   url: /local/docker-card/docker-card.js
   type: module
   ```

Basic card setup (YAML) using entities exposed by the Portainer integration:

```yaml
type: custom:docker-card
title: Docker @ MyServer
docker_overview:
  container_count: sensor.docker_containers_total
  containers_running: sensor.docker_containers_running
  containers_stopped: sensor.docker_containers_stopped
  docker_version: sensor.docker_version
  image_count: sensor.docker_images
  operating_system: sensor.host_os
  operating_system_version: sensor.host_os_version
  status: binary_sensor.docker_daemon_status
running_color: "var(--state-active-color)"
not_running_color: "#c22040"
containers:
  - name: Home Assistant
    status_entity: sensor.docker_homeassistant_status
    control_entity: switch.docker_homeassistant
    restart_entity: switch.docker_restart_homeassistant
    tap_action:
      action: more-info
    hold_action:
      action: url
      url_path: https://portainer.local/#!/2/docker/containers/homeassistant
  - name: Node-RED
    status_entity: sensor.docker_nodered_status
    control_entity: switch.docker_nodered
    restart_entity: button.docker_restart_nodered
    tap_action:
      action: toggle
    hold_action:
      action: call-service
      service: script.trigger_container_diagnostics
```

## Quick Start (Portainer integration)

1. Install the **Portainer** integration and complete its setup wizard (Settings → Devices & Services → + → Portainer).
2. Confirm entities such as `sensor.docker_containers_running`, `switch.docker_<container>`, and `button.docker_restart_<container>` exist.
3. Add the YAML snippet above to your dashboard (Edit Dashboard → Add Card → Manual → paste YAML).
4. Optionally tweak `running_color` or `not_running_color` to match your theme.

You now have an interactive Docker control panel that stays in sync with Portainer.

### Supported options

| Option | Required | Description |
| --- | --- | --- |
| `title` | No | Override the card header |
| `docker_overview` | No | Mapping of high-level stats to entity IDs |
| `running_color` | No | Global border/accent color for running containers and status pill |
| `not_running_color` | No | Global border/accent color for containers that are not running |
| `containers` | **Yes** | Array (or single object) describing each container |
| `containers[].name` | No | Friendly label (defaults to entity friendly name) |
| `containers[].status_entity` | Preferably | Entity whose state represents the container status |
| `containers[].control_entity` | Conditional | Entity that supports `turn_on`/`turn_off` (e.g. `switch`, `input_boolean`, `light`) to start/stop the container |
| `containers[].control_domain` | No | Override domain name when the entity uses a custom namespace |
| `containers[].restart_entity` | No | Entity to trigger a restart (`button`, `switch`, `script`, etc.) |
| `containers[].restart_domain` | No | Override domain for the restart entity |
| `containers[].switch_entity` | Legacy | Backwards compatible alias for `control_entity` |
| `containers[].running_color` | No | Per-container override for the running border/accent color |
| `containers[].not_running_color` | No | Per-container override for the not-running border/accent color |
| `containers[].running_states` | No | Custom list of states that count as “running” |
| `containers[].stopped_states` | No | Custom list of states that count as “stopped” |
| `containers[].tap_action` | No | Action to run when the row is tapped/clicked (standard Lovelace action object) |
| `containers[].hold_action` | No | Action to run on hold/long-press (supports the same syntax as `tap_action`) |
| `containers[].hold_delay` | No | Hold detection delay in milliseconds (defaults to 500) |

> Tip: `binary_sensor` entities are read-only. Use a `switch`, `input_boolean`, `light`, or similar domain for `control_entity` so the card can call `turn_on`/`turn_off`.

Color settings fall back to Home Assistant theme values (`var(--state-active-color)`, `var(--state-error-color)`) when omitted. Legacy keys `stopped_color` and `containers[].stopped_color` still map to the new not-running color options for backward compatibility.

## Styling and customization

- **Accent colors:** Override `running_color` and `not_running_color` globally, or set per-container overrides to highlight critical services.
- **Running/Total highlight:** The "Running / Total" overview pill turns the not-running color whenever the counts diverge—handy for spotting issues at a glance.
- **Theme alignment:** The card inherits typography, spacing, and background from your current Home Assistant theme, so it stays consistent without extra work.

## Exposing Docker to Home Assistant

If you rely on the Portainer integration you already have everything you need—just reference its entities in the card configuration above.

### Without Portainer

For environments that do not use Portainer, the example below shows how to surface equivalent entities with `command_line` sensors and `shell_command` helpers. Adjust container names to match your setup.

```yaml
# configuration.yaml or a dedicated package
sensor:
  - platform: command_line
    name: docker_containers_total
    command: "docker info --format '{{.Containers}}'"
    scan_interval: 60
  - platform: command_line
    name: docker_containers_running
    command: "docker info --format '{{.ContainersRunning}}'"
    scan_interval: 60
  - platform: command_line
    name: docker_containers_stopped
    command: "docker info --format '{{.ContainersStopped}}'"
    scan_interval: 60
  - platform: command_line
    name: docker_images
    command: "docker info --format '{{.Images}}'"
    scan_interval: 300
  - platform: command_line
    name: docker_version
    command: "docker version --format '{{.Server.Version}}'"
    scan_interval: 3600
  - platform: command_line
    name: docker_homeassistant_status
    command: "docker inspect -f '{{.State.Status}}' homeassistant"
    scan_interval: 30
  - platform: command_line
    name: docker_nodered_status
    command: "docker inspect -f '{{.State.Status}}' nodered"
    scan_interval: 30

binary_sensor:
  - platform: command_line
    name: docker_daemon_status
    command: "docker info > /dev/null && echo 'on' || echo 'off'"
    device_class: connectivity
    scan_interval: 30

shell_command:
  docker_start_homeassistant: "docker start homeassistant"
  docker_stop_homeassistant: "docker stop homeassistant"
  docker_restart_homeassistant: "docker restart homeassistant"
  docker_start_nodered: "docker start nodered"
  docker_stop_nodered: "docker stop nodered"
  docker_restart_nodered: "docker restart nodered"

script:
  docker_start_homeassistant:
    alias: Start Home Assistant container
    sequence:
      - service: shell_command.docker_start_homeassistant
  docker_stop_homeassistant:
    alias: Stop Home Assistant container
    sequence:
      - service: shell_command.docker_stop_homeassistant
  docker_restart_homeassistant:
    alias: Restart Home Assistant container
    sequence:
      - service: shell_command.docker_restart_homeassistant
  docker_start_nodered:
    alias: Start Node-RED container
    sequence:
      - service: shell_command.docker_start_nodered
  docker_stop_nodered:
    alias: Stop Node-RED container
    sequence:
      - service: shell_command.docker_stop_nodered
  docker_restart_nodered:
    alias: Restart Node-RED container
    sequence:
      - service: shell_command.docker_restart_nodered

```

To expose toggle-friendly entities, you can wrap the same shell commands in `command_line` switches or template buttons:

```yaml
switch:
  - platform: command_line
    switches:
      docker_homeassistant:
        friendly_name: Docker Home Assistant
        command_on: "docker start homeassistant"
        command_off: "docker stop homeassistant"
        command_state: "docker inspect -f '{{.State.Running}}' homeassistant"
        value_template: "{{ value == 'true' or value == 'running' }}"

button:
  - platform: template
    buttons:
      docker_restart_homeassistant:
        name: Restart Home Assistant container
        press:
          service: shell_command.docker_restart_homeassistant
```

Once the entities above are available, wire them into the card configuration as shown earlier.

## Troubleshooting

- **Custom card not found:** Ensure the resource URL is registered (`/hacsfiles/...` for HACS, `/local/...` for manual installs) and hard-refresh the browser.
- **Entities missing:** Double-check the Portainer integration is connected and that entity IDs in your YAML match the ones generated in Home Assistant.
- **Colors not updating:** Reload the dashboard after updating `running_color`/`not_running_color`, and confirm there are no typos in the CSS variables or hex codes.

## Development

- Distributed bundle lives in `docker-card.js`
- No build tooling required; the published file is ready-to-serve ES2021 JavaScript

## License

MIT © 2025
