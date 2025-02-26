# FAQ

This guide was written to address some frequently asked questions among users of RaspAP.
FAQ items are organized into thematic sections, below, for easier reference.

If you would like to see a new FAQ that you feel would assist other users, [start a discussion](https://github.com/RaspAP/raspap-webgui/discussions) or [open an issue](https://github.com/RaspAP/raspap-webgui-docs/issues).

## General
* [What do all these settings in the UI do? Changing them seems to have no effect.](#settings)
* [How do I prepare the SD card to connect to WiFi in headless mode?](#headless)
* [Can I use wlan0 and wlan1 rather than eth0 for my AP?](#interfaces)
* [Can I use RaspAP as a monitor only, without changing my configuration?](#monitor)
* [Can I use RaspAP with my custom dnsmasq configuration?](#dnsmasq)

## Troubleshooting
* [Clients cannot obtain an IP address from the hotspot.](#noip)
* [My wifi network disappeared and I can't access the webgui. Help!](#access)
* [My custom `hostapd.conf` / `php.ini` is gone. Help!](#custom)
* [I changed the admin password and forgot what it was. Help!](#password)
* [RaspAP control panel works but there is no WiFi after reboot.](#nowifi)
* [Bridged AP mode is unstable or clients can't connect. Help!](#bridged)
* [Managed mode AP doesn't work on the Pi Zero W. Help!](#pizero-w)
* [WiFi scanning doesn't work or I get the error `cannot execute "wpa_cli reconfigure"`. Help!](#scanning)

## Integrations
* [How do I integrate RaspAP with Pi-hole?](#pihole)
* [Can I integrate RaspAP with Adguard Home?](#adguard)
* [Can I configure RaspAP to work with a captive portal?](#captive)
* [How do I create an AP activation schedule?](#schedule)
* [Can I configure a managed mode AP without using the UI?](#managed)
* [Can I configure an alternate port for RaspAP's web service?](#webport)
* [What breaks RaspAP when Docker is installed on the same system and how I can fix it?](#docker)

## OpenVPN
* [OpenVPN fails to start and/or I have no internet. Help!](#openvpn-fails)
* [OpenVPN works but I have partial or no internet access. Help!](#partial)
* [OpenVPN is enabled but I am still blocked from country restricted websites. Help!](#restricted)  

## Networking
* [Why can't I access wireless mode 'N' (802.11n)?](#wireless-mode)
* [How do I exclude NAT rules from IP traffic on localhost?](#iptables)
* [Why is the 802.11ac 5GHz option disabled in Configure hotspot?](#80211ac)
* [I think my country allows 5 GHz AP channels. Can I test this?](#wificountries)
* [Why is the maximum throughput of my 802.11n AP reduced by half?](#wirelessn)

## Install & upgrade
* [How do I upgrade RaspAP?](#upgrade)
* [Do I need the RaspAP service to run at boot?](#raspap-service)
* [Can the Quick Installer accept the default options without prompting me?](#unattended)
* [How do I uninstall RaspAP?](#uninstall)

## <a name="settings"></a>What do all these settings in the UI do? Changing them seems to have no effect.
RaspAP manipulates several daemons, services and helper programs behind the scenes for you. In the footer of each management panel is a helpful "Information provided by..." label. These indicate which Linux daemon and/or program is being modified by the UI. Learning what these services are and how they work will go a long way toward demystifying things.

![](https://i.imgur.com/q1RflMy.png){: style="width:350px"}

For example, two of the best starting points for understanding `hostapd` (the service that implements 802.11 AP management) include the [hostapd Linux documentation page](https://wireless.wiki.kernel.org/en/users/Documentation/hostapd) and [hostapd Wifi homepage](https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf).

> :information_source: **Important:** After you choose **Save settings** for `hostapd` or `dhcpcd`, these services _must_ be restarted for your changes to take effect. If you're not sure if your AP is behaving as expected, enable logging in the **Logging** tab of **Hotspot** and check the output.

## <a name="headless"></a>How do I prepare the SD card to connect to WiFi in headless mode?
Since [May 2016](https://www.raspberrypi.org/blog/another-update-raspbian/), Raspbian has been able to copy wifi details from `/boot/wpa_supplicant.conf` into `/etc/wpa_supplicant/wpa_supplicant.conf` to automatically configure wireless network access. 

An example `wpa_supplicant.conf` file is shown below. Replace the fields with your settings:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=your_ISO-3166_two-letter_country_code

network={
    ssid="my_SSID"
    psk="my_PSK"
    key_mgmt=WPA-PSK
}
```

## <a name="interfaces"></a>Can I use wlan0 and wlan1 rather than eth0 for my AP?
Yes, this is supported by RaspAP. In this scenario, you may wish to use the `wlan0` interface as a wireless client with `wlan1` as the AP interface. Follow the steps below to enable this configuration:

1. Bind the two adapters with the interface names `wlan0` and `wlan1`.
2. Select `wlan1` as the AP interface in **Hotspot** and **Save Settings**.
3. Do not enable the Wifi AP (AP-STA) mode option. This is only needed when the client and AP are utilizing the same wlan adapter.
4. (Re)start the hotspot.

> :information_source: **Important:** Be aware that external WiFi adapters (ie, USB "dongles") vary greatly in terms of hardware capabilities and driver support. Many do not have support for AP mode, require a powered USB hub, manual driver and/or firmware installation or are otherwise not well suited for this application.

Recommended adapters such as the Edimax 7811Un and Ralink RT5370 work out of the box with Raspberry OS (32-bit) Buster Lite. Adapters that require compiling of third-party drivers or other workarounds can be problematic. For this reason, you must verify your adapter _before_ reporting an issue with this feature.

## <a name="monitor"></a>Can I use RaspAP as a monitor only, without changing my configuration?
Yes, RaspAP has support for a so-called "monitor mode". In `config.php` change the setting `RASPI_MONITOR_ENABLED` to `true`. This disables the ability to modify settings, start/stop daemons, shutdown or reboot the RPi. RaspAP will continue to report interface statistics, service settings and data usage as normal. See [this](/defaults/#managing-config-values) for more information. 

## <a name="dnsmasq"></a> Can I use RaspAP with my custom dnsmasq configuration?
Yes, RaspAP supports this through the use of `dnsmasq.d`. The primary `/etc/dnsmasq.d/090_raspap.conf` managed by the UI includes the following directive to enable your custom .conf files:

```
conf-dir=/etc/dnsmasq.d
```

Configuration files placed in this directory will be used by the dnsmasq service and are untouched by the UI. 

## <a name="noip"></a>Clients cannot obtain an IP address from the AP.
Clients may receive a "failed to obtain IP address" or similar error message when connecting to your AP. These are the most frequent reasons for this error:

1. A poor Wi-Fi signal from the access point. In this event, reduce the distance between your device and the AP.
2. Your device does not operate properly with the encryption method set by the AP. 
3. The access point is misconfigured.

The first and simplest fix is to reconnect the client to your Wi-Fi network. When you do this, the AP forgets the previous attempt and initiates a new process to assign an IP address to your device.
Exact methods vary between devices, however most will have a 'Forget this network' option or similar in the Wi-Fi settings. This is shown in iOS, below:
 
![](https://i.imgur.com/7xvx5JT.png){: style="width:350px"}

If clients still fail to connect, restart the AP. You may do this by choosing **Restart hotspot** from RaspAP. This reinitializes several related services in a predictable order and timing.
Assuming these services are configured to restart automatically on reboot (the default behavior when RaspAP's installer is used) you may also simply reboot your Pi.

RaspAP gives you control over many aspects of your Wi-Fi network, including DHCP. With its [default settings](https://docs.raspap.com/issues/#default-settings), RaspAP has been rigorously
tested and validated to provide connectivity in routed AP mode. If you suspect that RaspAP is misconfigured and not providing IP addresses to clients, you may troubleshoot this yourself.

Clients connecting to your AP are assigned, or leased, an IP address with `dnsmasq`. You can see how this proces works by enabling the **Log DHCP requests** option in the **DHCP Server > Logging** tab. 
When a client connects to your AP, a typical `dnsmasq-dhcp` exchange follows this pattern:

```
dnsmasq-dhcp[2516]: DHCPDISCOVER(wlan0) [MAC address] 
dnsmasq-dhcp[2516]: DHCPOFFER(wlan0) 10.3.141.249 [MAC address] 
dnsmasq-dhcp[2516]: DHCPREQUEST(wlan0) 10.3.141.249 [MAC address] 
dnsmasq-dhcp[2516]: DHCPACK(wlan0) 10.3.141.249 [MAC address] iPhone
```

If one or more steps in this exchange are missing, either your device is unable to respond to the server's `DHCPOFFER` or the AP itself is misconfigured.

> :information_source: **Important**: By default, the `dnsmasq` service listens on TCP/UDP port 53 and UDP port 67. If you have configured firewall software such as `ufw` or `iptables` to filter traffic on these ports, the service may not be
able to respond to DHCP requests.

As a last resort, you can assign a static IP address to your device. Copy the MAC address for your device as it appears above and create a new entry in RaspAP's **DHCP Server > Static Leases** tab. 
Save settings, restart `dnsmasq` and try connecting your client again.

## <a name="access"></a>My wifi network disappeared and I can't access the webgui. Help!
If you're running your Pi headless and are unable to access RaspAP's web interface from the default http://10.3.141.1/ address, do the following:

1. Be sure your browser isn't forcing SSL by appending https:// to the address, which can result in misleading errors. This may sound obvious but it's reported frequently. (Related: add [SSL support for RaspAP](https://docs.raspap.com/ssl-quick/))
2. Connect your device to wired ethernet and access it via the browser or SSH on the `eth0` interface using one of the methods described below. Check the logs for hostapd errors and reconfigure the service, or run the installer again to restore the [default configuration](/defaults/).
3. There are [several methods](https://www.raspberrypi.org/documentation/remote-access/ip-address.md) you can use to determine your Pi's IP address. RaspAP's installer only configures a static IP address for the AP interface on `wlan0`. If the AP has entered a failed state, you may still be able to connect on an alternate interface.
4. Recent versions of the RPi OS kernel include the `avahi-daemon` which facilitates local network discovery via multicast DNS (mDNS). On client computers with the Bonjour service installed (all macOS machines and Windows PCs with Apple iTunes), try accessing your Pi by entering [http://raspberrypi.local/](http://raspberrypi.local/) in the browser or via SSH with `ssh pi@raspberrypi.local`.
5. If you don't have access to wired ethernet or the above methods fail, configure your Pi for USB-OTG, aka 'on-the-go' or gadget mode. Instructions for enabling USB-OTG vary between various models and not all Pi hardware has support for this.

## <a name="custom"></a>My custom `hostapd.conf` / `php.ini` is gone. Help!
The [installer](https://docs.raspap.com/quick/) applies a "known good" [default configuration](/defaults/) to some services, including `hostapd`. It will also, optionally, optimize php by changing a very limited number of settings. Your custom configurations haven't been lost however; they've been moved to the backups directory in `/etc/raspap/backups`.

You are free to SSH in to restore those files to their rightful position. However, you may need to ensure that the RaspAP modifications are applied to your own custom configurations.

## <a name="password"></a>I changed the admin password and forgot what it was. Help!
Login credentials are stored in `/etc/raspap/raspap.auth`. The password is encrypted and cannot be edited manually. However, deleting this file with `sudo rm /etc/raspap/raspap.auth` will restore the default admin password.

## <a name="nowifi"></a>RaspAP control panel works but there is no WiFi after reboot.
This problem often occurs when another program tries to reconfigure hostapd at startup. It can also happen when your RPi is configured as both a WiFi client and access point, known as a 'managed mode' AP. To address this, RaspAP has added a `systemd` init service to bring up networking services in a predictable order and timing after the Linux kernel is booted. You can check the status of this service with:

```
sudo systemctl status raspapd.service
```

The `raspapd.service` is optionally installed and enabled by the Quick Installer. It is also included in the manual setup steps.

## <a name="bridged"></a>Bridged AP mode is unstable or clients can't connect. Help!
RaspAP [delegates all DHCP control to your router](https://docs.raspap.com/bridged/) in bridged AP mode. If you have trouble connecting clients, start with this project's [default configuration](https://docs.raspap.com/issues/#default-settings) in routed AP mode _first_ and try connecting a client. Enable logging for DHCP and hostapd to help you identify any problems. If you have no issues with client connectivity with the default routed AP, but cannot connect clients in bridged AP mode, in most cases the problem lies with your router—not RaspAP. Check your router's web interface and DHCP settings.

If clients disconnect intermittently, this often indicates an undervoltage issue with your RPi. Check the kernel log for any `Under-voltage detected!` errors. Be sure you are using an official 5.1V power supply (each model has [different power requirements](https://www.raspberrypi.org/documentation/hardware/raspberrypi/power/README.md)) and detach any USB devices. Executing `dmesg | grep br0` can also offer clues. Execute `sudo dhclient -v` to gain insights into DHCP requests between your device and router. A typical DHCP exchange follows this pattern:

```
CLIENT -> DHCPDISCOVER
SERVER -> DHCPOFFER
CLIENT -> DHCPREQUEST
SERVER -> DHCPACK
```

If your device (the client) broadcasts DHCPDISCOVER, but there is no DHCPOFFER response from your router, you have a misconfiguration or other issue with your network. Troubleshooting client connectivity in bridged AP mode is not supported. No hard feelings.

## <a name="pizero-w"></a>Managed mode AP doesn't work on the Pi Zero W. Help!
See [this walkthrough](https:docs.raspap.com/ap-sta/) where the installation is described in detail.

## <a name="scanning"></a>WiFi scanning doesn't work or I get the error `cannot execute "wpa_cli reconfigure"`. Help!
On some configurations, the **Configure WiFi client** panel may appear empty. This project uses the `wpa_supplicant` command line client `wpa_cli` to populate a list of available wireless networks. If you can't execute this from the shell, neither can the web UI. For example, the results of this command:

```
sudo wpa_cli -i wlan0 scan_results
Failed to connect to non-global ctrl_ifname: wlan0  error: No such file or directory
```
indicate a problem with the socket used to communicate with `wpa_supplicant`. You may also encounter errors such as "Could not connect to wpa_supplicant: wlan0 - re-trying".

If this happens, first check the contents of `wpa_supplicant` with `sudo cat /etc/wpa_supplicant/wpa_supplicant.conf`. You should see, at minimum, the following:

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
```

The above is present on clean installs of Raspbian. If you've made changes to this file, ensure that these lines appear first. Next, reinitialize the socket with:

```
sudo wpa_supplicant -B -Dnl80211,wext -c/etc/wpa_supplicant/wpa_supplicant.conf -iwlan0
```

substituting `wlan0` with your wireless interface, if necessary. You should then be able to perform scans as expected.

> :information_source: **Note:** If you are using `wpa_suplicant.conf` to connect to your device with SSH on a wireless interface, do _not_ reboot after running the Quick Installer. More information 
on this topic is [available here](https://docs.raspap.com/ap-sta/#when-to-reboot).

## <a name="pihole"></a>How do I integrate RaspAP with Pi-hole?
There have been several discussions around integrating RaspAP with Pi-hole, with the end goal of hosting a complete AP and ad-blocker on a single device. Both projects rely on `dnsmasq`, so integration between them is tricky. There are now several options available to users of RaspAP.

One option is to configure RaspAP to use a Pi-Hole installation on a separate device. Go to RaspAP's **DHCP Server** > **Advanced** page and enable the "Upstream DNS Server" option, add your Pi-Hole's DNS, save settings and restart dnsmasq.

Alternatively, you can run Pi-Hole and RaspAP on the same device by operating RaspAP in bridged mode. Go to RaspAP's **Hotspot** > **Advanced settings** page, enable the "Bridged AP mode" option and restart your hotspot. 

Finally, and by popular demand, RaspAP has released its own [ad blocking facility](https://docs.raspap.com/adblock/) with support for custom blocklists. 

## <a name="adguard"></a>Can I integrate RaspAP with Adguard Home?
Yes, you can run RaspAP and [Adguard Home](https://github.com/AdguardTeam/AdGuardHome) on the same device. Change Adguard Home’s listening port to `5300` and bind to `127.0.0.1`, then go to RaspAP's > **DHCP Server** > **Advanced** page and enable the "Upstream DNS Server".  Add `127.0.0.1#5300` as an upstream DNS Server. Save settings and restart dnsmasq. Tip via [@firestrife23](https://github.com/RaspAP/raspap-webgui/issues/542#issuecomment-609078400)

## <a name="captive"></a>Can I configure RaspAP to work with a captive portal?
Yes, the [nodogsplash project](https://github.com/nodogsplash/nodogsplash) works just fine with RaspAP and is recommended over other methods. A detailed setup guide is [available here](https://docs.raspap.com/captive/). 

## <a name="schedule"></a>How do I create an AP activation schedule?
This is a common function in consumer wireless routers. For example, let's assume you want to disable your AP on Monday through Friday between 02:00 and 08:00. You can implement this with cron to stop/start RaspAP's service control script at certain times. Run `sudo crontab -e` and add entries like so:

```
# Stop RaspAP services at 02:00 on Monday through Friday
0 2 * * 1-5 sudo /etc/raspap/hostapd/servicestart.sh --action stop

# Start RaspAP services at 08:00 on Monday through Friday
0 8 * * 1-5 sudo /etc/raspap/hostapd/servicestart.sh --seconds 3
```

For help with crontab, head over to <a href="https://crontab.guru/">crontab.guru</a>.

## <a name="managed"></a> Can I configure a managed mode AP without using the UI?
Let's assume you are creating a Raspbian image (or other supported OS) with scripts that setup RaspAP at first startup. In this scenario, to configure a managed mode AP you must manually connect via a browser, make some changes via the UI and then save your settings. This can be also be done programmatically. Assuming you have [wpa_supplicant.conf fully populated](https://docs.raspap.com/faq/#headless), set the following values in `/etc/raspap/hostapd.ini`:

```
LogEnable = 0
WifiAPEnable = 1
BridgedEnable = 0
WifiManaged = wlan0
```

substituting `wlan0` for your AP interface, if necessary. You may then restart the raspap daemon with `sudo systemctl restart raspapd.service`.

## <a name="webport"></a>Can I configure an alternate port for RaspAP's web service?
Yes, you can now do this from the **Advanced** tab in System. Manual steps for changing lighttpd's default port are included below.

Edit `/etc/lighttpd/lighttpd.conf` and change the following line:

```
server.port                 = 8080
```
then give the service a kick...
```
sudo systemctl restart lighttpd.service
```
You can then access RaspAP as before with the new port number in the URI, for example, http://raspberrypi.local:8080. This will allow you run another web server alongside lighttpd, if that is your goal. 

## <a name="docker"></a>What breaks RaspAP when Docker is installed on the same system and how I can fix it?
Installing RaspAP after installing Docker often results in connected clients not having internet access from the AP. The reason for this is Docker manipulates `iptables` rules to provide network isolation. Docker installs two custom iptables chains named `DOCKER-USER` and `DOCKER`, and it ensures that incoming packets are always checked by these two chains first. Docker also sets the policy for the `FORWARD` chain to `DROP`.

When RaspAP is started in its default router mode, this will result in the AP not forwarding traffic anymore. If you want RaspAP to continue functioning as a router, you can add explicit `ACCEPT` rules to the `DOCKER-USER` chain to allow it:

`sudo iptables -I DOCKER-USER -i src_if -o dst_if -j ACCEPT`

When Docker is correctly installed _after_ RaspAP, the following `iptables` chain should be present:

```
Chain INPUT (policy ACCEPT) target prot opt source destination
Chain FORWARD (policy ACCEPT)
target prot opt source destination DOCKER-USER all -- anywhere anywhere
DOCKER-ISOLATION-STAGE-1 all -- anywhere anywhere
ACCEPT all -- anywhere anywhere ctstate RELATED,ESTABLISHED DOCKER all -- anywhere anywhere
ACCEPT all -- anywhere anywhere ACCEPT all -- anywhere anywhere
Chain OUTPUT (policy ACCEPT) target prot opt source destination
Chain DOCKER (1 references) target prot opt source destination
```

Additional info [here](https://github.com/RaspAP/raspap-webgui/issues/458#issuecomment-643425658) and [here](https://docs.docker.com/network/iptables/). 

**tl;dr:** Install RaspAP _first_, followed by Docker, adding the explicit `iptables` rule `sudo iptables -I DOCKER-USER -i src_if -o dst_if -j ACCEPT`.


## <a name="openvpn-fails"></a> OpenVPN fails to start and/or I have no internet. Help!
RaspAP supports OpenVPN clients by uploading a valid .ovpn file to `/etc/openvpn/client` and, optionally, creating a `login.conf` file with your client auth credentials. Additionally, in line with the project's [default configuration](/defaults/), the following iptables rules are added to forward traffic from OpenVPN's `tun0` interface to your configured wireless interface (`wlan0` is the default):

```
-A FORWARD -i tun0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -i wlan0 -o tun0 -j ACCEPT
``` 

It is your responsibility to provide a valid .ovpn file; RaspAP does not attempt to validate the settings or RSA keys contained in this file. If OpenVPN fails to start, check for errors with `sudo systemctl status openvpn-client@client` and `journalctl --identifier openvpn`.

## <a name="partial"></a>OpenVPN works but I have partial or no internet access. Help!
Issues [like this](https://github.com/RaspAP/raspap-webgui/issues/612) are frequently reported. Begin by confirming the status of your connection:

```
$ sudo systemctl status openvpn-client@client
● openvpn-client@client.service - OpenVPN tunnel for client
   Loaded: loaded (/lib/systemd/system/openvpn-client@.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-06-12 15:45:41 CDT; 1min 39s ago
     Docs: man:openvpn(8)
           https://community.openvpn.net/openvpn/wiki/Openvpn24ManPage
           https://community.openvpn.net/openvpn/wiki/HOWTO
 Main PID: 2689 (openvpn)
   Status: "Initialization Sequence Completed"
    Tasks: 1 (limit: 2200)
   Memory: 1.1M
   CGroup: /system.slice/system-openvpn\x2dclient.slice/openvpn-client@client.service
           └─2689 /usr/sbin/openvpn --suppress-timestamps --nobind --config client.conf
```
You can also use `journalctl --identifier openvpn` to identify any errors. If your internet access is intermittent or otherwise degraded with the `openvpn-client` active, the next step is to test your connection for packet loss and latency. There are many Linux tools you can use to diagnose your network. `mtr` is a good choice as it combines functionality of the traceroute and ping programs. Install and use it to perform your own evaluation:

```
sudo apt install mtr -y
sudo mtr -rwc 50 -i 0.2 -rw duckduckgo.com

Start: 2020-06-13T11:42:26+0100
HOST: raspberrypi                                Loss%   Snt   Last   Avg  Best  Wrst StDev
  1.|-- 192.168.1.254                              0.0%    50   26.8  27.1  26.5  31.4   0.8
  2.|-- somerouter.net                            88.0%    50   392.0 390.4 362.1 596.7  1.2
```

The results are reported as round-trip response times in milliseconds and the percentage of packet loss. If you see loss and/or latency like the above example, report it to your VPN provider or find another one. Read [this](https://www.linode.com/docs/networking/diagnostics/diagnosing-network-issues-with-mtr/) for more on interpreting mtr results.

Protip: free VPNs are frequently oversubscribed and usually not worth the trouble.

## <a name="restricted"></a>OpenVPN is enabled but I am still blocked from country restricted websites. Help!
Remote hosts use a variety of methods to defeat VPNs, some more aggressively than others. Many VPN providers will advise you to configure custom DNS servers to mitigate [DNS leaks](https://dnsleaktest.com/), which you can do from RaspAP's **DHCP > Advanced** tab. Others have specific VPN nodes to use with popular streaming services. 

Several users have reported that Firefox's [DNS-over-HTTPS (DoH)](https://support.mozilla.org/en-US/kb/firefox-dns-over-https) has created problems with their VPN, in effect creating a DNS leak from the browser that circumvents RaspAP's DNS settings. Be sure to disable this "feature" when using a VPN service. 

If you suspect network traffic is not being routed through tun0 (or any other interface) for some reason, you can monitor this directly from your RPi with `iftop`:

```
sudo apt install iftop
sudo iftop -i [interface]
```

## <a name="wireless-mode"></a>Why can't I access wireless mode 'N' (802.11n)?
On the **Configure hotspot** > **Security** tab, be sure to select CCMP for the Encryption Type. Save the settings and restart the hotspot. The wireless mode should be reported on clients as 802.11b/g/n.

```
RaspAP:
  PHY Mode:		802.11n
  Channel:		1
  Network Type:		Infrastructure
  Security:		WPA2 Personal
  Signal / Noise:	-49 dBm / -86 dBm
  Transmit Rate:	73
```

If using TKIP for encryption with WPA, you will be restricted to 54 Mb/s. This is because the IEEE 802.11n draft prohibits using high throughput with WEP or TKIP ciphers.

## <a name="iptables"></a> How do I exclude NAT rules from IP traffic on localhost?
RaspAP's [Quick Installer](https://docs.raspap.com/quick/) configures network-address-translation (NAT) with iptables rules, so that the RPi can act as an internet gateway to multiple hosts on a local network with a single public IP address. This is done by rewriting the addresses of IP packets as they pass through the NAT system. Many access points, including RaspAP, use a combination of IP forwarding and masquerading to achieve this.

In some cases, NAT rules applied to `localhost` can interfere with other services running on an RPi. An example is the Plex Media Server, which has an API that listens on localhost. As of this writing, the Plex API has been built to not authenticate communication between service processes of the server. This can cause a failure to communicate with the Plex API or similar add-on services on your RPi.

The solution is to add a NAT rule ahead of the rule RaspAP installs to not apply NAT to connections destined to 127.0.0.0/8:

```
$ sudo iptables -t nat -I POSTROUTING -d 127.0.0.0/8 -j ACCEPT
```
The resulting iptables chain should look something like this:

```
$ sudo iptables -t nat -L -n -v
Chain PREROUTING (policy ACCEPT 31 packets, 4810 bytes)
 pkts bytes target prot opt in out source destination

Chain INPUT (policy ACCEPT 31 packets, 4810 bytes)
 pkts bytes target prot opt in out source destination

Chain OUTPUT (policy ACCEPT 23 packets, 1338 bytes)
 pkts bytes target prot opt in out source destination

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target prot opt in out source destination
   17 999 ACCEPT all -- * * 0.0.0.0/0 127.0.0.0/8
   2422 158K MASQUERADE all -- * * 0.0.0.0/0 0.0.0.0/0
```
Refer to [this issue](https://github.com/RaspAP/raspap-webgui/issues/333#issue-454352554).

## <a name="80211ac"></a> Why is the 802.11ac 5GHz wireless mode option disabled in Configure hotspot?
**Short answer:** because of wireless regulatory restrictions for your country. If the AC option is disabled on your RPi, there are two small configuration changes you can make to enable it. First, configure your RPi's wireless regulatory domain with `sudo iw reg set US`. Next, on the **Advanced** tab of **Configure hotspot**, choose "United States" as the country code and save settings. You can now choose 802.11ac from the wireless mode select on the **Basic** tab.

Choosing the AC wireless mode will populate the supported 5 GHz channels for you.

![](https://i.imgur.com/ZAxB8Wf.png){: style="width:350px"}


**Longer answer:** AC support is not simply a function of your device's hardware capabilities. It must also take into account regulatory restrictions of the wireless spectrum. The regulatory info for `brcmfmac`, the kernel driver that supports the Broadcom wireless chipset, is embedded in the firmware of RPi models 3B+ and 4. There are lots of [international issues with WiFi](https://en.wikipedia.org/wiki/List_of_WLAN_channels#5_GHz_or_5.8_GHz_(802.11a/h/j/n/ac/ax)) that restrict channel use, transmission power, etc. on a regional and per-country basis. As a result, only combinations of certain frequencies (channels) and countries are capable of hosting an AC access point with the RPi's wireless adapter. 

If the country configured on your RPi does not allow use of a particular segment of the 5 GHz wireless spectrum, an AC configured AP will fail to start. Errors like these are common:

```
nl80211: Failed to set channel (freq=5180): -22 (Invalid argument)
hostapd: Could not set channel for kernel driver
```

In testing, stable AP's on the RPi's supported AC channels were only reliably obtained with 'US' as the regulatory domain. To get a list of the supported channels on your RPi for the 2.4 and 5 GHz bands, use `iw phy phy0 channels`. Refer to [this issue](https://github.com/RaspAP/raspap-webgui/issues/450#issuecomment-569343686).

## <a name="wificountries"></a>I think my country allows 5 GHz AP channels. Can I test this?
Yes, you can. In the spirit of experimentation, this project allows you to override RaspAP's [default configuration](/defaults/). The file [wireless.json](https://github.com/RaspAP/raspap-webgui/blob/master/config/wireless.json) contains the regulatory domains and channels for the 2.4 and 5 GHz bands. Add a valid ISO Alpha-2 country code to the list of `5Ghz_max48ch` countries and save the file. Next, edit `includes/config.php` and add the same country to this constant:

```
// Constant for the 5GHz wireless regulatory domain
define('RASPI_5GHZ_ISO_ALPHA2', array('US'));
``` 

The **Configure hotspot** page will now let you select AC as a wireless mode option for your country. If you succeed in creating a stable AP, feel free to share your results in a [discussion](https://github.com/RaspAP/raspap-webgui/discussions/).

> :information_source: **Note:** it is recommended to monitor logs such as `dmesg` and the hostapd error log (available in the **Logfile output** tab of RaspAP) while doing this. Bug reports like "AC doesn't work" and/or troubleshooting requests will not be considered. No hard feelings.

## <a name="wirelessn"></a>Why is the maximum throughput of my 802.11n AP reduced by half? 
In order to achieve optimal throughput with 802.11n, the wireless stream must operate at a 40 MHz wide channel on the 2.4 GHz band. A 20 MHz channel will restrict you to 72 Mbps. Your `hostapd.conf` might have  the required settings, but this is no guarantee of a 40 MHz channel.

In practice, this can be quite difficult due to interference on the 2.4 GHz band. There are many things that will cause an AP to fallback to 20 MHz. The most common reason is if an AP detects another wireless network within 40 MHz, i.e. two channels, of its own channel. For example, if an AP is set to channel 6, another network operating anywhere from channel 4 to 8 will trigger a fallback. hostapd will usually report a fallback like so:

```
20/40 MHz operation not permitted on channel pri=3 sec=7 based on overlapping BSSes
```

For more information on optimizing 802.11n, refer to this [resource](https://www.lmi.net/wp-content/uploads/Optimizing_802.11n.pdf).

Generally speaking, the 5 GHz band has substantially greater capacity due to more non-overlapping radio channels and less radio interference as compared to the 2.4 GHz band. 

## <a name="upgrade"></a>How do I upgrade RaspAP?
Upgrading an existing install without changing your configuration is very straightforward. To upgrade to the [latest release](https://github.com/RaspAP/raspap-webgui/releases/latest) version, simply run the [Quick Installer](https://docs.raspap.com/quick/) with the `--upgrade` option:

```
curl -sL https://install.raspap.com | bash -s -- --upgrade
```

The installer upgrade is _idempotent_, meaning it can be repeated an arbitrary number of times and the result will be as if it had been done only once. If you choose this method, you're done! Confirm the upgrade by checking the release version on the **About** page.

If you want to install a specific version you can do so by referencing a tag:

```
sudo git fetch --tags
sudo git checkout 2.5
```

A tag is a pointer that isn't connected to the main development tree that git knows about. As a result, git will reply that you're in a 'detached HEAD' state. This isn't a big deal, it just means that you have a specific version of the code that isn't connected to the git tree.

Alternatively, if you want the latest _bleeding edge_ commits from the master branch, use the following:

```
sudo git checkout -b master
sudo git pull origin master
```

If you've customized your installation by editing `config.php`, update the release version in this file:
```
sudo nano /var/www/html/includes/config.php
```
Change the first line to the release version, save the file and exit.

> :information_source: **Note:** `RASPI_VERSION` is only used on the About page; it does not affect any other functionality.
```
define('RASPI_VERSION', '2.5');
```

Finally, give the lighttpd service a kick with:
```
sudo systemctl restart lighttpd.service
```

Whichever method you choose (installer upgrade, specific release or latest updates), your RaspAP configuration won't be changed.

## <a name="raspap-service"></a>Do I need the RaspAP service to run at boot?
If you are using your RPi as a client on a WiFi network (also known as managed mode) and hosting an access point simultaneously, the `raspapd.service` will ensure that your hotspot is active after a reboot. It does this by detecting WiFi client AP mode, adding the `uap0` interface and starting up networking services in a specific order.

If your RPi is configured with wired ethernet (`eth0`) or you haven't experienced problems with the AP starting on boot, you can disable the RaspAP daemon like so:

```
sudo systemctl disable raspapd.service
```

## <a name="unattended"></a> Can the Quick Installer accept the default options without prompting me?
Yes, the [Quick Installer](https://docs.raspap.com/quick/) has a non-interactive mode that lets you perform unattended setups. This mode assumes "yes" as an answer to all prompts. You can do an unattended install of RaspAP by appending the `--yes` command-line option, like so:

```
curl -sL https://install.raspap.com | bash -s -- --yes
```

The options `-y` or `--assume-yes` are also accepted and have the same result. 

## <a name="uninstall"></a>How do I uninstall RaspAP?
We have provided an uninstall script to remove RaspAP cleanly, and also restore any backups of your configuration that were created before RaspAP was installed. The uninstall script is located in `installers/uninstall.sh`. To start the uninstaller, simply run the following from the project root folder (default location is `/var/www/html`):

```
cd /var/www/html
sudo installers/uninstall.sh
```

