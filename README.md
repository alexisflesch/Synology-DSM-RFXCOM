# RFXCOM and Domoticz on Synology with DSM 7.2

## Set up DSM

* Enable SSH via Control Panel > Terminal & SNMP > Terminal > Enable SSH service
* Enable Docker via Apps > Container

## Enable kernel modules for RFXCOM

RFXCOM requires specific modules to be properly recognized by the system. After connecting through ssh, ensure that RFXCOM is recognized by the system:
```bash
$ lsusb
|__usb1          1d6b:0002:0404 09  2.00  480MBit/s 0mA 1IF  (Linux 4.4.302+ xhci-hcd xHCI Host Controller 0000:00:15.0) hub
  |__1-1         0403:6001:0600 00  2.00   12MBit/s 90mA 1IF  (RFXCOM RFXtrx433 A13XQMM)
  |__1-4         f400:f400:0100 00  2.00  480MBit/s 200mA 1IF  (Synology DiskStation 65003A03E4FFE872)
|__usb2          1d6b:0003:0404 09  3.00 5000MBit/s 0mA 1IF  (Linux 4.4.302+ xhci-hcd xHCI Host Controller 0000:00:15.0) hub
```
Load the required kernel modules using the following commands:
```bash
/sbin/modprobe usbserial
/sbin/modprobe ftdi_sio
/sbin/modprobe cdc-acm
```

## Install domoticz using docker
Connect to the Synology via SSH and run the following command to install domoticz using docker.

```bash
sudo docker run -d --name=domoticz \
-p 8084:80 \
-e TZ=Europe/Paris \
-e WWW_PORT=80 \
-v /volume1/docker/domoticz:/opt/domoticz/userdata \
--device=/dev/ttyUSB0:/dev/ttyUSB0 \   # Map the RFXCOM device from host to container
--privileged \
--restart always \
domoticz/domoticz
```

## Make the module loading permanent

Add the following lines to `/etc/rc.local` to enable the kernel modules for RFXCOM at boot time.
```bash
/sbin/modprobe usbserial
/sbin/modprobe ftdi_sio
/sbin/modprobe cdc-acm
```
Reboot the Synology to apply the changes and check that everything is working as expected.


## Enjoy !

Domoticz should now be available at `http://<synology-ip>:8084`. You can now add devices and start automating your home.