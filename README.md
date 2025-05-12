# ☀️ Home Assistant Blueprints for GoodWe Photovoltaic Systems

This repository contains a set of Home Assistant blueprints designed to automate and optimize residential photovoltaic systems equipped with **GoodWe inverters**. These automations focus on smart battery management, solar forecast adaptation and energy optimization tailored to users with solar and battery installations in order to reduce energy costs. In its current state, the solution is most suitable for households that purchase electricity at a fixed rate and sell surplus power on the spot market.

---

## Prerequisites

### GoodWe Inverter
The main prerequisite for use this set of blueprints is solar system within Goodwe inverter, baterry system and Home Assistant **GoodWe Inverter** integration available at https://www.home-assistant.io/integrations/goodwe/ which is native Home Assistant integration. Most of the blueprints assume **experimental** version of that integration available at https://github.com/mletenay/home-assistant-goodwe-inverter as a HACS component. Details are mentioned in each blueprint documentation.

### Input Season
The blueprints expect that the current season is set to know which scenarios are usable. To define this input select, you as a user can use code below and copy it to **configuration.yaml** file.

```YAML
input_select:
  season:
    name: Season
    options:
      - Summer
      - Spring/Autumn
      - Winter
    icon: mdi:calendar
```

---

## 🔋 Auto Set DoD

Automatically adjusts the Depth of Discharge (DoD) of the battery based on today's solar production forecast and your typical energy consumption. It ensures conservative discharge during low production periods (e.g., winter), while allowing deeper discharge during expected sunny days. Standard winter DoD value is set when SoC (state of charge) drops down to that value. The **basic GoodWe Inverter integration** is expected.

### Entities
**GoodWe DoD Entity**
- Target Depth of Discharge Entity available in GoodWe integration

**PV Production Forecast**
- Sensor ensuring daily forecast of PV production
- Available from HA integrations like *Forecast.Solar* which is recommended approach. Eventually you may have access to another service as *Solcast* etc.

**Threshold for Changing DoD**
- Number specifiying the PV production prediction value that must be met to start automation (kWh)
- If Prediction is bigger than this value, automation starts, checks other conditions and sets DoD to the new value

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

Ensures that the battery is fully charged at least once a week during winter, if it hasn't been fully charged in the past 7 days. This can help preserve battery health and ensure readiness for colder periods. Triggered at a specific time and only if the production forecast and input season allow it. **Experimental GoodWe Inverter integration** is expected. This blueprint is usable in case you DON'T buy electricity on spot mode because of fixed trigger time.

### Entities
**Time**
- Time of start battery charging

**Count of 100% Battery State in a Week**
- Sensor which holds count of 100% battery charges in last 7 days based on history stats
- Available for example after defining in **configuration.yaml** file or you can have your own sensor

```YAML
sensor:
  - platform: history_stats
    name: Count of 100% Battery State in a Week
    entity_id: sensor.battery_state_of_charge
    state: 100
    type: count
    start: "{{ now().replace(hour=0, minute=0, second=0) - timedelta(days=7) }}"
    end: "{{ now() }}"
```

**PV Production Forecast**
- Sensor ensuring daily forecast of PV production
- Available from HA integrations like *Forecast.Solar* which is recommended approach. Eventually you may have access to another service as *Solcast* etc.

**GoodWe Charging Power**
- Target entity describing power of charging in eco charge mode
- Part of GoodWe Inverter integration

**GoodWe Final SoC Entity**
- Target entity describing final value of battery charging (%)
- Part of GoodWe Inverter integration

**GoodWe Inverter Mode**
- Target entity describing mode of the inverter
- Part of GoodWe Inverter integration

**GoodWe Soc Sensor**
- Sensor containing actual State of Charge of the battery
- Available directly in GoodWe Inverter integration

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-pv-system%2Fblob%2Fmain%2Fbattery_fully_charge.yaml)

---

## ❌ Disable Overflow

Disables electricity overflow when the spot price of electricity is negative (i.e., Export Limit is set to 0). Once the spot price becomes positive again, it restores the Export Limit to its original value. The **basic GoodWe Inverter integration** is expected.

### Entities
**Energy Spot Price**
- Actual energy spot price
- Data available from national electricity markets, often also as HA integrations (Nordpool, EPEX, Czech Energy Spot Prices, ...)

**GoodWe Export Limit**
- Target entity describing value of allowed export limit
- Available directly in GoodWe Inverter integration

**Export Limit Value (W)**
- Numeric value of the original export limit (W) which is set when the price is positive again
- Set this value if your Export Limit entity uses WATTS
- Value in % left unchanged
- If you don't set this value even if your Export Limit entity uses Watts, default value is 10000 W

**Export Limit Value (%)**
- Numeric value of the original export limit (%) which is set when the price is positive again
- Set this value if your Export Limit entity uses PERCENTAGE
- Value in Watts left unchanged
- If you don't set this value even if your Export Limit entity uses percentage, default value is 100 %

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-pv-system%2Fblob%2Fmain%2Fdisable_overflow.yaml)

