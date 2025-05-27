## Modified MeteoAlarm integration with multiple alerts

This `MeteoAlarm` integration is basically the same from Home Assistant, except that it exposes multiple alerts in the sensor's attributes, while the official one only supports showing a single alert. The `MeteoAlarm` platform allows one to watch for weather alerts in Europe from [MeteoAlarm](https://www.meteoalarm.org) (EUMETNET). To use this binary sensor, you need the country and the province name from [MeteoAlarm](https://feeds.meteoalarm.org). Please note that it is crucial to write the country name exactly as it appears in the URL starting with https://feeds.meteoalarm.org/feeds/meteoalarm-legacy-atom-, including any hyphens used in the name. Failure to do so may result in errors or incorrect data.

The binary sensor state shows the warning messages, if applicable. The details are available as attribute.

## HACS Installation instructions

HACS (Home Assistant Community Store) is a popular way to easily add and manage custom integrations and plugins in Home Assistant. By using HACS, you'll be able to easily update the `MeteoAlarm` integration in the future.

### Prerequisites

Before you begin, make sure you have HACS installed and configured in your Home Assistant setup. If you haven't installed HACS yet, you can follow the [official HACS installation guide](https://hacs.xyz/docs/setup/download).

### Steps to Install

1. **Open HACS:**
   - In your Home Assistant dashboard, go to the sidebar and click on "HACS".

2. **Add the Custom Repository:**
   - Click on "Integrations" at the top.
   - In the bottom right corner, click the "+" icon to add a new repository.
   - In the window that appears, enter the following repository URL and select the "Integration" category:
     ```
     https://github.com/ov1d1u/meteoalarm
     ```
   - Click "Add" to include this custom repository.

3. **Install the Integration:**
   - Search for "MeteoAlarm" in the HACS integrations list after it appears.
   - Click on the "MeteoAlarm" integration.
   - Click "Install" to add it to your Home Assistant.

4. **Restart Home Assistant:**
   - After installation, you must restart Home Assistant to load the new integration.
   - Go to "Configuration" > "Settings" > "Server Controls".
   - Under "Server management", click "Restart" to apply changes.

## Manual Installation

Manual installation involves downloading the integration files and placing them in the appropriate directory in your Home Assistant setup. This method is suitable if you prefer not to use HACS or if you're running a Home Assistant environment where HACS is not supported.

### Steps to Install

1. **Download the Integration:**
   - Go to the [MeteoAlarm GitHub repository](https://github.com/ov1d1u/meteoalarm).
   - Click on the green "Code" button.
   - Select "Download ZIP" to download the repository as a ZIP file to your computer.

2. **Extract the ZIP File:**
   - Once downloaded, extract the contents of the ZIP file.

3. **Locate the Custom Components Directory:**
   - Find the `custom_components` directory in your Home Assistant configuration directory.
   - This is typically located at `/config/custom_components/` if you're using Home Assistant OS or Home Assistant Supervised.
   - If `custom_components` does not exist, you can create it.

4. **Copy the Integration Files:**
   - Rename the extracted folder from `meteoalarm-main` (or similar) to `meteoalarm`.
   - Copy this `meteoalarm` folder into the `custom_components` directory.

5. **Verify the Directory Structure:**
   - Ensure that the directory structure looks like this:
     ```
     /config
       /custom_components
         /meteoalarm
           __init__.py
           manifest.json
           ... (other integration files)
     ```

## Configuration

To enable this binary sensor, add the following lines to your {% term "`configuration.yaml`" %}:

```yaml
binary_sensor:
  - platform: meteoalarm
    country: "netherlands"
    province: "Groningen"
```

{% configuration %}
name:
  description: Binary sensor name
  required: false
  default: meteoalarm
  type: string
country:
  description: The fullname of your country in English format (lowercases)
  required: true
  type: string
province:
  description: The province
  required: true
  type: string
language:
  description: "The ISO code of your language, please be aware that this is only possible in the current country. So 'nl-NL' or 'nl-BE' is only possible in Netherlands or Belgium"
  required: false
  type: string
  default: "en-US"
{% endconfiguration %}

Example output

You will find an example below when the state is "on".

```yaml
attribution: Information provided by MeteoAlarm
alerts:
  - language: en-GB
    category: Met
    event: Severe forest-fire warning
    responseType: Monitor
    urgency: Immediate
    severity: Severe
    certainty: Likely
    effective: 2019-05-02T22:00:00+00:00
    onset: 2019-05-02T22:00:00+00:00
    expires: 2019-05-03T21:59:00+00:00
    senderName: Stig Carlsson
    headline: Orange forest-fire for Hedmark, Oppland
    description: High grass and heather fire hazard in areas without snow until significant amount of precipitation.
    Consequences: Vegetation is very easily ignited and very large areas may be affected.
    instruction: Be very careful with open fire. Follow the instructions from the local authorities. Emergency services should assess a necessary level of alertness.
    awareness_level: 3; orange; Severe
    awareness_type: 8; forest-fire
    unit_of_measurement:
    friendly_name: meteoalarm
    icon: mdi:alert
```

There are a few awareness levels:

- 2; yellow; Moderate
- 3; orange; Severe
- 4; red; High

Example automation

{% raw %}

```yaml
automation:
  - alias: "Alert me about weather warnings"
    triggers:
      - trigger: state
        entity_id: binary_sensor.meteoalarm
        from: "off"
    actions:
      - action: notify.notify
        data:
          title: "{{(state_attr('binary_sensor.meteoalarm', 'alerts')|first)['headline']}}"
          message: "{{(state_attr('binary_sensor.meteoalarm', 'alerts')|first)['description']}} is effective on {{(state_attr('binary_sensor.meteoalarm', 'alerts')|first)['effective']}}"
```

{% endraw %}

{% note %}
This integration is not affiliated with MeteoAlarm and retrieves data from the website by using the XML feeds. Use it at your own risk.
{% endnote %}