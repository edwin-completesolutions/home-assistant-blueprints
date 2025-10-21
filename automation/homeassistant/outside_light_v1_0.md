---

# 🌙 Outside Light Presence Evening · v1.0

### Smart outdoor lighting control — fully elevation-driven

This Home Assistant **blueprint** intelligently manages your outdoor light using the **solar elevation**, motion, door activity, and a manual button.
It keeps the light dimly on in the evening, brightens temporarily on movement, and smoothly adapts between day and night — without relying on fixed sunrise/sunset times.

---

## ⚙️ Features

* **Automatic evening dimming**
  When it’s dark *after midday* but before 21:00, the light turns on at a low brightness (default 20 %).

* **Motion & door boost**
  Movement or door activity while it’s dark sets the light to full brightness (default 100 %) for a timed boost (default 120 s).

* **Button integration**
  The button behaves intuitively:

  * During the evening: toggles between dim and boost modes.
  * When dark outside the evening window: toggles light on/off via a 2-minute boost.
  * During daylight: turns the light off (if it’s on).

* **Fully elevation-based**
  Uses the solar elevation sensor and a configurable threshold (e.g. 0° or –3°) to decide when it’s dark — no dependency on sunset events.

* **Independent of real time**
  Logic works the same all year — dynamic darkness detection keeps it seasonal-proof.

---

## 🧩 Required Inputs

| Input                       | Description                                         |
| --------------------------- | --------------------------------------------------- |
| **Light Entity**            | The light to control                                |
| **Motion Sensor**           | Motion sensor to trigger boost                      |
| **Door Sensor**             | Door sensor to trigger boost                        |
| **Button Device (ZHA)**     | Manual toggle button                                |
| **Timer Entity**            | Timer helper for boost duration                     |
| **Solar Elevation Sensor**  | Numeric sensor with current solar elevation         |
| **Low Brightness (%)**      | Brightness for evening baseline *(default: 20 %)*   |
| **High Brightness (%)**     | Brightness for boost *(default: 100 %)*             |
| **Boost Duration (s)**      | How long the boost lasts *(default: 120 s)*         |
| **Elevation Threshold (°)** | Considered “dark” below this angle *(default: 0 °)* |

---

## 🕹️ Behavior Summary

| Condition                                           | Light Behavior                                                                                                |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| **Daytime**                                         | Light off. Button press → off only.                                                                           |
| **Evening window (dark, after noon, before 21:00)** | Light at low brightness (20 %). Motion, door, or button → boost to 100 % for 2 min.                           |
| **Dark, after 21:00**                               | Light off unless motion/door/button triggers a 2-min boost.                                                   |
| **Boost active**                                    | Timer keeps light at 100 %. When timer finishes → returns to 20 % if still in evening window, else turns off. |

---

## 🧠 Design Notes

* Built entirely with **variables** and clean, traceable logic — no hardcoded entities.
* Based purely on **elevation threshold**, adjustable for tuning darkness sensitivity.
* Works with any ZHA button, motion sensor, and light.
* Can easily be duplicated for multiple lights (e.g. front door / back door).

---

### 🪄 Tip

For smoother twilight handling, try setting
`Elevation Threshold` = **–0.5° to –2°**.

---

**Author:** Home Automation v1.5 logic adapted for blueprint use
**Version:** v1.0 (blueprint format)
