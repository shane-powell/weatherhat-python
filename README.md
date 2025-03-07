# Weather HAT Python Library & Examples

[![Build Status](https://shields.io/github/workflow/status/pimoroni/weatherhat-python/Python%20Tests.svg)](https://github.com/pimoroni/weatherhat-python/actions/workflows/test.yml)
[![Coverage Status](https://coveralls.io/repos/github/pimoroni/weatherhat-python/badge.svg?branch=master)](https://coveralls.io/github/pimoroni/weatherhat-python?branch=master)
[![PyPi Package](https://img.shields.io/pypi/v/weatherhat.svg)](https://pypi.python.org/pypi/weatherhat)
[![Python Versions](https://img.shields.io/pypi/pyversions/weatherhat.svg)](https://pypi.python.org/pypi/weatherhat)

# Pre-requisites

You must enable:

* i2c: `sudo raspi-config nonint do_i2c 0`
* spi: `sudo raspi-config nonint do_spi 0`

You can optionally run `sudo raspi-config` or the graphical Raspberry Pi Configuration UI to enable interfaces.

# Installing

Stable library from PyPi:

* Just run `pip3 install weatherhat`

In some cases you may need to use `sudo` or install pip with: `sudo apt install python3-pip`

Latest/development library from GitHub:

* `git clone https://github.com/pimoroni/weatherhat-python`
* `cd weatherhat-python`
* `sudo ./install.sh`

Some of the examples use additional libraries. You can install them with:

```bash
pip3 install fonts font-manrope pyyaml adafruit-io numpy
```

You may also need to install `libatlas-base-dev`

```
sudo apt-get install libatlas-base-dev
```

# Using The Library

Import the `weatherhat` module and create an instance of the `WeatherHAT` class.

```python
import weatherhat

sensor = weatherhat.WeatherHAT()
```

Weather HAT updates the sensors when you call `update(interval=5)`.

Temperature, pressure, humidity, light and wind direction are updated continuously.

Rain and Wind measurements are measured over an `interval` period. Weather HAT will count ticks of the rain gauge and (half) rotations of the anemometer, calculate rain/wind every `interval` seconds and reset the counts for the next pass.

For example the following code will update rain/wind speed every 5 seconds, and all other readings will be updated on demand:

```python
import time
import weatherhat

sensor = weatherhat.WeatherHAT()

while True:
    sensor.update(interval=5.0)
    time.sleep(1.0)
```

# Averaging Readings

The Weather HAT library supplies set of "history" classes intended to save readings over a period of time and provide access to things like minimum, maximum and average values with unit conversions.

For example `WindSpeedHistory` allows you to store wind readings and retrieve them in mp/h or km/h, in addition to determining the "gust" (maximum wind speed) in a given period of time:

```python
import time
import weatherhat
from weatherhat.history import WindSpeedHistory

sensor = weatherhat.WeatherHAT()
wind_speed_history = WindSpeedHistory()

while True:
    sensor.update(interval=5.0)
    if sensor.updated_wind_rain:
        wind_speed_history.append(sensor.wind_speed)
        print(f"Average wind speed: {wind_speed_history.average_mph()}mph")
        print(f"Wind gust: {wind_speed_history.gust_mph()}mph")
    time.sleep(1.0)
```

# Quick Reference

## Temperature

Temperature readings are given as degrees celsius and are measured from the Weather HAT's onboard BME280.

### Device Temperature

```python
sensor.device_temperature
```

Device temperature in degrees celsius.

This is the temperature read directly from the BME280 onboard Weather HAT. It's not compensated and tends to read slightly higher than ambient due to heat from the Pi.

### Compensated (Air) Temperature

```python
sensor.temperature
```

Temperature in degrees celsius.

This is the temperature once an offset has been applied. This offset is fixed, and taken from `sensor.temperature_offset`.

## Pressure

```python
sensor.pressure
```

Pressure in hectopascals.

## Humidity

```python
sensor.humidity
```

Humidity in %.

### Relative Humidity

```python
sensor.relative_humidity
```

Relative humidity in %.

Relative humidity is the water content of the air compensated for temperature, since warmer air can hold more water.

It's expressed as a percentage from 0 (no moisture) to 100 (fully saturated).

### Dew Point

```python
sensor.dewpoint
```

Dew point in degrees celsius.

Dew point is the temperature at which water - at the current humidity - will condense out of the air.

## Light / Lux

```python
sensor.lux
```

Light is given in lux.

Lux ranges from 0 (complete darkness) to.

## Wind

Both wind and rain are updated on an interval, rather than on-demand.

To see if an `update()` call has resulted in new wind/rain measurements, check:

```python
sensor.updated_wind_rain
```

### Wind Direction

```python
sensor.wind_direction
```

Wind direction in degrees.

Wind direction is measured using a potentiometer and uses an analog reading internally. This is converted to degrees for convenience, and will snap to the nearest 45-degree increment with 0 degrees indicating North.

### Wind Speed

```python
sensor.wind_speed
```

Wind speed in meters per second.

Weather HAT counts every half rotation and converts this to cm/s using the anemometer circumference and factor.

It's updated depending on the update interval requested.

## Rain

```python
sensor.rain
```

Rain amount in millimeters per second.

Weather HAT counts every "tick" of the rain gauge (roughly .28mm) over the given update internal and converts this into mm/sec.

### Total Rain

```python
sensor.rain_total
```

Total rain amount in millimeters for the current update period.
