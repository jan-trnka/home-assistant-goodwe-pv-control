# ☀️ Home Assistant Blueprints for GoodWe Photovoltaic Systems

This repository contains a set of Home Assistant blueprints designed to automate and optimize residential photovoltaic systems equipped with **GoodWe inverters**. These automations focus on smart battery management, solar forecast adaptation, and energy optimization tailored to users with solar and battery installations.

---

## 🔋 Auto Set DoD

Automatically adjusts the Depth of Discharge (DoD) of the battery based on today's solar production forecast and your typical energy consumption. It ensures conservative discharge during low production periods (e.g., winter), while allowing deeper discharge during expected sunny days.

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

[[![Import Blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?repository_url=https://raw.githubusercontent.com/jan-trnka/home-assistant-pv-system/blob/main/turn_off_eco_discharge.yaml)](https://github.com/jan-trnka/home-assistant-pv-system/blob/main/turn_off_eco_discharge_when_peak.yaml)

---

## 📦 Usage

1. Click the "Import Blueprint" button under any blueprint.
2. The button will open your Home Assistant UI and pre-load the blueprint import screen.
3. Customize entities and parameters as needed in your automation editor.

---
