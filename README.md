Awtrix Dynamic Travel Time Display
This blueprint displays real-time travel time on an Awtrix device, with dynamic icons based on traffic conditions. You can set custom icons for high, medium, and normal traffic levels, and configure various display options like scroll speed and duration.

Blueprint Details
Name: Awtrix Dynamic Travel Time Display
Domain: Automation
Description: Display travel time with dynamic icons based on duration.
Inputs
Input	Description	Type	Default
awtrix_device	The Awtrix Device to display travel time.	Device	N/A
toggle_helper	Toggle to control display on/off state.	Entity	N/A
travel_time_sensor	Sensor providing travel time data.	Entity	N/A
display_label	Label text for the display.	Text	Travel Time
high_traffic_icon	Icon ID for high traffic condition.	Text	28684
medium_traffic_icon	Icon ID for medium traffic condition.	Text	28686
normal_traffic_icon	Icon ID for normal traffic condition.	Text	28685
scroll	Enable/Disable scrolling of the text.	Boolean	true
scrollspeed	Speed of scrolling.	Number (0-100)	100
duration	Duration to display the message (in seconds).	Number (0-300)	0
Triggers
This blueprint triggers the display update based on:

Toggle Helper: Turns the display on or off.
Travel Time Sensor: Updates the display when travel time changes.
Variables
Variable	Description
device_ids	List of Awtrix devices specified.
travel_time_entity	Entity for travel time data.
app_name	Name of the application to display.
label	Display label for travel time.
high_icon	Icon ID for high traffic.
medium_icon	Icon ID for medium traffic.
normal_icon	Icon ID for normal traffic.
display_icon	Icon based on current traffic condition.
display_text	Text format with current travel time and status.
Actions
This blueprint has the following actions:

Toggle ON: When the toggle helper is turned on, display travel time with the appropriate icon and settings.
Toggle OFF: Clears the display when the toggle helper is turned off.
Travel Time Change: Updates the display text and icon when travel time changes, if the toggle is on.
Example Usage
Add the Blueprint: Copy the YAML code into your Home Assistant’s blueprints.
Configure Inputs: Specify your Awtrix device, travel time sensor, toggle helper, and customize the icons and display settings as desired.
Create Automation: Use this blueprint in an automation to dynamically update travel time information on your Awtrix device.
YAML Code
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
