# Awtrix-Dynamic-Travel-Time-Display

Awtrix Dynamic Travel Time Display
This Home Assistant blueprint displays dynamic travel times on an Awtrix device, providing real-time travel information with traffic status icons. Travel times are sourced from a sensor (e.g., Waze Travel Time), and icons adjust based on the level of traffic, making it a convenient way to visualize commute details.

Features
Dynamic Icons: Displays different icons based on traffic conditions (High, Medium, Normal).
Real-Time Updates: Automatically updates travel time when the sensor changes.
Customizable Display: Adjust labels, scroll speed, duration, and more.
Awtrix Compatibility: Designed specifically for Awtrix matrix displays.
Blueprint Inputs
Input	Description	Default
awtrix_device	The Awtrix device (via MQTT integration)	-
toggle_helper	Helper entity to enable/disable the display	-
travel_time_sensor	Travel time sensor to display on the Awtrix	-
display_label	Label for the travel time display	"Travel Time"
high_traffic_icon	Icon ID for high traffic conditions	"28684"
medium_traffic_icon	Icon ID for medium traffic conditions	"28686"
normal_traffic_icon	Icon ID for normal traffic conditions	"28685"
scroll	Enable scrolling for the display	true
scrollspeed	Set scroll speed (0-100)	100
duration	Display duration in seconds (0 = infinite)	0
Setup Instructions
Download and import the blueprint in Home Assistant.
Configure your Awtrix device with MQTT and add it as an input.
Set up a travel time sensor (e.g., using the Waze Travel Time integration).
Select the icons for traffic conditions, display label, scroll settings, and more.
Example Use Case
This blueprint is ideal for those who commute regularly and want quick access to travel time updates with visual indicators. The setup displays the travel time and adjusts the icon based on traffic levels:

High Traffic: Shows a red icon to signal delays.
Medium Traffic: Uses an amber icon for moderate traffic.
Normal Traffic: Green icon to indicate light traffic.
YAML Configuration
Here’s the YAML code for the blueprint:

yaml
Copy code
blueprint:
  name: Awtrix Dynamic Travel Time Display
  description: Display travel time with dynamic icons based on duration
  domain: automation
  input:
    awtrix_device:
      name: Awtrix Device
      selector:
        device:
          integration: mqtt
          manufacturer: Blueforcer
    
    toggle_helper:
      name: Toggle Helper
      selector:
        entity:
          domain: input_boolean

    travel_time_sensor:
      name: Travel Time Sensor
      selector:
        entity:
          domain: sensor

    display_label:
      name: Display Label
      selector:
        text:
      default: "Travel Time"

    high_traffic_icon:
      name: High Traffic Icon ID
      selector:
        text:
      default: "28684"

    medium_traffic_icon:
      name: Medium Traffic Icon ID
      selector:
        text:
      default: "28686"

    normal_traffic_icon:
      name: Normal Traffic Icon ID
      selector:
        text:
      default: "28685"

    scroll:
      name: Scroll the notification?
      selector:
        boolean:
      default: true

    scrollspeed:
      name: Scroll Speed
      selector:
        number:
          min: 0
          max: 100
      default: 100

    duration:
      name: Duration (seconds)
      selector:
        number:
          min: 0
          max: 300
      default: 0

mode: queued
max: 60

trigger:
  - platform: state
    entity_id: !input toggle_helper
    to: "on"
    id: "On"
  - platform: state
    entity_id: !input toggle_helper
    to: "off"
    id: "Off"
  - platform: state
    entity_id: !input travel_time_sensor
    id: "Changes"

variables:
  device_ids: [!input awtrix_device]
  devices: >-
    {% macro get_device(device_id) %}
    {{ states((device_entities(device_id) | select('search','device_topic') | list)[0] | default('unknown')) }}
    {% endmacro %}
    {% set ns = namespace(devices=[]) %} {% for device_id in device_ids %}
    {% set device=get_device(device_id)|replace(' ','')|replace('\n','') %}
    {% set ns.devices = ns.devices + [ device ] %}
    {% endfor %} {{ ns.devices | reject('match','unavailable|unknown') | list }}
  travel_time_entity: !input travel_time_sensor
  app_name: "travel_time_display"
  label: !input display_label
  high_icon: !input high_traffic_icon
  medium_icon: !input medium_traffic_icon
  normal_icon: !input normal_traffic_icon
  display_icon: >-
    {% set time = states(travel_time_entity) | float(0) %}
    {% if time >= 65 %}{{ high_icon }}{% elif time >= 50 %}{{ medium_icon }}{% else %}{{ normal_icon }}{% endif %}
  display_text: >-
    {% set time = states(travel_time_entity) | float(0) %}
    {% set status = ' (High Traffic)' if time >= 65 else (' (Medium Traffic)' if time >= 50 else ' (Normal)') %}
    {{ label }} {{ time | round(0) }}min{{ status }}

action:
  - choose:
      - conditions:
          - condition: trigger
            id: "On"
        sequence:
          - repeat:
              for_each: "{{ devices }}"
              sequence:
                - service: mqtt.publish
                  data:
                    qos: 0
                    retain: false
                    topic: "{{repeat.item}}/custom/{{app_name}}"
                    payload: >-
                      {   
                        "text": "{{ display_text }}",
                        "background": [0, 0, 0],
                        "color": [255, 255, 255],
                        "icon": "{{ display_icon }}",
                        "textCase": 0,
                        "pushIcon": 2,
                        "rainbow": false,
                        "repeat": -1,
                        "duration": {{ duration }},
                        "lifetime": 0,
                        "noScroll": {{ not scroll }},
                        "scrollSpeed": {{ scrollspeed }}
                      }
                - service: mqtt.publish
                  data:
                    qos: 0
                    retain: false
                    topic: "{{repeat.item}}/switch"
                    payload: >-
                      {   
                        "name": "{{app_name}}"
                      }
      - conditions:
          - condition: trigger
            id: "Off"
        sequence:
          - repeat:
              for_each: "{{ devices }}"
              sequence:
                - service: mqtt.publish
                  data:
                    qos: 0
                    retain: false
                    topic: "{{repeat.item}}/custom/{{app_name}}"
                    payload: "{}"
      - conditions:
          - condition: and
            conditions:
              - condition: trigger
                id: "Changes"
              - condition: state
                entity_id: !input toggle_helper
                state: "on"
        sequence:
          - repeat:
              for_each: "{{ devices }}"
              sequence:
                - service: mqtt.publish
                  data:
                    qos: 0
                    retain: false
                    topic: "{{repeat.item}}/custom/{{app_name}}"
                    payload: >-
                      {   
                        "text": "{{ display_text }}",
                        "background": [0, 0, 0],
                        "color": [255, 255, 255],
                        "icon": "{{ display_icon }}",
                        "textCase": 0,
                        "pushIcon": 2,
                        "rainbow": false,
                        "repeat": -1,
                        "duration": {{ duration }},
                        "lifetime": 0,
                        "noScroll": {{ not scroll }},
                        "scrollSpeed": {{ scrollspeed }}
                      }