---

## ⚡ Discharge Battery to the Grid When Price Peaks

Triggers eco discharge mode during the most expensive hour of the day, provided tomorrow's PV production is forecasted to be high enough and house consumption allows for discharging. The system calculates optimal discharging power to ensure battery will reach the defined DoD (depth of discharge) by 9:00 the next morning. Ideal during spring or summer when selling electricity to the grid is most profitable. **Experimental GoodWe Inverter integration** is expected. It is highly recommended to combine this blueprint with **Turn Off Eco Discharge Mode When Peak**.

### Entities
**Energy Spot Price**
- Actual energy spot price
- Data available from national electricity markets, often also as HA integrations (Nordpool, EPEX, Czech Energy Spot Prices, ...)

**Daily Max of Spot Price**
- Sensor with maximum energy spot price during the day
- Typically available via statistics or spot price integrations

**Minimum Energy Price (for kWh)**
- Threshold value for minimum price to allow discharge
- Prevents starting the automation when prices are too low to justify battery discharge

**PV Production Forecast Tomorrow**
- Forecast for PV production for the next day
- Available from HA integrations like *Forecast.Solar* which is recommended approach. Eventually you may have access to another service as *Solcast* etc.

**Prediction Threshold for Tomorrow**
- Minimal kWh value of tomorrow’s production forecast to start automation
- Helps ensure the battery will be sufficiently recharged after discharge

**GoodWe Soc Sensor**
- Sensor containing actual State of Charge of the battery
- Available directly in GoodWe Inverter integration

**GoodWe DoD Entity**
- Entity defining Depth of Discharge available in GoodWe integration

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

**Average Night Consumption**
- Value of average night consumption for estimating time of discharging battery
- Used to calculate if or how much the battery can be discharged in time
- Available for example after defining in **configuration.yaml** file or you can have your own sensor

```YAML
sensor:
  - platform: template
    sensors:
      nighttime_consumption:
        friendly_name: "Nighttime Consumption"
        unit_of_measurement: "W"
        value_template: >-
          {% set hour = now().hour %}
          {% if hour >= 20 or hour < 9 %}
            {{ states('sensor.house_consumption') | float(default=0) }}
          {% else %}
            0
          {% endif %}
  - platform: statistics
    unique_id: sensor.avg_night_consumption
    name: Average Night Consumption In Last 3 Days
    entity_id: sensor.nighttime_consumption
    state_characteristic: mean
    sampling_size: 28080
    max_age:
      hours: 85
```
  
**House Consumption**
- Sensor of actual house consumption
- Available directly in GoodWe Inverter integration

**GoodWe Discharging Power**
- Target entity describing power of discharging in eco discharge mode
- Part of GoodWe Inverter integration

**GoodWe Inverter Mode**
- Target entity describing mode of the inverter
- Part of GoodWe Inverter integration

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-goodwe-pv-control%2Fblob%2Fmain%2Fdischarge_battery_to_grid.yaml)

---

## 💰 Eco Discharge When Low Price at Noon

Checks conditions at specified time (PV prediction, energy prices) and sets eco discharge mode in the morning (moves the overflows from the afternoon to the morning) by script, then sets general mode again when price is low enough (block of 3 consecutive hours is the cheapest) or if the time to full charge of battery is too long. Ideal in spring or autumn period when energy prices are different during the day. **Experimental GoodWe Inverter integration** is expected. It is highly recommended to combine this blueprint with **Turn Off Eco Discharge Mode When Peak**.

### Entities
**Time**
- Time when automation start - checks PV production prediction and spot prices for the day

**PV Production Forecast**
- Sensor ensuring daily forecast of PV production
- Available from HA integrations like *Forecast.Solar* which is recommended approach. Eventually you may have access to another service as *Solcast* etc.

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

**Average PV Power Last 20 Minutes**
- Average value of the PV generation during last 20 minutes
- Available for example after defining in **configuration.yaml** file or you can have your own sensor

```YAML
sensor:
  - platform: statistics
    unique_id: sensor.avg_power
    name: "Average PV Production In Last 20 Minutes"
    entity_id: sensor.pv_power
    state_characteristic: mean
    sampling_size: 240
    max_age:
      minutes: 20
```

**Average House Consumption Last 20 Minutes**
- Average value of the house consumption during last 20 minutes
- Available for example after defining in **configuration.yaml** file or you can have your own sensor

```YAML
sensor:
  - platform: statistics
    unique_id: sensor.avg_consumption
    name: "Average Consumption In Last 20 Minutes"
    entity_id: sensor.house_consumption
    state_characteristic: mean
    sampling_size: 240
    max_age:
      minutes: 20
```

