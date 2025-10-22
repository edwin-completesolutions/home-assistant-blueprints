# Home Assistant · EMS Battery Management

## Overview

This repository contains **automations** and **blueprints** that together form an **Energy Management System (EMS)** for Home Assistant.

The EMS continuously manages local battery storage to keep **grid exchange close to zero** — using as much **self-produced solar energy** as possible.  
It is designed for **local control**, not electricity price trading or external optimization services.

---

## Core Principle — “Zero on the Meter”

The EMS monitors grid power and expected solar production throughout the day to decide how the battery should behave.

- **Goal:** Keep the household’s net grid flow around **0 W**.  
- **Charge:** When solar generation exceeds consumption.  
- **Discharge:** Only when today’s remaining solar energy **cannot fully fit in the battery**.  
- **Hold:** When remaining solar can still be stored without overflow.  
- **Avoid toggling:** Minimum step sizes and guard intervals prevent frequent switching.  
- **Once discharging starts:** The battery is allowed to run down naturally — no micro-adjustments.

---

## Battery Management Rules

| Parameter | Default | Description |
|------------|----------|-------------|
| **Minimum SoC** | 20 % | Protects battery health by avoiding deep discharge. |
| **Maximum SoC** | 90 % | Limits high-voltage stress for daily use. |
| **Monthly Full Charge** | Yes | Once per month the battery is charged to 100 % for calibration and balancing. |
| **Step Size** | Configurable | Prevents oscillation and small, inefficient corrections. |
| **Forecast Source** | OpenMeteo Solar Forecast | Predicts remaining solar energy for the current day. |

All thresholds and parameters are **configurable via Home Assistant helpers**.

---

## Connectivity

The EMS communicates with the battery system via **MQTT**, ensuring **fully local operation** without vendor cloud dependence.

The **only cloud-based data** used is the **OpenMeteo Solar Forecast**, which estimates the remaining solar production for the day.  
This forecast determines whether to store or discharge energy based on available battery capacity.

---

## Philosophy

This project focuses on:
- **Local autonomy** — no cloud control, all logic within Home Assistant.  
- **Simplicity** — predictable and transparent operation.  
- **Efficiency** — maximum use of self-generated solar energy.  
- **Battery longevity** — careful SoC limits and minimal toggling.

---

## License

Shared under the **MIT License** for educational and personal use.  
Always verify your helper and entity configurations before deployment.

---

**Project:** Home Assistant EMS Battery Management  
**Author:** Edwin F.  
**Goal:** *Zero on the Meter — Simple, Local, and Predictable.*
