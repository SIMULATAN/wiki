---
title: Zendure
description: 
published: true
date: 2026-10-22T13:02:23.709Z
tags: 
editor: markdown
dateCreated: 2026-10-21T12:04:40.826Z
---

# References
- Properties Explanation: https://github.com/Zendure/zenSDK/blob/main/docs/openapi.yaml

# Setup Architecture
Although most facts stated below will apply to generic setups, it is important to note how this information came to be and is applied in my home.

The Zendure SolarFlow + Battery are a secondary system. It runs in parallel to the existing primary system. Hence, it is not connected to any PV panels - the SF only exists to physically integrate the Battery.

# Relevant Parameters
## Zendure
### `inputLimit`
> AC charge power limit (W).

Limits how much Power the Battery charges with. Keep low enough to prevent charging from the grid.

### `outputLimit`
> Output power limit (W).

Limits how much Power the Battery discharges with. Maintain a healthy level to keep the primary Battery around the expected percentage.

### `socSet`
> Target state of charge (%). Range: 70-100.

Limits the maximum battery level. Lower values preserve the battery health for longer while nearing 100 leads to faster degradation.

## Options & Algorithm Config
### Winter Mode
Sets the min SoC charge to 80% (must not discharge below that) and the input limit to the maximum available.
Any auto-balancing is disabled.

### Summer Mode
Charge using excess.
Once the main battery is below 75%, discharge as needed. To fix bouncing/yo-yo, it may be worth using a tracker variable:

```
if (charge <= 75%)
	useBalancing = true
else if (charge >= 80%)
	useBalancing = false
```

After that variable is updated, the logic may run, gated behind it.

# Auto Balancing Setup
Make sure to enable the `smartMode` Switch (=> `smartMode: 1`) - otherwise, frequent updates to the parameters will be flushed to the Flash Memory, thus negatively impacting the device.