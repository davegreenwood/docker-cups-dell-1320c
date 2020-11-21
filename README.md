# About

Specically to run an old Dell 1320c printer.

Modified copy of source code at:

https://github.com/quadportnick/docker-cups-airprint

and:

https://github.com/RagingTiger/docker-cups-airprint

This i386/debian:buster-slim Docker image runs a CUPS instance that is meant
as an AirPrint relay for printers that are already on the network but not AirPrint capable.
The local Avahi will be utilized for advertising the printers on the network.

This allows a printer to be used with iphones, etc.

There was never a driver for the Dell 1320c on linux, but the FujiXerox C525 32bit driver works fine.


# Requirements

The host must have avahi-daemon installed and running.


## Build

   docker build -t dgnwd/cups-airprint:latest .


## Create
Creating a container is often more desirable than directly running it:
```
$ docker create \
       --name=cups \
       --restart=always \
       --net=host \
       -v /var/run/dbus:/var/run/dbus \
       -v /opt/appdata/cups/config:/config \
       -v /opt/appdata/cups/services:/services \
       --device /dev/bus \
       --device /dev/usb \
       -e CUPSADMIN="admin" \
       -e CUPSPASSWORD="password" \
       dgnwd/cups-airprint:latest
```
Follow this with `docker start` and your cups/airprint printer is running:
```
$ docker start cups
```
To stop the container simply run:
```
$ docker stop cups
```
To remove the container simply run:
```
$ docker rm cups
```

### Parameters
* `--name`: gives the container a name making it easier to work with/on (e.g.
  `cups`)
* `--restart`: restart policy for how to handle restarts (e.g. `always` restart)
* `--net`: network to join (e.g. the `host` network)
* `-v /opt/appdata/cups/config:/config`: where the persistent printer configs
   will be stored
* `-v /opt/appdata/cups/services:/services`: where the Avahi service files will
   be generated
* `-e CUPSADMIN`: the CUPS admin user you want created
* `-e CUPSPASSWORD`: the password for the CUPS admin user
* `--device /dev/bus`: device mounted for interacting with USB printers
* `--device /dev/usb`: device mounted for interacting with USB printers

## Using
CUPS will be configurable at <http://host-server-ip:631> using the
CUPSADMIN/CUPSPASSWORD when you do something administrative.

If the `/services` volume isn't mapping to `/etc/avahi/services` then you will
have to manually copy the .service files to that path at the command line.

## Notes
* CUPS doesn't write out `printers.conf` immediately when making changes even
though they're live in CUPS. Therefore it will take a few moments before the
services files update
* Don't stop the container immediately if you intend to have a persistent
configuration for this reason.

## Example adding the Dell Printer - attached to host usb

Visit the cups interface and:

* Click on "System - Administration - Printing"
* Click "Add Printer"
* The printer attached to usb is shown as "Dell Color Laser 1320c (Dell Color Laser 1320c)"
* Select the printer and click continue.
* Name it something useful
* Select "Share this printer", click continue...
* Select "Another make/manufacturer" and choose FX, click continue...
* Select "FX Docuprint C525 A-AP v1.0 (en)", click Add printer...
* Click "Options Installed" and for "Optional Tray Module" select "250 Sheet Feeder".
* Click "Basic" and change "Paper Source" to "Tray 1 (250 Sheets)"
* Click "Set Default Options"
* Test by selecting Maintenance > Print Test Page.

If all is well, the printer should be visible on the network to devices.
