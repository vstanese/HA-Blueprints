# HA-Blueprints

My own collection of custom blueprints for Home Assistant

## Door Open Thermostat Control

### Description

This automation blueprint allows you to remember the state of the thermostat when a selected door is opened, turn off the thermostat after a set amount of time and send a recurring notification until the door is closed. Once the door is closed, resume the stat of the thermostat.

### Requirements

- Input select to be used for storing the thermostat state

### Installation

Click the below button to install the blueprint.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fraw.githubusercontent.com%2Fvstanese%2FHA-Blueprints%2Fmain%2Fdoor_thermostat_control.yaml)

Or import the code into a new file in the blueprints file.

```yaml
blueprint:
  name: Door Open Thermostat Control
  description: Turn off the thermostat when a door is opened and restore its state when the door is closed. Send repeated notifications if the door remains open.
  domain: automation
  input:
    door_sensor:
      name: Door Sensor
      description: The door sensor to monitor.
      selector:
        entity:
          domain: binary_sensor
    thermostat:
      name: Thermostat
      description: The thermostat to control.
      selector:
        entity:
          domain: climate
    notification_target:
      name: Notification Target
      description: Who to notify when the door is open/closed.
      selector:
        target:
          entity:
            domain: notify
    reminder_interval:
      name: Reminder Interval
      description: Time interval for the reminder notification when the door remains open (in minutes).
      default: 5
      selector:
        number:
          min: 1
          max: 60
          unit_of_measurement: minutes
    delay_before_reminder:
      name: Delay Before First Reminder
      description: Delay before sending the first reminder (in minutes).
      default: 15
      selector:
        number:
          min: 1
          max: 60
          unit_of_measurement: minutes
    thermostat_state_helper:
      name: Thermostat State Helper
      description: Input select to store the thermostat state.
      selector:
        entity:
          domain: input_select

variables:
  door_sensor: !input door_sensor
  thermostat: !input thermostat
  notification_target: !input notification_target
  reminder_interval: !input reminder_interval
  delay_before_reminder: !input delay_before_reminder
  thermostat_state_helper: !input thermostat_state_helper

trigger:

- platform: state
    entity_id: !input door_sensor
    from: "off"
    to: "on"

action:

- service: input_select.select_option
    target:
      entity_id: !input thermostat_state_helper
    data:
      option: "{{ states[thermostat].state }}"
- service: climate.turn_off
    target:
      entity_id: !input thermostat
- delay: "{{ delay_before_reminder | int }}:00"
- repeat:
      while:
        - condition: state
          entity_id: !input door_sensor
          state: "on"
      sequence:
        - service: notify.notify
          target: !input notification_target
          data:
            message: "The door is still open!"
        - delay: "{{ reminder_interval | int }}:00"

- wait_for_trigger:
  - platform: state
        entity_id: !input door_sensor
        from: "on"
        to: "off"
- service: notify.notify
    target: !input notification_target
    data:
      message: "The door is closed now."
- service: climate.set_hvac_mode
    target:
      entity_id: !input thermostat
    data:
      hvac_mode: "{{ states[thermostat_state_helper].state }}"

mode: restart
```