**Energy Spot Price**
- Actual energy spot price
- Data available from national electricity markets, often also as HA integrations (Nordpool, EPEX, Czech Energy Spot Prices, ...)

**Energy Spot Prices for the Day**
- Dictionary with hourly prices of energy
    - Czech - current_spot_electricity_hour_order
    - Nordpool - the only Nordpool entity, default name is "nordpool_<energy_scale>_<region>_<currency>_<some-numbers>"
- Available by one of the supported integrations - **Czech Energy Spot Prices**, **Nordpool**

**Actual Block of 3 Hours Energy Price Is Cheapest**
- Binary sensor defining the cheapest 3 hours block of energy prices
- Czech Energy Spot Prices - contains directly entity of that type
- Nordpool - You have to manually config sensor of that type in **configuration.yaml** file

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

**GoodWe Discharging Power**
- Target entity describing power of discharging in eco discharge mode
- Part of GoodWe Inverter integration

**GoodWe Inverter Mode**
- Target entity describing mode of the inverter
- Part of GoodWe Inverter integration

**GoodWe Soc Sensor**
- Sensor containing actual State of Charge of the battery
- Available directly in GoodWe Inverter integration

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-goodwe-pv-control%2Fblob%2Fmain%2Feco_discharge_when_low_price.yaml)

---

## ⏱️ Time to Use Overflow

This blueprint monitors battery state of charge, spot electricity price, and the balance between solar production and household consumption. When the battery is full, the spot price is below a defined threshold, and production significantly exceeds consumption, it turns on a switch to indicate it's the right time to use electricity overflow (e.g., to power controllable devices). The switch turns off again when conditions are no longer met. The **basic GoodWe Inverter integration** is expected.

### Entities
**GoodWe Soc Sensor**
- Sensor containing actual State of Charge of the battery
- Available directly in GoodWe Inverter integration

**Energy Spot Price**
- Actual energy spot price
- Data available from national electricity markets, often also as HA integrations (Nordpool, EPEX, Czech Energy Spot Prices, ...)

**Maximal Energy Price (for kWh)**
- Numeric value of threshold for the highest possible price for triggering automation (for kWh in currency of your energy price integration)
- If actual energy spot price is higher than this value, automation does not start and overflow is sold to the grid

**Average PV Power Last 20 Minutes**
- Average value of the PV generation during last 20 minutes
- Available for example after defining in **configuration.yaml** file or you can have your own sensor

```YAML
sensor:
  - platform: statistics
    unique_id: sensor.avg_power
    name: "Average PV Production In Last 20 Minutes"
    entity_id: sensor.pv_power
    state_characteristic: mean
    sampling_size: 240
    max_age:
      minutes: 20
```

**Average House Consumption Last 20 Minutes**
- Average value of the house consumption during last 20 minutes
- Available for example after defining in **configuration.yaml** file or you can have your own sensor

```YAML
sensor:
  - platform: statistics
    unique_id: sensor.avg_consumption
    name: "Average Consumption In Last 20 Minutes"
    entity_id: sensor.house_consumption
    state_characteristic: mean
    sampling_size: 240
    max_age:
      minutes: 20
```

**Target Switch Indicating Time to Use Overflow**
- If the switch is turned on, it is the right time to start using overflow.
- Available for example after defining in **configuration.yaml** file or you can have your own input boolean switch

```YAML
input_boolean:
  time_to_use_overflows:
    name: Time to Use Overflows
    icon: mdi:lightning-bolt-circle
```

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-goodwe-pv-control%2Fblob%2Fmain%2Ftime_to_use_overflow.yaml)

---

## ⚡ Turn Off Eco Discharge Mode When Peak

This automation turns off the inverter’s **eco discharge mode** when consumption exceeds production or battery power. It helps in preventing buying energy from the grid when energy consumption is higher than the available solar production or set battery power. When production is sufficient or consumption drops, the mode is automatically switched back to eco discharge. **Experimental GoodWe Inverter integration** and unchanged names of automations from this repository is expected. Using this blueprint is available with **Eco Discharge When Low Price at Noon** and **Discharge Battery to the Grid**.

### Entities
**House Consumption**
- Sensor of actual house consumption
- Available directly in GoodWe Inverter integration

**PV Power**
- Sensor of actual PV generation
- Available directly in GoodWe Inverter integration

**GoodWe Discharging Power**
- GoodWe entity describing power of discharging in eco discharge mode
- Part of GoodWe Inverter integration

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

**GoodWe Inverter Mode**
- Target entity describing mode of the inverter
- Part of GoodWe Inverter integration

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fjan-trnka%2Fhome-assistant-pv-system%2Fblob%2Fmain%2Fturn_off_eco_discharge_when_peak.yaml)

---

## 📦 Usage

1. Click the "Import Blueprint" button under any blueprint.
2. The button will open your Home Assistant UI and pre-load the blueprint import screen.
3. Customize entities and parameters as needed in your automation editor.

---
