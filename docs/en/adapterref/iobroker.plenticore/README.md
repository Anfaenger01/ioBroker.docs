![Logo](admin/plenticore.png)

[![NPM version](https://img.shields.io/npm/v/iobroker.plenticore.svg)](https://www.npmjs.com/package/iobroker.plenticore)
[![Build Status](https://travis-ci.org/StrathCole/ioBroker.plenticore.svg?branch=master)](https://travis-ci.org/StrathCole/ioBroker.plenticore)
[![License](https://img.shields.io/badge/license-MIT-blue.svg?style=flat)](https://github.com/StrathCole/iobroker.plenticore/blob/master/LICENSE)

# ioBroker.plenticore


An ioBroker adapter for KOSTAL Plenticore Plus inverter (i. e. Plenticore Plus 8.5)

This adapter uses the internal web interface of the inverter to access the properties and settings of your inverter and connected devices (e. g. battery or smart energy meter). To use the adapter you need to have the ioBroker instance connected to the network your KOSTAL Plenticore is in.

This adapter is NOT an official product of KOSTAL neither it is supported or endoresed by KOSTAL. It is a private project still in early development state, so use at your own risk!

## Config

Set the IP address of your inverter (e. g. 192.168.0.23) and your password you are using to connect as plant owner to the inverter's web interface. The polling interval is in milliseconds (i. e. 10000 is 10 seconds).

## Adapter 

The adapter does not use screenscraping. It uses the same REST API that the web interface does. There might be (lots of) features that are not (yet) used by the adapter.

### Why not (simply) use modbus?

The inverter has modbus tcp enabled so you could use the modbus adapter to query values. KOSTAL has not allowed writing any of the modbus addresses, though. So you can't set e. g. battery minimum SoC using ioBroker.

### Using the adapter

The adapter should fill some objects under the plenticore.X object tree. Some of those are read-only, e. g. the current PV output or home power consumption. Others are changeable, e. g. the battery's minimum SoC or the battery management modes. I tested the adapter on the Plenticore Plus 10.

I have not yet implemented all the API endpoints, especially the energy flow statistics that are used for the "statistics" page on the web interface. Also the adapter is highly lacking translations as I am completely new to developing for ioBroker.

## Objects

Following is an excerpt of the most important objects used and filled by this adapter. All Settings marked with `[**]` should be editable but not all of them were tested and there may be (and will be) bugs.

### plenticore.X.devices.local

The devices.local tree contains information about the inverter and possibly attached smart energy meter and / or battery.

`plenticore.X.devices.local.Dc_P` - the current DC power including the self-used power of the inverter. This value should be near the value of `plenticore.X.devices.local.ac.P` (about +30-40W)  
`plenticore.X.devices.local.Home_P` - the current total home power used  
`plenticore.X.devices.local.HomeBat_P` - the current home power provided by the battery  
`plenticore.X.devices.local.HomePv_P` - the current home power directly provided by the plant  
`plenticore.X.devices.local.HomeGrid_P` - the current home power provided by the grid  
`plenticore.X.devices.local.LimitEvuAbs` - the calculated current limit of power sent to the grid. if more power is generated by the plant it will be lost.  
`plenticore.X.devices.local.StateKey0` - if true the battery management of the inverter has been unlocked


#### plenticore.X.devices.local.ac

This channel contains information about the AC side of the inverter. Most important are:  
`plenticore.X.devices.local.ac.Frequency` - the net frequency  
`plenticore.X.devices.local.ac.L1_P` - the current power of phase 1 in W  
`plenticore.X.devices.local.ac.L2_P` - the current power of phase 2 in W  
`plenticore.X.devices.local.ac.L3_P` - the current power of phase 3 in W  
`plenticore.X.devices.local.ac.P` - the current total power emitted by the inverter, including battery discharge

#### plenticore.X.devices.local.battery

`plenticore.X.devices.local.battery.Cycles` - the lifetime battery cycles up to now  
`[**] plenticore.X.devices.local.battery.DynamicSoc` - true if dynamic SoC is enabled (only if `SmartBatteryControl` is true, too)  
`[**] plenticore.X.devices.local.battery.MinHomeConsumption` - the minimum home power consumption that is needed for the battery to be used  
`[**] plenticore.X.devices.local.battery.MinSoc` - the desired minimum SoC (State of Charge) of the battery. The actual SoC might go below this if there is lacking sun power.  
`plenticore.X.devices.local.battery.P` - the current battery power (negative if charging, positive if discharging)  
`[**] plenticore.X.devices.local.battery.SmartBatteryControl` - true if the smart battery management is enabled. Regarding the official manual this shall only be enabled if there is no further AC source like a second inverter involved  
`plenticore.X.devices.local.battery.SoC` - the current state of charge of the battery  

#### plenticore.X.devices.local.inverter

`plenticore.X.devices.local.inverter.MaxApparentPower` - the maximum power that the inverter can provide

#### plenticore.X.devices.local.pv1 / pv2

`plenticore.X.devices.local.pvX.P` - the current power that is provided by phase X of the plant

### plenticore.X.scb

This channel contains information and settings of the device itself

#### plenticore.X.scb.modbus

`[**] plenticore.X.scb.modbus.ModbusEnable` - true if the modbus tcp is enabled  
`[**] plenticore.X.scb.modbus.ModbusUnitId` - modbus unit id of the device

#### plenticore.X.scb.network

`[**] plenticore.X.scb.network.Hostname` - the current host name of the inverter  
`[**] plenticore.X.scb.network.IPv4Auto` - use DHCP to provide the ip address settings for the inverter  
`[**] plenticore.X.scb.network.IPv4Address` - the current ip address of the inverter  
`[**] plenticore.X.scb.network.IPv4DNS1` and `plenticore.X.scb.network.IPv4DNS2` - the currently used DNS servers  
`[**] plenticore.X.scb.network.IPv4Gateway` - the currently used network gateway  
`[**] plenticore.X.scb.network.IPv4Subnetmask` - the network subnet mask  

#### plenticore.X.scb.time

`[**] plenticore.X.scb.time.NTPservers` - the currently used time servers (NTP). Those can be multiple ones separated by space.  
`[**] plenticore.X.scb.time.NTPuse` - use NTP to set current device time settings  
`[**] plenticore.X.scb.time.Timezone` - the time zone of the device

### plenticore.X.scb.statistic.EnergyFlow

The datapoints in this section contain the statistics that are visible in the Plenticore web UI. Following only the `Day` datapoints are mentioned, but each of them is also available for `Month`, `Year` and `Total`.

`plenticore.0.scb.statistic.EnergyFlow.AutarkyDay` - the autarky in percent for the current day  
`plenticore.0.scb.statistic.EnergyFlow.CO2SavingDay` - the estimated saved CO2 in kg for the current day  
`plenticore.0.scb.statistic.EnergyFlow.EnergyHomeDay` - the total home consumption in Wh for the current day  
`plenticore.0.scb.statistic.EnergyFlow.EnergyHomePvDay` - the total home consumption provided by the PV plant for the current day  
`plenticore.0.scb.statistic.EnergyFlow.EnergyHomeBatDay` - the total home consumption provided by the battery for the current day  
`plenticore.0.scb.statistic.EnergyFlow.EnergyHomeGridDay` - the total home consumption provided by the power grid for the current day  
`plenticore.0.scb.statistic.EnergyFlow.OwnConsumptionRateDay` - the own consumption rate (generated plant power NOT sent to the grid) for the current day  
`plenticore.0.scb.statistic.EnergyFlow.YieldDay` - the total yield of the plant for the current day

## Forecast data

To be able to use the forecast feature you need to have the ioBroker.darksky or the ioBroker.weatherunderground adapter installed. In addition you need to have the system's global geo position (longitude and latitude) configured and set the extended config of the plenticore adapter (panel and battery data if applicable).

### plenticore.0.forecast.consumption

`plenticore.0.forecast.consumption.day` - current power consumption average for daytime during last 3 days  
`plenticore.0.forecast.consumption.night` - current power consumption average for nighttime during last 3 days  
`plenticore.0.forecast.consumption.remaining` - estimated remaining power consumption for current forecast day until sunset

### plenticore.0.forecast.current

`plenticore.0.forecast.current.sky` - current cloud coverage from weather adapter  
`plenticore.0.forecast.current.visibility` - current visibility from weather adapter  
`plenticore.0.forecast.current.power.generated` - generated plant power on current day until current time  
`plenticore.0.forecast.current.power.max` - calculated maximum plant power on clear sky (0% cloud coverage)  
`plenticore.0.forecast.current.power.sky` - calculated plant power taking into account current cloud coverage from weather adapter  
`plenticore.0.forecast.current.power.skyvis` - calculated plant power taking into account current cloud coverage and visibility from weather adapter  
`plenticore.0.forecast.current.sun.azimuth` - current sun position (azimuth)  
`plenticore.0.forecast.current.sun.elevation` - current sun position (eleevation)  
`plenticore.0.forecast.current.sun.sunrise` - sunrise time of forecast date (either today or tomorrow)  
`plenticore.0.forecast.current.sun.sunset` - sunset time of forecast date (either today or tomorrow)  

### plenticore.0.forecast.power

`plenticore.0.forecast.power.date` - date the current power forecast info is for  
`plenticore.0.forecast.power.day` - total power forecast for the day  
`plenticore.0.forecast.power.day_high` - total power forecast for the day ignoring the weather adapter's visibility data  
`plenticore.0.forecast.power.remaining` - remaining power of forecast total for the day, based on `plenticore.0.forecast.power.day`  
`plenticore.0.forecast.power.Xh.power` - estimated total power from plant on sun hour X of the forecast day, where 1h is the hour of sunrise  
`plenticore.0.forecast.power.1h.time` - the time the sun hour for `plenticore.0.forecast.power.Xh.power` starts

## Changelog

### 2.0.0

-   Code rework
-   Outsourced many functions to libraries
-   This version has new dependencies and requires a newer adapter-core version!
-   Several fixes

### 1.1.1

-   No changes

### 1.1.0

-   Added support for weatherunderground weather adapter. The adapter can be choosen as alternative forecast source over the DarkSky adapter.

### 1.0.2

-   Fixed a warning message occuring far too often

### 1.0.1

-   Added forecast features to readme

### 1.0.0

-	Added power forecast feature

### 0.1.5

-   Added translations
-   Fixed shadow management handling.

### 0.1.4

-   Added shadow management datapoint.

### 0.1.3

-   Do not query battery values if battery management is not unlocked.

### 0.1.2

-   Resolved adapter check issues, see https://github.com/StrathCole/ioBroker.plenticore/issues/1
-   Added statistics data points.

### 0.1.1

-   Removed admin adapter dependency

### 0.1.0

-   First running Version

## License

The MIT License (MIT)

Copyright (c) 2020 Marius Burkard

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.


## Donate
[![paypal](https://www.paypalobjects.com/en_US/i/btn/btn_donateCC_LG.gif)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=SFLJ8HCW9T698&source=url)
