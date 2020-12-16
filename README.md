# Driver for rtl8723bu.

## Device

The device is an USB dongle for WiFi and Bluetooth connectivity.

## Realtek driver

Realtek provides a driver, but it cannot be downloaded from their [site][Realtek].

We found a driver here [lm811][lm811].
It can be extracted from LM817_WIFI_BT_LINUX_v4.4.5_RTL8723BU.zip
This is the version this repository uses as starting point.
Unfortunately it does not compile on newer kernels (4.15 and later).

A newer driver v5.2.17.1 can be downloaded from here
[thisiswangle][thisiswangle].

Even newer driver v5.8.4 can be downloaded from here [edimax][edimax].
It supports kernels up to 5.2.

## lwfinger driver

There is a github repository maintained by [independent developers][lwfinger].
The code there is based on some older Realtek driver.
Changes to the original driver include:
- compilation with newer kernel versions,
- editorial changes,
- dkms support,
- loading firmware from binary file,
- readme

### Tests.

We have tested the driver on both x86_64 (debian buster) and arm (yocto sumo).
The results are same.

What works:
- scanning for WiFi networks,
- connecting to WiFi networks,
- Bluetooth (needs a workaroud)

What does not work:
- after disconnecting from WiFi network, the device is no longer able to connect
or scan for new networks (there is a workaround, but not reliable)

### Workarounds.

#### Bluetooth and power management.

It seems that power management for WiFi is affecting Bluetooth.
The power saving can be disabled with:
```
echo "options 8723bu rtw_power_mgnt=0 rtw_ips_mode=0 rtw_smart_ps=0" > /etc/modprobe.d/8723bu.conf
```

#### Device not operable after disconnection.

A solution is to reaload the kernel module:
```
/sbin/modprobe -r 8723bu && /sbin/modprobe 8723bu
```
Unfortunately the time after which next connection will be possible seems to be
random.

## This driver.

This driver was based on the latest available [Realtek driver][lm811].
Later it was updated to version v5.2.17.1 from here [thisiswangle][thisiswangle].
The driver was updated again with version v5.8.4 from [edimax][edimax].

The compilation patches for newer kernels are stolen from [lwfinger][lwfinger] and
[joy4eg][joy4eg].

### Tests

#### State at 2019-11-05, v5.2.17.1.
It does compile and was tested on x86_64 Debian (kernel 5.2)
and raspberry Pi CM3 (kernel 4.14).

Basic tests confirmed both WiFi and Bluetooth to work.
Workarounds needed for lwfinger driver are not needed.
Workarounds described below are needed instead.

#### State at 2020-12-16, v5.8.4.
It does compile and was tested on x86_64 Debian (kernel 5.9.0)
and raspberry Pi CM3 (kernel 4.14).

Basic tests confirmed both WiFi and Bluetooth to work.
The test were less extensive than v5.2.17.1, but more is to follow.
The need for workarounds has net yet been confirmed.

### Manual install

Run the following commands in the Linux terminal.

```
make
sudo make install
sudo modprobe -v 8723bu
```

### Workarounds.

#### Cannot connect to WiFi with NetworkManager.

When a WiFi connection is started with NetworkManager, connection fails and
NetworkManager asks for password again, as if the password was wrong. Workaround
is taken from [this forum thread][nm_workaround_thread].
```
echo "[device-8723bu]" > /etc/NetworkManager/conf.d/8723bu.conf
echo "match-device=driver:rtl8723bu" >> /etc/NetworkManager/conf.d/8723bu.conf
echo "wifi.scan-rand-mac-address=no" >> /etc/NetworkManager/conf.d/8723bu.conf
```

#### Prevent rtl8xxxu from being loaded

rtl8xxxu could try to take over control over our device.
To prevent this use following command.
```
echo "blacklist rtl8xxxu" >> /etc/modprobe.d/8723bu.conf
```

#### Bluetooth and power management.

Power management for WiFi is still affecting Bluetooth.
It was tested that Bluetooth will allow for scanning and pairing,
but will not allow to make a RFCOMM connection from external device,
if WiFi is not enabled.
Situation can be sligthly improved with options below.
WiFi will still have to be enabled at least for a moment after module is loaded,
but WiFi can be disabled later and Bluetooth connection will still be possible.
```
echo "options 8723bu rtw_ips_mode=0" > /etc/modprobe.d/8723bu.conf
```

#### Noisy dmesg

By default the driver prints lots of debug messages to the kernel log,
which can be annoying and can grow log files uncontrollably.
To reduce the load use following commands:
```
echo "options 8723bu rtw_drv_log_level=3" >> /etc/modprobe.d/8723bu.conf
```
Default log level is 4.
The lower level is chosen the less messages will be printed.

## Other drivers

More drivers can be found on the net.

### ulli-kroll

Based on v4.4.5. WiFi only, not evaluated. [ulli-kroll]

### johnheenan

Patch to generic kernel driver rtl8xxxu. Not evaluated. [johnheenan]

## Links

[Device page at Realtek][Realtek]

[v4.4.5 at LM Technologies page][lm811]

[v5.2.17.1 from thisiswangle on github][thisiswangle]

[v5.8.4 at Edimax page][edimax]

[lwfinger fork on github][lwfinger]

[joy4eg fork with patches for kernel 5.8][joy4eg]

[NetworkManager workaround][nm_workaround_thread]

[ulli-kroll fork on github][ulli-kroll]

[johnheenan patch on github][johnheenan]

[Realtek]: https://www.realtek.com/en/products/communications-network-ics/item/rtl8723bu
[lm811]: https://www.lm-technologies.com/product/wifi-and-bluetooth-usb-module-4-0-dual-mode-class-1-lm811/
[thisiswangle]: https://github.com/thisiswangle/WGD-rtl8723bu
[edimax]: https://www.edimax.com/edimax/download/download/data/edimax/global/download/for_home/wireless_adapters/wireless_adapters_n150/ew-7611ulb
[lwfinger]: https://github.com/lwfinger/rtl8723bu
[joy4eg]: https://github.com/SonarWireless/rtl8723bu_realtek
[nm_workaround_thread]: https://github.com/diederikdehaas/rtl8812AU/issues/71
[ulli-kroll]: https://github.com/ulli-kroll/rtl8723bu
[johnheenan]: https://github.com/johnheenan/rtl8xxxu
