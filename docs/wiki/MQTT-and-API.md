[Français](MQTT-et-API) · **English**

# MQTT and API

Default prefix `ocean2mqtt`; Home Assistant discovery on by default (`homeassistant` prefix).

| Topic | Content |
| --- | --- |
| `ocean2mqtt/bridge/state` | `online` / `offline` |
| `ocean2mqtt/bridge/info`, `bridge/devices` | Gateway, paired devices |
| `ocean2mqtt/bridge/request/<command>` | Requests; answer on `bridge/response/<command>` |
| `ocean2mqtt/bridge/event` | Pairing, new device, suggestion, Remote Commissioning |
| `ocean2mqtt/bridge/unknown` | Heard but unpaired transmitters |
| `ocean2mqtt/<device>` | JSON state, settings included |
| `ocean2mqtt/<device>/set` | Command or setting (`ON`, `{"mode": "comfort"}`, `{"position": 50}`, `{"led": "OFF"}`) |
| `ocean2mqtt/<device>/get` | Refresh state |
| `ocean2mqtt/<device>/action` | Button presses |

Requests: `info`, `devices`, `library`, `pairing/start` (`{"product": "nodon/sin-2-fp-01"}`), `device/rename|remove|set|get`, `device/settings/set`, `device/change_product`, `device/recom/<action>` (`status`, `ping`, `assign`, `detect`, `unlock`, `code`, `links`…), `migration/*`, `backup/create`, `telegrams`. Full list on the [French page](MQTT-et-API). The web interface uses the same API over WebSocket (`/api`).

## Home Assistant illuminance automation

With Home Assistant connected to the same MQTT broker, create an automation and open **Edit in YAML**. This example turns on a porch light when the garden sensor reports less than 50 lx. The sensor's state message contains a numeric `illuminance` field, for example `{"illuminance": 35}`.

```yaml
alias: Turn on the porch light when it is dark
triggers:
  - trigger: mqtt
    topic: ocean2mqtt/garden_light_sensor
conditions:
  - condition: template
    value_template: >-
      {% set state = trigger.payload_json | default({}) %}
      {% set lux = state.get('illuminance') if state is mapping else none %}
      {{ lux is number and lux is not boolean and lux < 50 }}
actions:
  - action: light.turn_on
    target:
      entity_id: light.porch
```

- Replace `ocean2mqtt/garden_light_sensor` with your sensor's state topic. Use your configured MQTT prefix and the device topic name or ID shown in the `topic` field of the `bridge/devices` response.
- Replace `50` with your threshold in lux and `light.porch` with your light's Home Assistant entity ID.

The condition is checked for each message on that topic, including any retained message delivered when Home Assistant subscribes.
