# AP-STA mode

## Overview

This walkthrough describes an installation of RaspAP on the [Raspberry Pi Zero W](https://www.raspberrypi.org/products/raspberry-pi-zero-w/). A managed mode AP, variously known as **WiFi client AP mode**, a **micro-AP** or simply **AP-STA**, works "out-of-the-box" with the Quick Installer if the steps below are followed carefully. This feature was added to RaspAP specifically to support Internet of Things (IoT) and embedded applications for the Pi Zero W, however it is equally useful for a broad range of projects.

Continue reading for an explanation of AP-STA mode, or [skip to the installation](#installation).

![](https://i.imgur.com/gppLmAj.png)

## What is AP-STA mode?
Many wireless devices support _simultaneous_ operation as both an access point (AP) and as a wireless client/station (STA). This is sometimes called Wi-Fi AP/STA concurrency. In this configuration, it is possible to create a software AP acting as a "wireless repeater" for an existing network, using a single wireless device. This capability is listed in the following section in the output of `iw list`:

```
$ iw list | grep -A 4 'valid interface'
    valid interface combinations:
	* #{ managed } <= 1, #{ P2P-device } <= 1, #{ P2P-client, P2P-GO } <= 1,
	  total <= 3, #channels <= 2
	* #{ managed } <= 1, #{ AP } <= 1, #{ P2P-client } <= 1, #{ P2P-device } <= 1,
	  total <= 4, #channels <= 1
```

The second valid interface combination indicates that both a `managed` and `AP` configuration is possible. The constraint `#channels <= 1` means that your software AP must operate on the same channel as your Wi-Fi client connection. 

> :information_source: **Note:** if you have a second wireless adapter bound to `wlan1` on a Pi Zero W (or other device), refer to [this FAQ](/faq/#interfaces). 

## Use cases
There are many scenarios in which AP-STA mode might be useful. These are some of the more popular ones:

1. A device that connects to a wireless AP but needs an admin interface to configure the network and/or other services.
2. A hub for Internet of Things devices, while also creating a bridge between them and the internet.
3. A guest interface to your home wireless network. 

Security is an important consideration with IoT and it can be beneficial to keep your devices on a separate network, for safety’s sake. No one wants a random internet user turning your lights on and off.

## How does AP-STA work?
In this configuration, we create a virtual network interface (here `uap0`) and add it as the AP to the physical `wlan0` device. This virtual interface is used by several of the services needed to operate a software access point. RaspAP manages these configurations in the background for you. Relevant sections are displayed below as examples.

`dhcpcd.conf`:
```
# RaspAP uap0 configuration
interface uap0
static ip_address=192.168.50.1/24
nohook wpa_supplicant
```

`hostapd.conf`:
```
# RaspAP wireless client AP mode
interface=uap0
```

`dnsmasq.conf`:
```
# RaspAP uap0 configuration
interface=lo,uap0               # Use interfaces lo and uap0
bind-interfaces                 # Bind to the interfaces
domain-needed                   # Don't forward short names
bogus-priv                      # Never forward addresses in the non-routed address spaces
```

On AP-STA startup and system reboots, RaspAP's service control script adds the virtual `uap0` interface and brings it up, like so:

```
iw dev wlan0 interface add uap0 type __ap
ifconfig uap0 up
```

After the virtual `uap0` interface is added to the `wlan0` physical device, we can then start up `hostapd`. It is important that the virtual interface is brought up first, otherwise it will fail with the message "could not configure driver mode". We also need to be sure that the interface is not managed by `systemd-networkd`, so this service should be disabled. These steps are handled by the [RaspAP daemon](/faq/#raspap-service). 

With a basic understanding of AP-STA mode, we can proceed with the installation.

## Installation

1. Begin by flashing an SD card with the latest release of [Raspberry Pi OS (32-bit) Lite](https://www.raspberrypi.org/downloads/raspbian/). 
2. Prepare the SD card to connect to your WiFi network in headless mode [according to this FAQ](/faq/#headless).
3. Enable `ssh` access by creating an empty file called "ssh" (no extension) in the SD card's root. 
4. Insert the SD card into the Pi Zero W and connect it to power. **Note:** the standard power supply for the Raspberry Pi is 5.1V @ 2.5A. Other power sources may result in undervoltage or other issues. Do _not_ use the micro USB connection. 
5. Connect to your Pi via ssh. `ssh pi@raspberrypi.local` is typical.
6. Follow the [project prerequisites](/#quick-start) exactly. Do _not_ skip any of these steps.
7. Invoke the [Quick Installer](/quick/) as normal: `curl -sL https://install.raspap.com | bash`.
8. The installer automatically detects a Pi (or other device) without an active `eth0` interface. In this case, you will _not_ be prompted to reboot your Pi.

> ![](https://i.imgur.com/mwKYBKF.png){: style="width:350px"}

9. Open the RaspAP admin interface in your browser, usually http://raspberrypi.local.
10. The status widget should indicate that hostapd is inactive. This is expected.
11. Confirm that the **Wireless Client** dashboard widget displays an active connection.
12. Choose **Hotspot > Advanced** and enable the **WiFi client AP mode** option.
13. Optionally, enable **Logfile output** as this is often helpful for troubleshooting. 
14. Choose **Save settings** and **Start hotspot**.
15. Wait a few moments and confirm that your AP has started. 

![](https://i.imgur.com/t5p53zP.png){: style="width:450px"}

> :information_source: **Note:** The **WiFi client AP mode** option will be disabled, or "greyed out", until a wireless client is configured.

## When to reboot?
Rebooting _before_ configuring AP-STA mode is likely the main cause of problems for users with the Pi Zero W. The reason is the [default configuration](/defaults/) is designed for a wired (ethernet) AP. 

Once the Pi Zero W is configured in AP-STA mode, RaspAP will store several values in `/etc/raspap/hostapd.ini`:
```
LogEnable = 1
WifiAPEnable = 1
BridgedEnable = 0
WifiManaged = wlan0
```
These are used by RaspAP's [systemd control service](/faq/#raspap-service) `raspapd` to determine that a managed mode AP is enabled for the Pi and restore the connection after subsequent reboots.

## Changing hostapd settings
Changes to the hotspot configuration should be applied to the `wlan0` physical device, not `uap0` (a virtual interface). In other words, configure hostapd _before_ enabling AP-STA mode.

