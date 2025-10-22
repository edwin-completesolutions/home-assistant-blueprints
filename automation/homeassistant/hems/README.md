# Home Assistant · EMS Battery Management

## Overview

This repository contains **automations** and **blueprints** that together form an **Energy Management System (EMS)** for Home Assistant.

The EMS continuously manages local battery storage to keep **grid exchange close to zero** — using as much **self-produced solar energy** as possible.  
It is designed for **local control**, not electricity price trading or cloud-based optimization.

---

## Core Principle — “Zero on the Meter”

The EMS monitors grid power and the expected solar production throughout the day to decide how the battery should behave.

- **Goal:** Keep the household’s net grid flow around **0 W**.  
- **Charge:** When solar generation exceeds consumption.  
- **Discharge:** Only when today’s remaining solar energy **cannot fully fit in the battery**.  
- **Hold:** When the remaining solar can still be stored without overflow.  
- **Avoid toggling:** Minimum step sizes and guard intervals prevent frequent switching.  
- **Once discharging starts:** The battery is allowed to discharge naturally — no short bursts or micro-adjustments.

---

## Battery Management Rules

| Parameter | Default | Description |
|------------|----------|-------------|
| **Minimum SoC** | 20 % | Protects battery health by avoiding deep discharge. |
| **Maximum SoC** | 90 % | Limits high-voltage stress during normal operation. |
| **Monthly Full Charge** | Yes | Once per month the battery is charged to 100 % for calibration and balancing. |
| **Step Size** | Configurable | Prevents oscillation and small, inefficient corrections. |
| **Forecast Source** | OpenMeteo Solar Forecast | Predicts remaining solar energy for the current day. |

All thresholds and parameters are **configurable through Home Assistant helpers**.

---

## Connectivity

The EMS communicates with the battery system via **MQTT**, ensuring **fully local operation** without vendor cloud dependence.

The **only cloud-based data** used is the **OpenMeteo Solar Forecast**, which estimates the remaining solar production for the day.  
This forecast determines whether to store or discharge energy based on available battery capacity.

---

## Hardware

The system uses a **Zendure SolarFlow 2400** with connected battery modules.  
In this setup it is **integrated through MQTT** for **local management** and monitoring.

- The battery is connected through a **Shelly Plug S Gen3**, which measures both **energy in (charging)** and **energy out (discharging)**.  
  This allows the battery to be configured in Home Assistant as **Home Battery Storage**, as the Zendure system itself does not expose MQTT sensors for total incoming and outgoing energy.  
  Charging and discharging limits are configured to stay safely **within the rated limits of the Shelly Plug S Gen3**, ensuring reliable long-term operation.  
- The Shelly plug also provides a **hardware failsafe**, allowing the EMS to disconnect power to the battery if required.  
- This approach makes the setup **simple, safe, and completely local**, without relying on any cloud services.

---

## Automations Overview

The EMS consists of three cooperating automations:

1. **Remaining Capacity Calculation**  
   - Computes how much energy is currently stored and how much capacity remains in the battery.  
   - Runs whenever the SoC or configured limits change.  
   - Provides core input values for the next automation.

2. **Discharge Allow Decision**  
   - Uses the **remaining capacity** together with the **forecasted solar production** for the rest of the day.  
   - Decides whether discharging should be allowed.  
   - If forecasted remaining solar would exceed available capacity, discharging is enabled; otherwise, it stays disabled.

3. **Zero-on-Meter Control**  
   - The main control loop that adjusts battery power through MQTT.  
   - Uses the outcomes of the first two automations to determine how much to charge or discharge.  
   - Reacts to live power data from the **P1 meter**, maintaining near-zero grid flow while respecting all limits.

Together, these three automations provide stable, autonomous, and fully local energy management.

---

## Philosophy

This project focuses on:
- **Local autonomy** — all logic runs in Home Assistant, no vendor cloud.  
- **Simplicity** — predictable and transparent operation.  
- **Efficiency** — maximum self-consumption of solar energy.  
- **Battery longevity** — careful SoC limits and minimal toggling.  
- **Safety** — integrated hardware cutoff via Shelly monitoring.

---

## License

Shared under the **MIT License** for educational and personal use.  
Always verify your helper and entity configurations before deployment.

---

**Project:** Home Assistant EMS Battery Management  
**Author:** Edwin F.  
**Goal:** *Zero on the Meter — Simple, Local, and Predictable.*
