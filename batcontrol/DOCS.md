## Rationale

In winter time there is usually not enough energy available from your residential pvinstallation to meet your demand.
So you need to get additional energy from the grid.
This software helps you to get the additional energy at the cheapest price by controlling your inverter in a smart way.

## Customizing the load profile. (HA ADDON)

Step 1
download the default profile from https://raw.githubusercontent.com/muexxl/batcontrol/b08fa281efd63f5087d2da777e14f4403b5d3357/config/load_profile.csv

Step 2
edit the load profile according to your requirements.

Step 3
place the edited load profile in your home assistant instance using below path. You can use the vscode plugin to do so.
/addon_configs/HEXCODE_batcontrol/load_profile.csv

Step 4
Restart the addon

## Wiki

The Wiki of the underlying project provides further and up-to-date information about the project and its configuration parameters
https://github.com/muexxl/batcontrol/wiki

## Configuration Parameters

`timezone:` Europe/Berlin your time zone. not optional.
`loglevel:` Verbosity of the add-on logs.

### `battery_control:`

`min_price_difference:` 0.05 in EUR/kwH # minimum price difference to justify charging your battery. This reflects the wear and tear of your battery as well as the electrical losses in the charging/ discharging cycle. 0.05 to 0.10 is a good range.

`always_allow_discharge_limit`: 0.90 # 0.00 to 1.00 above this SOC limit using energy from the battery is allowed regardless of other parameters. This value can be set lower to 1.00 if you want to ensure that always some capacity remains available in case the sun starts shining more than forecasted.
`max_charging_from_grid_limit`: 0.90 # 0.00 to 1.00 charging from the grid is only allowed until this SOC limit. Above this limit only Solar Power may be used to increase the charge level.

### `battery_control_expert`:

`charge_rate_multiplier`: 1.1 # Increase (>1) calculated charge rate to compensate charge inefficencies.`soften_price_difference_on_charging`:
False # enable earlier charging based on a more relaxed calculation # future_price <= current_price-min_price_difference/soften_price_difference_on_charging_factor
`soften_price_difference_on_charging_factor`: 5
`round_price_digits`: 4 # round price to n digits after the comma

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

### `solar_forecast_provider`:

choose betwween fcsolarapi and solarprognose. Check https://github.com/muexxl/batcontrol/wiki/Solar-Forecast for details.

### `pvinstallations`:

`- name:` Haus #name of the installation. multiple installations can be added. Dont forget the hyphen in front of `name` to indicate that its a list
`lat`: 48.4334480 Latitude in degree. Negative values indicate locations south of the equator.
`lon`: 8.7654968 Longitude. Negative Values indicate locations in the West.
`declination`: 32 #inclination toward horizon 0..90 degree 0=flat 90=vertical (e.g. wallmounted)
`azimuth`: -90 # -90:East, 0:South, 90:West Range: -180..180
`kWp`: 15.695 # power in kWp
`apikey`: apikey for solarprognose or fcsolarapi if you are a customer and have a key
`algorithm`: only for solarprognose. optional.
`item`: only for solarprognose. optional.
`token`: only for solarprognose. optional.

### consumption_forecast:

`annual_consumption`: 4500 # total consumption in kWh p.a. . The load profile is scaled to meet this value. (optional)
`load_profile`: load_profile.csv #name of the load profile file within the config folder. You can also add your own load profile. To do so you need to place a file in the running container in the directory /batcontrol/config/

### `mqtt`:

Enables publishing of battery status and control messages via MQTT and integrates with Home Assistant through MQTT discovery.

`enabled`: true / false # enable or disable MQTT support
`logger`: true / false # if true, batcontrol log messages are also sent via MQTT
`broker`: localhost # MQTT broker hostname or IP
`port`: 1883 # MQTT broker port
`topic`: house/batcontrol # base topic for published MQTT messages
`username`: user # MQTT username
`password`: password # MQTT password
`retry_attempts`: 5 # number of reconnect attempts before failing (optional, default: 5)
`retry_delay`: 10 # seconds to wait between reconnect attempts (optional, default: 10)
`tls`: true / false # enable TLS encryption
`cafile`: /etc/ssl/certs/ca-certificates.crt # path to CA certificate
`certfile`: /etc/ssl/certs/client.crt # client certificate (if required)
`keyfile`: /etc/ssl/certs/client.key # client private key (if required)
`tls_version`: tlsv1.2 # TLS version (default: tlsv1.2)
`auto_discover_enable`: true / false # enable MQTT discovery for Home Assistant
`auto_discover_topic`: homeassistant # base topic for Home Assistant discovery messages

### `evcc`:

Allows integration with [evcc](https://docs.evcc.io/) to coordinate battery usage when charging an electric vehicle.
Batcontrol can pause/disallow battery discharge while EV charging is active.

`enabled`: true / false # enable or disable evcc integration
`broker`: localhost # MQTT broker used by evcc
`port`: 1883 # MQTT broker port
`status_topic`: evcc/status # topic for general evcc status
`loadpoint_topic`: # list of MQTT topics to monitor for charging status per loadpoint

- evcc/loadpoints/1/charging
- evcc/loadpoints/2/charging
  `block_battery_while_charging`: true / false # if true, disallow battery discharge while EV is charging
  `username`: user # MQTT username
  `password`: password # MQTT password
  `tls`: true / false # enable TLS encryption
  `cafile`: /etc/ssl/certs/ca-certificates.crt # path to CA certificate
  `certfile`: /etc/ssl/certs/client.crt # client certificate (if required)
  `keyfile`: /etc/ssl/certs/client.key # client private key (if required)
  `tls_version`: tlsv1.2 # TLS version (default: tlsv1.2)
  `battery_halt_topic`: evcc/site/bufferSoc # optional: topic that delivers SOC threshold below which the battery will be locked

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
