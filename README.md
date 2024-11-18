# Awtrix Dynamic Travel Time Display

This blueprint displays real-time travel time on an Awtrix device, with dynamic icons based on traffic conditions. You can set custom icons for high, medium, and normal traffic levels, and configure various display options like scroll speed and duration.

## Blueprint Details
- **Name**: Awtrix Dynamic Travel Time Display
- **Domain**: Automation
- **Description**: Display travel time with dynamic icons based on duration.

## Inputs

| Input | Description | Type | Default |
|-------|-------------|------|---------|
| **awtrix_device** | The Awtrix Device to display travel time. | Device | N/A |
| **toggle_helper** | Toggle to control display on/off state. | Entity | N/A |
| **travel_time_sensor** | Sensor providing travel time data. | Entity | N/A |
| **display_label** | Label text for the display. | Text | `Travel Time` |
| **high_traffic_icon** | Icon ID for high traffic condition. | Text | `28684` |
| **medium_traffic_icon** | Icon ID for medium traffic condition. | Text | `28686` |
| **normal_traffic_icon** | Icon ID for normal traffic condition. | Text | `28685` |
| **scroll** | Enable/Disable scrolling of the text. | Boolean | `true` |
| **scrollspeed** | Speed of scrolling. | Number (0-100) | `100` |
| **duration** | Duration to display the message (in seconds). | Number (0-300) | `0` |

## Triggers

This blueprint triggers the display update based on:
1. **Toggle Helper**: Turns the display on or off.
2. **Travel Time Sensor**: Updates the display when travel time changes.

## Variables

| Variable | Description |
|----------|-------------|
| **device_ids** | List of Awtrix devices specified. |
| **travel_time_entity** | Entity for travel time data. |
| **app_name** | Name of the application to display. |
| **label** | Display label for travel time. |
| **high_icon** | Icon ID for high traffic. |
| **medium_icon** | Icon ID for medium traffic. |
| **normal_icon** | Icon ID for normal traffic. |
| **display_icon** | Icon based on current traffic condition. |
| **display_text** | Text format with current travel time and status. |

## Actions

This blueprint has the following actions:
1. **Toggle ON**: When the toggle helper is turned on, display travel time with the appropriate icon and settings.
2. **Toggle OFF**: Clears the display when the toggle helper is turned off.
3. **Travel Time Change**: Updates the display text and icon when travel time changes, if the toggle is on.

## Example Usage

1. **Add the Blueprint**: Copy the YAML code into your Home Assistant's blueprints.
2. **Configure Inputs**: Specify your Awtrix device, travel time sensor, toggle helper, and customize the icons and display settings as desired.
3. **Create Automation**: Use this blueprint in an automation to dynamically update travel time information on your Awtrix device.
