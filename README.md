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
[thisiswangle][thisiswangle] or here [edimax][edimax].

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

This driver was based on the latest available [Realtek driver][lm811].
Later it was updated to version v5.2.17.1 from here [thisiswangle][thisiswangle].

The compilation patches for newer kernels are stolen from [lwfinger][lwfinger].

### Tests

It does compile with kernels up to 5.2 and was tested on x86_64 Debian
Testing on 2019-11-05, kernel 5.2.17-1.

Basic tests confirmed both WiFi and Bluetooth to work.
Workarounds needed for lwfinger driver are not needed.
Workarounds described below are needed instead.

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

## Links

[Device page at Realtek][Realtek]

[v4.4.5 at LM Technologies page][lm811]

[v5.2.17.1 from thisiswangle on github][thisiswangle]

[v5.2.17.1 at Edimax page][edimax]

[lwfinger fork on github][lwfinger]

[NetworkManager workaround][nm_workaround_thread]


[Realtek]: https://www.realtek.com/en/products/communications-network-ics/item/rtl8723bu
[lm811]: https://www.lm-technologies.com/product/wifi-and-bluetooth-usb-module-4-0-dual-mode-class-1-lm811/
[thisiswangle]: https://github.com/thisiswangle/WGD-rtl8723bu
[edimax]: https://www.edimax.com/edimax/download/download/data/edimax/global/download/for_home/wireless_adapters/wireless_adapters_n150/ew-7611ulb
[lwfinger]: https://github.com/lwfinger/rtl8723bu
[nm_workaround_thread]: https://github.com/diederikdehaas/rtl8812AU/issues/71
