**Version 0.2.8** published on 29.10.2024

- fixed issue with timezones in docker #3 https://github.com/muexxl/batcontrol_addon/issues/3


**Version 0.2.7** published on 21.10.2024

- fixed bug in force_charge() introduced with variable renaming in 0.2.0
- fixed bug in Forecast SolarAPI when providing empty apikeys

**Version 0.2.5** published on 18.10.2024

- fixed bug / Bad request to inverter when setting time of use
- code refactoring: renamed fronius module to inverter

**Version 0.2.3** published on 17.10.2024

- added optional parameter `apikey` in pv installations. This parameter allows you to use your own API Key if you are a forecast solar customer
- fixed bug if parameter api is missing

**Version 0.2.0** published on 17.10.2024

- included MQTT capabilities (deactivated per default)
- updated to batcontrol 0.2.1
- new parameter :`max_pv_charge_rate`. Limits the rate at which your PV System charges your battery
- renamed parameter `max_charge_rate` to `max grid charge rate`. Limits the maximum charging rate of your battery when charging from the grid.
- remove unused parameter `max_grid_power`

**Version 0.1.5** published on 09.02.2024

- introduced limit for log file
- added hints in configuration via translations/en.yaml

**Version 0.1.3** published on 16.11.2023

- added support for Awattar (DE+AT)
- enhanced documentation

**Version 0.1.2** published on 8.11.2023

- enhanced documentation

**Version 0.1.1** published on 7.11.2023

- Updated logo.
- Updated icon.

**Version 0.1.0** published on 7.11.2023

starting point
