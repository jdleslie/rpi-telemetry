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
`sudo apt-get update`
`sudo apt-get install owhttpd owserver ow-shell i2c-tools`
`sudo systemctl enable owserver owhttpd`
`sudo systemctl start owserver owhttpd`

## Modify /etc/owfs.conf
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
Restart daemons: `sudo systemctl restart owserver owhttpd`
