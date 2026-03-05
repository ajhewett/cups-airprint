
Forked from https://github.com/xirixiz/cups-airprint

# Working on Synology DSM 7.x (ARM and AMD64)

This Ubuntu-based Docker image runs a CUPS instance that is meant as an AirPrint relay for printers that are already on the network but not AirPrint capable.
* `Included drivers HP, Samsung, Canon, Xerox, etc.`

## Easy run command (use username and password: admin/admin):
```docker run --name airprint --restart unless-stopped --net host xirixiz/synology_airprint:latest```

### Before run docker conteiner on DSM7 Synology run this commands in ssh terminal:
* `sudo synosystemctl stop cupsd`
* `sudo synosystemctl stop cups-lpd`
* `sudo synosystemctl stop cups-service-handler`
* `sudo synosystemctl disable cupsd`
* `sudo synosystemctl disable cups-lpd`
* `sudo synosystemctl disable cups-service-handler`

### Add and setup printer:
* CUPS will be configurable at http://[host ip]:631 using the `admin/admin`.
* Make sure you select `Share This Printer` when configuring the printer in CUPS.
* ***After configuring your printer, you need to close the web browser for at least 60 seconds. CUPS will not write the config files until it detects the connection is closed for as long as a minute.***

### After setup and testing AirPrint, you can back run on services. (maybe you will need restart nas)
* `sudo synosystemctl start cupsd`
* `sudo synosystemctl start cups-lpd`
* `sudo synosystemctl start cups-service-handler`
* `sudo synosystemctl enable cupsd`
* `sudo synosystemctl enable cups-lpd`
* `sudo synosystemctl enable cups-service-handler`

## Manual Configuration

### Volumes:
* `/config`: where the persistent printer configs will be stored
* `/services`: where the Avahi service files will be generated

### Variables:
* `CUPSADMIN`: the CUPS admin user you want created - default is `admin` if unspecified
* `CUPSPASSWORD`: the password for the CUPS admin user - default is `admin` username if unspecified
* `CUPSMODE`: can be "dnssd" or "cups"
* `DNS`: the ip address of a DNS server
* `UID`: user id e.g. $(id -u)
* `GID`: group ip e.g. $(id -g)


### Ports/Network:
* **Must be run on host network. This is required to support multicasting which is needed for Airprint.**

### Example docker compose run command:
```
docker compose up -d
```

### Example docker run env command:
```
docker run --name cups --restart unless-stopped  --net host\
  -v <your services dir>:/services \
  -v <your config dir>:/config \
  -e CUPSADMIN="<username>" \
  -e CUPSPASSWORD="<password>" \
  local/cups-airprint:latest
```

### Example adding a printer:
```
# Install "IPP Everywhere" printer samsung-m283x which has broken AirPrint support and no firmware updates
sudo lpadmin -p samsung-m283x -E \
	-v "ipp://samsung-m283x/ipp/print" \
	-m everywhere \
	-D "Samsung M283x Series" \
	-L "Walli's Room" \
	-o sides-default=two-sided-long-edge \
	-o media-default=iso_a4_210x297mm \
	-o cupsIPPSupplies=true \
	-o printer-is-shared=true
```
