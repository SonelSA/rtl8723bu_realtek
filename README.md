# Driver for rtl8723bu.

## Device

The device is an USB dongle for WiFi and Bluetooth connectivity.

## Realtek driver

Realtek provides a driver, but it cannot be downloaded from their [site][Realtek].

The newest driver we found on the net comes from [lm811][lm811].
It can be extracted from LM817_WIFI_BT_LINUX_v4.4.5_RTL8723BU.zip
This is the version this repository uses as starting point.
Unfortunately it does not compile on newer kernels (4.15 and later).

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
echo "options 8723bu rtw_power_mgnt=0 rtw_ips_mode=0 rtw_smart_ps=0" > ${D}/etc/modprobe.d/8723bu.conf
```

#### Device not operable after disconnection.

A solution is to reaload the kernel module:
```
/sbin/modprobe -r 8723bu && /sbin/modprobe 8723bu
```
Unfortunately the time after which next connection will be possible seems to be
random.

## This driver.

This driver is based on the latest available [Realtek driver][lm811].
The compilation patches for newer kernels are stolen from [lwfinger][lwfinger].

### Tests

This driver was tested on x86_64 (debian buster 4.18.20-2).
Both WiFi and Bluetooth work, but workaround is needed for NetworkManager.

### Workarounds.

#### Cannot connect to WiFi with NetworkManager.

When a WiFi connection is started with NetworkManager, connection fails and
NetworkManager asks for password again, as if the password was wrong. Workaround
is taken from [this forum thread][nm_workaround_thread].
```
echo "[device-8723bu]" > ${D}/etc/NetworkManager/conf.d/8723bu.conf
echo "match-device=driver:rtl8723bu" >> ${D}/etc/NetworkManager/conf.d/8723bu.conf
echo "wifi.scan-rand-mac-address=no" >> ${D}/etc/NetworkManager/conf.d/8723bu.conf
```

#### Prevent rtl8xxxu from being loaded
rtl8xxxu could try to take over control over our device.
To prevent this use following command.
```
echo "blacklist rtl8xxxu" >> /etc/modprobe.d/8723bu.conf
```

## Links

[Device page at Realtek][Realtek]

[LM Technologies][lm811]

[lwfinger fork on github][lwfinger]

[NetworkManager workaround][nm_workaround_thread]


[Realtek]: https://www.realtek.com/en/products/communications-network-ics/item/rtl8723bu
[lm811]: https://www.lm-technologies.com/product/wifi-and-bluetooth-usb-module-4-0-dual-mode-class-1-lm811/
[lwfinger]: https://github.com/lwfinger/rtl8723bu
[nm_workaround_thread]: https://github.com/diederikdehaas/rtl8812AU/issues/71
