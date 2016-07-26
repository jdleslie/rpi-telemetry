# Overview
Goal is to use a Raspberry Pi to gather sensor readings from a variety of 1-Wire sensors (mainly 
[DS18B20+](http://datasheets.maximintegrated.com/en/ds/DS18S20.pdf) assembled [inside RJ45 
jacks](http://www.flickr.com/photos/aaronpk/13953336384)) embedded around
the house. Previously I used [ControlByWeb](http://controlbyweb.com) devices which support either 4 or 8 sensors per $150-200
control module. This Rpi based approach scales up to a much larger number of sensors with less hardware and a more convenient
1U rackmount package.

Included in this repo:
* Design for 1U rackmount bracket to mount Raspberry Pi, [PiWire+ HAT](http://www.axiris.eu/en/index.php/1-wire/abiowire), and four [1-Wire Breakout Boards](http://www.axiris.eu/en/index.php/1-wire/1-wire-breakout-board).
  * These are simple laser cut designs that can be assembled in a single step with wood glue.
  * Any 3mm thick material of the requisite thickness is fine, mine are 0.112-in/3mm masonite "hardboard" ordered from [Ponoko US](http://ponoko.com).
  * Sadly the only Ponoko material size that is wide enough is "P3", which is so large that two brackets can be ganged up in a single order
* Notes (below) on configuration for Raspberry Pi

Thermal sensors are DS18B20 ICs inside RJ45 plugs:

![rj45 thermal sensor](https://photos.smugmug.com/Projects/House-Projects/1-Wire-Sensor-Hub/i-nn9RN6m/0/L/2016-07-26%2008.59.05-L.jpg)

inspired by [aaronpk](http://c1.staticflickr.com/6/5084/13953336384_6527c25317_k.jpg)

Assembled, senssor hub fits in 1U:

![rpi sensor hub in 1U bracket](https://photos.smugmug.com/Projects/House-Projects/1-Wire-Sensor-Hub/i-7vHKcCM/1/L/2016-07-25%2018.40.12-L.jpg)

# Software configuration
The actual code that gathers data from this system this runs in a Docker instance on a different host, connecting to `owserver` over the network. Configuration on the Pi is minimal.

## Configure Raspbian

* Log in via ssh, user is `pi` and password is `raspberry` on default Jessie-Lite image
* `sudo raspi-config`
* Set boot option to expand filesystem at next startup
* Advanced -> I2C = Yes and Yes
* Make i2c device nodes [readable by all users](http://blog.chrysocome.net/2012/11/raspberry-pi-i2c.html)
```
echo << EOF > /etc/udev/rules.d/99-i2c.rules
SUBSYSTEM=="i2c-dev", MODE="0666"
EOF
```
* Reload udev rules `udevadm trigger`

## Install Packages

* `sudo apt-get update`
* `sudo apt-get install owhttpd owserver ow-shell owfs i2c-tools`
* `sudo systemctl enable owserver owhttpd`
* `sudo systemctl start owserver owhttpd`

## Configure OWFS
* Edit `/etc/owfs.conf`
```
# 1-Wire interfaces for PiWire+/AbioWire HAT (http://www.axiris.eu/en/index.php/1-wire/abiowire)
server: i2c = /dev/i2c-1:0
server: i2c = /dev/i2c-1:1
server: i2c = /dev/i2c-1:2

# Allow owserver access from all IP addresses
#server: port = localhost:4304
server: port = 4304

# Allow owhttpd access from all IP addresses
http: port = 2121
http: port = 80

# 2 minute caching timeout for "stable" (e.g. temperature) values
timeout_stable = 120

# Human-readable aliases, per http://owfs.org/index.php?page=aliases
#alias = /etc/owfs-alias.conf # don't bother if aliases will be maintained on client side
```
* Restart daemons: `sudo systemctl restart owserver owhttpd`
