## Rationale

In winter time there is usually not enough energy available from your residential pvinstallation to meet your demand.
So you need to get additional energy from the grid.
This software helps you to get the additional energy at the cheapest price by controlling your inverter in a smart way.

## Configuration Parameters

`timezone:` Europe/Berlin your time zone. not optional.  
`loglevel:` Verbosity of the add-on logs.

### `battery_control:`

`min_price_difference:` 0.05 in EUR/kwH # minimum price difference to justify charging your battery. This reflects the wear and tear of your battery as well as the electrical losses in the charging/ discharging cycle. 0.05 to 0.10 is a good range.

`always_allow_discharge_limit`: 0.90 # 0.00 to 1.00 above this SOC limit using energy from the battery is allowed regardless of other parameters. This value can be set lower to 1.00 if you want to ensure that always some capacity remains available in case the sun starts shining more than forecasted.  
`max_charging_from_grid_limit`: 0.90 # 0.00 to 1.00 charging from the grid is only allowed until this SOC limit. Above this limit only Solar Power may be used to increase the charge level.

### `inverter:`

`type`: fronius_gen24 #currently only fronius_gen24 supported  
`address`: 192.168.0.XX # the local IP of your inverter. needs to be reachable from the machine that runs batcontrol  
`user`: customer #customer or technician lowercase only!!  
`password`: YOUR-PASSWORD # password for your inverter
`max_pv_charge_rate`. Limits the rate at which your PV System charges your battery
`max grid charge rate`. Limits the maximum charging rate of your battery when charging from the grid. 

### `utility`:

`type`: your electricity provider. Currently supported: `tibber`, `awattar_at`, `awattar_de`  
`apikey`: Zz-YOURAPIKEYYOURAPIKEYXXXXX # only required for tibber , ignored for awattar  
`vat`: 0.20 # only required for awattar, ignored for tibber  
`fees`: 0.015 # only required for awattar, ignored for tibber  
`markup`: 0.03 # only required for awattar, ignored for tibber  
Tibber provides the personal prices directly via the API. So you need the apikey to get the price for YOUR home.  
Awattar only provides the EPEX Spot prices via the API. So the final price is calculated in the software using this formula:  
(Spotprice x (1 + markup) + fees) x (1 + VAT)

### `pvinstallations`:

`- name:` Haus #name of the installation. multiple installations can be added. Dont forget the hyphen in front of `name` to indicate that its a list  
`lat`: 48.4334480 Latitude in degree. Negative values indicate locations south of the equator.  
`lon`: 8.7654968 Longitude. Negative Values indicate locations in the West.  
`declination`: 32 #inclination toward horizon 0..90 degree 0=flat 90=vertical (e.g. wallmounted)  
`azimuth`: -90 # -90:East, 0:South, 90:West Range: -180..180  
`kWp`: 15.695 # power in kWp

### consumption_forecast:

`annual_consumption`: 4500 # total consumption in kWh p.a. . The load profile is scaled to meet this value. (optional)  
`load_profile`: load_profile.csv #name of the load profile file within the config folder. You can also add your own load profile. To do so you need to place a file in the running container in the directory /batcontrol/config/

## Algorithm

The addon gets three forecasts

1. electricty price forecast - from the Tibber API
1. your electricty production forecast based on the pv installation that you enter in config/batcontrol_config.yaml.
   KUDOS to forecast.solar!!!
1. your electricty consumption forecast based on the load profile. The load profile is scaled to the annual consumption that you provide in config/batcontrol_config.yaml

Based on the three forecasts AND the current state of charge (SOC) the software puts the inverter in one of the three modes:

**MODE 10 - DISCHARGE ALLOWED. (DEFAULT)**
This is the normal mode of the battery if there is sufficient energy available or energy is currently expensive. If the battery is full the surplus will be feed-in. If there is not enough energy coming from your pv installatation to meet your demand energy from the battery will be used.

**MODE 0 - AVOID DISCHARGE.**
If your consumption exceeds your current production energy from the grid will be used and the battery will not be discharged. This mode is used if prices are increasing in the future and the energy from the battery can be more efficiently used in the future

**MODE -1 - CHARGE FROM GRID.**
The battery is charged from the grid at a certain charge rate. This mode calculates the estimated required energy for future high priced hours. The objective is to charge the battery enough so that you do not need to consume energy from the grid in hours with high prices.

## Fronius Inverter control

Fronius inverters do not allow setting parameters via the solar web API.
So instead the software pretends to be a user of the web UI to modify the corresponding settings of your inverter.
This procedure has several disatvantages:

- usage of non documented APIs
- a firmware update of the inverter might cause disruptions
- unintended side effects (e.g. manual settings in the WebUI are overwritten to put the inverter in the defined Modes)

Do not use this software if you do not want those side effects. 