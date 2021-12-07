## Settings for Raspberry Pi to act as an Access Point

To allow the Raspberry Pi to host a http server that the user's phone can access, the Pi
needs to somehow let the phone connect to it. This is done by using already made Linux 
applications called *hostapd* and *dnsmasq*.

What *hostapd* does is change the normal Wifi interface for Linux where it connects to
an access point and insteads reverts it to where Linux acts as the access point for other
devices to connect to.

Then *dnsmasq* is a lightweight DNS forwarder that is designed to provide DNS services to
small networks like what we are doing. This will also handle the DHCP service to assign
IP addresses to devices that connect to the Pi.

The point of this document is to provide the guideline of how to configure the Pi's 
settings to allow for this to work.

### Installing Required Software
1. Update the Raspberry Pi software.
```
sudo apt-get update
sudo apt-get upgrade
```
2. Install *hostapd* and *dnsmasq* with this command.
```
sudo apt-get install dnsmasq hostapd
```
3. After installing the software, they haven't been configured yet, so turn them off
```
sudo systemctl stop dnsmasq
sudo systemctl stop hostapd
```

### Configuring the Static IP for Pi
Since the Pi is acting as a standalone network for the purpose of acting like a server,
the Pi needs to have a static IP address assigned to it on the wlan0 interface. This will set 
the IP to a local network IP address like *192.168.x.x*.

1. To configure the static IP address, edit the dhcpcd configuration file.
```
sudo nano /etc/dhcpcd.conf
```
2. Go to the end of the file and add these lines to it. Note that a tab is used 
and not spaces.
```
interface wlan0
	static ip_address=192.168.4.1/24
```
3. Now restart the dhcpcd daemon.
```
sudo service dhcpcd restart
```

### Configuring the DHCP server
The DHCP service is provided by the *dnsmasq* software. The original configuration file comes
with a lot of unneeded information, so starting from scratch is the easiest way.

1. Rename the original configuration file and create a new one.
```
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.orig
sudo nano /etc/dnsmasq.conf
```
2. Copy the information below into the new configuration file created in step 1 and save it.
```
interface=wlan0
  dhcp-range=192.168.4.2,192.168.4.20,255.255.255.0,24h
```

So from step 2, we are providing IP addresses 192.168.4.2 to 192.168.4.20 with a lease time of
24 hours.

### Configuring the Access Point Software
The access point configuration is handled by the *hostapd* software. The configuration file
controls various parameters that will be used to configure the wireless network that we are
creating. This file should be empty if the software was just installed.

1. Open the configuration file for editing.
```
sudo nano /etc/hostapd/hostapd.conf
```
2. Add information below to the configuration file.
```
interface=wlan0
driver=nl80211
ssid=SpeakEz
hw_mode=g
channel=7
wmm_enabled=0
macaddr_acl=0
auth_algs=1
ignore_broadcast_ssid=0
wpa=2
wpa_passphrase={ChooseOne}
wpa_key_mgmt=WPA-PSK
wpa_pairwise=TKIP
rsn_pairwise=CCMP
```

What the above does is says that we are using channel 7 for the network, and we 
name the network SpeakEz. Then you guys can choose the password you want for the 
network. Note that the ssid and password don't need to have quotes around them. Also,
the password needs to be >8 characters in length.

3. Now tell the system where to find this file.
```
sudo nano /etc/default/hostapd
```
4. Input the information below in the file that was just opened. There should be a 
commented out line with the name shown below.
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

### Now start the services back up.
```
sudo systemctl hostapd start
sudo systemctl dnsmasq start
```
You might get an error that the hostapd.service or dnsmasq.service is masked. Use the commands below to fix this issue.
```
sudo systemctl unmask hostapd
sudo systemctl enable hostapd
sudo systemctl start hostapd
```
