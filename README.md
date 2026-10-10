<!-- prettier-ignore -->
# Ecoflow PowerOcean
[![GitHub release](https://img.shields.io/github/release/KieranHolroyd/powerocean?include_prereleases=&sort=semver&color=blue)](https://github.com/KieranHolroyd/powerocean/releases/)
[![issues - powerocean](https://img.shields.io/github/issues/KieranHolroyd/powerocean)](https://github.com/KieranHolroyd/powerocean/issues)
[![GH-last-commit](https://img.shields.io/github/last-commit/KieranHolroyd/powerocean?style=flat-square)](https://github.com/KieranHolroyd/powerocean/commits/main)
[![hacs_badge](https://img.shields.io/badge/HACS-Custom-41BDF5.svg)](https://hacs.xyz/docs/faq/custom_repositories/)
[![Validate](https://github.com/KieranHolroyd/powerocean/actions/workflows/validate.yml/badge.svg)](https://github.com/KieranHolroyd/powerocean/actions/workflows/validate.yml)

[Home Assistant](https://home-assistant.io/) custom integration that reads live data from an EcoFlow PowerOcean system (inverter, batteries, PowerPulse wallbox, PowerGlow heating rod) through the EcoFlow cloud.

This is a fork of [niltrip/powerocean](https://github.com/niltrip/powerocean), translated to English and packaged as its own HACS custom repository. All the original work is niltrip's; see [Credits](#credits).

## Prerequisites

- An EcoFlow account with the PowerOcean system added (the same login you use in the EcoFlow app or at https://portal.ecoflow.com).
- The serial number of your PowerOcean inverter (shown in the EcoFlow app).
- Home Assistant 2025.2.4 or newer.

## Installation

### HACS (recommended)

[![Open your Home Assistant instance and open this repository inside HACS.](https://my.home-assistant.io/badges/hacs_repository.svg)](https://my.home-assistant.io/redirect/hacs_repository/?owner=KieranHolroyd&repository=powerocean&category=integration)

Or add it by hand:

1. In Home Assistant open **HACS**, then the **⋮** menu in the top right, then **Custom repositories**.
2. Repository: `https://github.com/KieranHolroyd/powerocean`, type: **Integration**, then **Add**.
3. Search HACS for **Ecoflow PowerOcean**, open it and select **Download**.
4. Restart Home Assistant.

> [!IMPORTANT]
> This fork uses the same `powerocean` domain as niltrip's version in the HACS default store, so only one of them can be installed at a time.
> If you already have niltrip's version, remove it in HACS first (**HACS → Ecoflow PowerOcean → ⋮ → Remove**), then install this one and restart.
> Your existing PowerOcean config entry, entities and history stay in place because the domain and unique IDs are unchanged.

### Manual

1. Download `powerocean.zip` from the [latest release](https://github.com/KieranHolroyd/powerocean/releases/latest).
2. Extract it into `<config>/custom_components/powerocean/`.
3. Restart Home Assistant.

## Configuration

Go to **Settings → Devices & services → Add integration → Ecoflow PowerOcean** and follow the flow.

![step 1](documentation/setup_step_01.png)
![step 2](documentation/setup_step_02.png)
![step 3](documentation/setup_step_03.png)

## Sensors

Sensors are registered to the device as `sensor.{device_name}_{sensor_name}`, with `sensor_name` as the friendly name.
Each sensor also carries these attributes:

- Product description: the internal EcoFlow field name, or an English label for common fields
- Internal unique ID: `{serial}_{sensor_name}` or `{serial}_{report}_{sensor_name}`
- Device name: `{serial}`
- Product serial: serial number of the device the value belongs to

Batteries, the PowerPulse wallbox and the PowerGlow heating rod show up as their own devices, keyed by serial number:

![integration](documentation/integration.PNG)
![powerpulse](documentation/powerpulse.png)
![powerglow](documentation/powerglow.png)

### Calculated solar total

`mpptPv_pwrTotal` is calculated by the integration from the individual MPPT strings:

![sensor](documentation/mpptPv_pwrTotal.PNG)

### Which values update, and which do not

The EcoFlow cloud does not refresh all values at the same rate. Some of them keep returning the same number for hours. The integration reports what the API hands it, so a flat line is not a bug in this component.

Measured on one system over 72 hours (three-phase PowerOcean, one battery, release 2026.03.04):

| sensor | report | distinct values in 72 h |
|---|---|---|
| `mpptPwr`, `bpPwr` | `data` | 1 (always 0 W) |
| `pv1Pwr`, `pv2Pwr`, `pvInvPwr` | `ENERGY_STREAM_REPORT` | 1 (always 0 W) |
| `sysGridPwr`, `sysLoadPwr` | `data` | 4 |
| `todayElectricityGeneration` | `data` | about one per hour |
| `pcsMeterPower`, `pcsActPwr`, `mpptPv_pwrTotal` | `EMS_HEARTBEAT` | more than 2100 |

On 2026-08-21 at 13:16 `mpptPwr` read 0 W while `mpptPv_pwrTotal` read 4238 W. Both describe the same solar power, and both came out of the same response.

Users report in [niltrip/powerocean#34](https://github.com/niltrip/powerocean/issues/34) and [niltrip/powerocean#28](https://github.com/niltrip/powerocean/issues/28) that the stale values start moving again while the EcoFlow app or web portal is open.

For a dashboard, use the values from `EMS_HEARTBEAT`:

- grid: `pcsMeterPower` (positive = import, negative = export) rather than `sysGridPwr`
- solar: `mpptPv_pwrTotal` rather than `mpptPwr`, `pv1Pwr` or `pvInvPwr`
- battery: `emsBpPower` rather than `bpPwr`
- house: `housePower`, which is calculated from the three values above

### Energy dashboard

![dashboard](documentation/dashboard.PNG)

The EcoFlow cloud does not expose lifetime grid import/export counters through the endpoint this integration uses. Until the integration provides them itself, [`config/configuration.yaml`](config/configuration.yaml) shows how to build kWh sensors from the power sensors with template sensors and Home Assistant's [Integral (Riemann sum)](https://www.home-assistant.io/integrations/integration/) helper.

## Troubleshooting

Turn on debug logging for this integration during setup. Once everything works you can remove it again:

```yaml
logger:
  default: warn
  logs:
    custom_components.powerocean: debug
```

Please report problems at https://github.com/KieranHolroyd/powerocean/issues and attach the debug log.

## Credits

- [niltrip](https://github.com/niltrip), who wrote and maintains the original [niltrip/powerocean](https://github.com/niltrip/powerocean) integration, and everyone who contributed to it.
- The original was inspired by [tolwi/hassio-ecoflow-cloud](https://github.com/tolwi/hassio-ecoflow-cloud) and started from [evercape/hass-resol-KM2](https://github.com/evercape/hass-resol-KM2).
- The Home Assistant team and community.
