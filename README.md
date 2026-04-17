# HomeAssisnan_-Zombie-Radar-Battery-Fail-Monitor
## ⚠️ The Problem: The Battery Percentage Lie

Most Home Assistant users rely on battery percentage thresholds (e.g., `if battery < 20%`) to know when to replace batteries. This is an **Indirect Measurement** and it is inherently flawed for two reasons:

1. **The Alkaline "Voltage Sag":** Devices report battery % while idle. However, transmitting a Zigbee packet requires a massive amperage spike. A weak alkaline battery might report 35% at rest, but when asked to fire the radio, the voltage instantly collapses (Voltage Sag), causing the radio to crash. The order is ignored, yet HA still happily reports 35% battery.
2. **The 1.5V Lithium Trap:** Rechargeable 1.5V Lithium batteries have internal regulators. They output a perfectly flat 1.5V (reporting 100% or 80% to HA) until they are completely empty, then instantly drop to 0V. 

In both cases, the device enters the **"Zombie Phase"**: It has enough power to read a physical button click, but is completely deaf to the Zigbee network. If left in this state, it will eventually drop off the network entirely, often corrupting its NVRAM and requiring a frustrating **manual re-pairing**.

A particular one pro to this situation is ThirdReality gen3 battery operated switch. Where manual re-pairing is required once the unavailable tag appears in homeassistant. 

## 💡 The Solution: Vision by Intent (Empirical Monitoring)

Stop trusting the battery sensor. Trust the **Action**. 
This automation acts as a "Radar" by performing Empirical Verification:
1. It intercepts every `call_service` event on the Home Assistant Event Bus targeting your switches.
2. It calculates the **Intended State** (`turn_on` = on, `toggle` = opposite of current).
3. It waits exactly **8 seconds** (to account for Zigbee mesh routing latency).
4. It checks if the actual physical state matches the Intended State. 
5. If the state didn't change, the device failed the physical test (Zombie Phase). It instantly notifies you to swap the battery *before* the device loses its pairing key.
