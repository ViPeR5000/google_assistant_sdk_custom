# Google Assistant SDK Custom integration for Home Assistant

[![hacs_badge](https://img.shields.io/badge/HACS-Default-41BDF5.svg)](https://github.com/hacs/integration)

# About

This integration is a copy of [Google Assistant SDK integration](https://www.home-assistant.io/integrations/google_assistant_sdk/) with the following additional features:

- HTML responses as text from commands to Google Assistant

## Why aren't these changes in the core Google Assistant SDK integration?

Due to a [bug](https://github.com/googlesamples/assistant-sdk-python/issues/391) in the Google Assistant API,
not all responses contain text, especially for home control commands, like turn on the lights.
The workaround, per the linked bug, is to enable HTML output and then parse the HTML for the text.
Because the core integrations are [not allowed](https://github.com/home-assistant/architecture/blob/master/adr/0004-webscraping.md) to parse HTML,
this custom integration is needed.

# Installation

## HACS

1. [Add](http://homeassistant.local:8123/hacs/integrations) custom integrations repository: <https://github.com/tronikos/google_assistant_sdk_custom>
2. Select "Google Assistant SDK Custom" in the Integration tab and click download
3. Restart Home Assistant
4. Enable the integration

## Manual

1. Copy directory `custom_components/google_assistant_sdk_custom` to your `<config dir>/custom_components` directory
2. Restart Home-Assistant
3. Enable the integration

## Enable the integration

1. Go to [Settings / Devices & Services / Integrations](http://homeassistant.local:8123/config/integrations). Click **+ ADD INTEGRATION**
2. Search for "Google Assistant SDK Custom" and click on it

# Examples

## Nest Guard

Following example allows controlling Nest Guard and getting its status in a text helper.
A similar approach, especially the automation that polls the status, can be used for other devices,
for which a template, e.g. [template light](https://www.home-assistant.io/integrations/light.template/), might make more sense. 
Here a [Template Alarm Control Panel](https://www.home-assistant.io/integrations/alarm_control_panel.template/) wouldn't work
because Google Assistant doesn't allow disarming Nest Guard.

Create a text [helper](http://homeassistant.local:8123/config/helpers) with:

```yaml
Name: Nest Guard Status
Icon: mdi:shield-home
Entity ID: input_text.nest_guard_status
```

Create following [scripts](http://homeassistant.local:8123/config/script/dashboard):

```yaml
nest_guard_refresh:
  alias: "Nest Guard: Refresh"
  sequence:
    - service: google_assistant_sdk_custom.send_text_command
      data:
        command: what is the status of nest guard
      response_variable: response
    - service: input_text.set_value
      data:
        value: "{{ response.responses[0].text }}"
      target:
        entity_id: input_text.nest_guard_status
  mode: single
  icon: mdi:shield-refresh
nest_guard_away:
  alias: 'Nest Guard: Away'
  sequence:
  - service: google_assistant_sdk_custom.send_text_command
    data:
      command: Set Nest Guard to away and guarding
  - service: script.nest_guard_refresh
    data: {}
  mode: single
  icon: mdi:shield-lock
nest_guard_home:
  alias: 'Nest Guard: Home'
  sequence:
  - service: google_assistant_sdk_custom.send_text_command
    data:
      command: Set Nest Guard to home and guarding
  - service: script.nest_guard_refresh
    data: {}
  mode: single
  icon: mdi:shield-home
```

Create [automation](http://homeassistant.local:8123/config/automation/dashboard):

```yaml
alias: "Nest Guard: status"
description: ""
trigger:
  - platform: time_pattern
    minutes: "10"
condition: []
action:
  - service: script.nest_guard_refresh
    data: {}
mode: queued
max: 10
```

Add the following Entities card in the lovelace dashboard:

```yaml
type: entities
entities:
  - input_text.nest_guard_status
footer:
  type: buttons
  entities:
    - entity: script.nest_guard_home
      show_icon: true
      show_name: true
    - entity: script.nest_guard_away
      show_icon: true
      show_name: true
    - entity: script.nest_guard_refresh
      show_icon: true
      show_name: true
```

## Other examples

(You will have to replace `google_assistant_sdk.send_text_command` with `google_assistant_sdk_custom.send_text_command`).

- [360 vacuum](https://community.home-assistant.io/t/360-s6-vacuum-robot/124990/29)
- [robot mowers](https://github.com/tronikos/google_assistant_sdk_custom/issues/2#issuecomment-1473697969)
- [Hisense air conditioning](https://github.com/tronikos/google_assistant_sdk_custom/issues/3#issuecomment-1520227069)
