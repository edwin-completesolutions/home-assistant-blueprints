# 🌙 Outside Light Presence Evening · v1.0

### Smart Outdoor Lighting — Elevation-Based, Time-Aware

This Home Assistant **blueprint** provides intelligent control for your outdoor light using **solar elevation**, **motion**, **door sensors**, and a **manual button**.

It automatically dims the light in the evening, boosts to full brightness when motion or a door opens at night, and adapts behavior based on time of day — all without relying on fixed sunrise/sunset times.

---

## ⚙️ Features

* **Evening Window Dimming**  
  When it’s dark between noon and your chosen end time, the light turns on at a low brightness (default: 20%).

* **Motion & Door Boost**  
  Any motion or door activity while it’s dark triggers the light to go to full brightness (default: 100%) for a set duration (default: 2 minutes).

* **Smart Button Control**  
  A single Zigbee button adapts behavior based on the time of day:
  - **Evening**: toggles boost on/off
  - **Dark (outside evening window)**: toggles light on/off
  - **Daytime**: only turns the light off

* **Elevation-Based Logic**  
  Uses a solar elevation sensor and a configurable threshold (e.g., 0° or –3°) to determine when it’s dark — no reliance on fixed sunset times.

* **Seasonal Adaptation**  
  Works consistently year-round — the automation adapts to changing daylight patterns automatically.

---

## 🧩 Required Inputs

| Input                        | Description                                                                 |
|-----------------------------|-----------------------------------------------------------------------------|
| **Light to Control**        | A dimmable light that supports brightness control                          |
| **Motion Sensor**           | Triggers full brightness when motion is detected (anytime it’s dark)       |
| **Door Sensor**             | Triggers full brightness when the door opens (anytime it’s dark)           |
| **Button Device (Zigbee)**  | A Zigbee wall button or remote for manual override                         |
| **Timer Helper**            | A dedicated timer (created in Home Assistant) for boost duration            |
| **Evening Window End Time** | Time when the evening window ends (default: 22:00)                         |
| **Solar Elevation Sensor**  | A sensor providing the current solar elevation                             |
| **Low Brightness (%)**      | Brightness during the evening window *(default: 20 %)*                     |
| **High Brightness (%)**     | Brightness during boost *(default: 100 %)*                                 |
| **Boost Duration (s)**      | How long the boost lasts *(default: 120 s)*                                |
| **Elevation Threshold (°)** | Solar elevation below which it’s considered dark *(default: 0 °)*          |

---

## 🕹️ Behavior Summary

| Condition                                             | Light Behavior                                                                 |
|------------------------------------------------------|--------------------------------------------------------------------------------|
| **Daytime (sun high)**                               | Light off. Button press → off only.                                            |
| **Evening window (dark, noon to end time)**          | Light on at low brightness. Motion/door/button → boost to 100% for set time.   |
| **Night (dark after end time)**                      | Light off unless motion/door/button triggers a boost.                         |
| **Boost active**                                     | Light at 100%. After timer ends → returns to 20% if still in window, else off. |

---

## 🧠 Design Notes

* Built with **clear, traceable logic** using variables — no hardcoded entities.
* Fully **elevation-driven**, allowing fine-tuned sensitivity to twilight.
* Works with **any Zigbee button**, motion sensor, and dimmable light.
* Easily duplicated for multiple lights (e.g., front/back doors).

---

### 🪄 Tip

For smoother twilight detection, try setting  
`Elevation Threshold` = **–0.5° to –2°**.

---

**Author:** Home Automation v1.5 logic adapted for blueprint use  
**Version:** v1.0 (blueprint format)
