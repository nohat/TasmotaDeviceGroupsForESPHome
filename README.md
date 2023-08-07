# Tasmota Device Groups for ESPHome
This repository is for an external component for ESPHome to add (limited) Tasmota device groups compatibility.  Since the majority of the code is taken directly from Tasmota, the license from Tasmota (GNU GPL v3.0) also follows.

# Configuration
Configuration for ESPHome is opt-in instead of automatic.  The reason for this is detached entities are not a special flag, and we don't want to attach to anything meant to be detached.

```yaml
external_components:
  - source: github://cossid/tasmotadevicegroupsforesphome@main
    components: [ device_groups ]
    refresh: 10 min

device_groups:
  - id: "testgroup1"     # Tasmota device group name
    switches:
      - gpio_switch      # ESPHome entity id
      - template_switch  # ESPHome entity id
    lights:
      - light_rgbww1     # ESPHome entity id
      - light_rgbww2     # ESPHome entity id
  - id: "testgroup2"     # Tasmota device group name
    switches:
      - gpio_switch2     # ESPHome entity id
```

### Supported:
* Power States
* Light States
  * On/Off
  * Brightness
  * Color channels

### Not yet supported
* Send/Recieve masking
* Fade/Transitions/Speed
* Schemes
* Commands (ESPHome doesn't have a direct equivalent)

### Command alternative
Since commands are not implemented, the suggested workaround is to make internal matching entites backed by templates to what you want to change, add that internal entity to your group, and set the desired state on that entity.

For example, if you have a switch with a button and would like to control a light entity, you could use:
```yaml
output:
  - platform: template
    id: dummy_output
    type: float
    write_action:
      - lambda: return;

light:
  - platform: rgbww
    id: internal_light
    color_interlock: true
    cold_white_color_temperature: 6500 K
    warm_white_color_temperature: 2700 K
    red: dummy_output
    green: dummy_output
    blue: dummy_output
    cold_white: dummy_output
    warm_white: dummy_output

button:
  - platform: template
    name: "Set light to red"
    on_press:
      - light.control:
          id: internal_light
          state: on
          red: 100%
          green: 0%
          blue: 0%

device_groups:
  - id: "light_group"    # Tasmota device group name
    lights:
      - internal_light   # ESPHome entity id
```

Button is just an example, but you could hook into any of the `on_` events for `binary_sensor`, `button`, `switch`, etc.

### Misc:
You may see a notice about blocking for too long.  This should not really be a problem, it is a generic ESPHome notice about performance.  Delays are mostly caused by waiting on network traffic.  The notice will look like this:
```
[W][component:204]: Component device_groups took a long time for an operation (0.15 s).
[W][component:205]: Components should block for at most 20-30ms.
```
