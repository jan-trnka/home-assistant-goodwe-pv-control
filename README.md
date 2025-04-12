# ☀️ Home Assistant Blueprints for GoodWe Photovoltaic Systems

This repository contains a set of Home Assistant blueprints designed to automate and optimize residential photovoltaic systems equipped with **GoodWe inverters**. These automations focus on smart battery management, solar forecast adaptation, and energy optimization tailored to users with solar and battery installations.

---

## Prerequisites

The main prerequisite for use this set of blueprints is **GoodWe Inverter** integration available at https://www.home-assistant.io/integrations/goodwe/ which is native Home Assistant integration. Most of the blueprints assume **experimental** version of that integration available at https://github.com/mletenay/home-assistant-goodwe-inverter as a HACS component. Details are mentioned in each blueprint documentation.

---

## 🔋 Auto Set DoD

Automatically adjusts the Depth of Discharge (DoD) of the battery based on today's solar production forecast and your typical energy consumption. It ensures conservative discharge during low production periods (e.g., winter), while allowing deeper discharge during expected sunny days. The basic GoodWe Inverter integration is expected.

### Entities
**GoodWe DoD Entity**
- Target Depth of Discharge Entity available in GoodWe integration

**PV Production Forecast**
- Sensor ensuring daily forecast of PV production
- Available from HA integrations like *Forecast.Solar* which is recommended approach. Eventually you may have access to another service as *Solcast* etc.

**Battery Capacity**
- Input number defining Battery Capacity
- It's up on user which value is set
- Available for example after defining in **configuration.yaml** file or you can have your own input number

```YAML
input_number:
  battery_capacity:
    name: Battery Capacity
    icon: mdi:battery
    min: 0
    max: 30
    unit_of_measurement: kWh
    step: 0.1
```

**Average Daily House Consumption**
- Sensor based on Final Daily House Consumption sensor which holds final value of consumption day before
- Contains average daily house consumpiton in last 7 days
- Available for example after defining in **configuration.yaml** file or you can have your own sensor

```YAML
sensor:
  # statistics sensor for average daily house consumption in last week
  - platform: statistics
    unique_id: sensor.avg_daily_consumption
    name: Average Daily Consumption
    entity_id: sensor.final_daily_house_consumption
    state_characteristic: mean
    sampling_size: 7
    max_age:
      days: 7
template:
  # template sensor for checking final house consumption at the end of the day
  - trigger:
      - trigger: time
        at: "23:59:55"
    sensor:
      - name: "Final Daily House Consumption"
        state: >
          {% set value = states('sensor.daily_house_consumption') | float(default=0) %}
          {{ value*1 | round(1, default=0) }}
        unit_of_measurement: "kWh"
```

**Winter DoD Value**
- Input number defining battery Depth of Discharge during winter time
- It's up on user which value is set
- Available for example after defining in **configuration.yaml** file or you can have your own input number

```YAML
input_number:
  winter_dod:
    name: Winter DoD Value
    icon: mdi:battery
    min: 0
    max: 100
    unit_of_measurement: "%"
    step: 1
```

**GoodWe SoC Sensor**
- Sensor containing actual State of Charge of the battery
- Available directly in GoodWe Inverter integration

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-pv-system%2Fblob%2Fmain%2Fauto_set_dod.yaml)

---

## 🔁 Fully Charge Battery Once a Week

Ensures that the battery is fully charged at least once a week during winter, if it hasn't been fully charged in the past 7 days. This can help preserve battery health and ensure readiness for colder periods. Triggered at a specific time and only if the production forecast and input season allow it.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-pv-system%2Fblob%2Fmain%2Fbattery_fully_charge.yaml)

---

## ❌ Disable Overflow

Disables electricity overflow when the spot price of electricity is negative (i.e., Export Limit is set to 0). Once the spot price becomes positive again, it restores the Export Limit to its original value.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-pv-system%2Fblob%2Fmain%2Fdisable_overflow.yaml)

---

## ⚡ Turn Off Eco Discharge Mode When Peak

This automation turns off the inverter’s eco discharge mode when consumption exceeds production. It helps in preventing excessive battery discharge when energy consumption is higher than the available solar production.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-pv-system%2Fblob%2Fmain%2Fturn_off_eco_discharge_when_peak.yaml)

---

## 📦 Usage

1. Click the "Import Blueprint" button under any blueprint.
2. The button will open your Home Assistant UI and pre-load the blueprint import screen.
3. Customize entities and parameters as needed in your automation editor.

---
