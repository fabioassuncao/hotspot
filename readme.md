# hotspot

shell script for setup and management of hotspot (hostapd) functions on rpi platform

functions:

- try
- start
- stop
- restart
- retry
- status
- setup
- setchan [channel]
- syslog [lines]
- tor [start|stop]
- version
- wlan [start|stop]
- modpar \<dnsmasq|hostapd\> \<name\> [value]

will use on board wlan adaptor for hotspot functionality and\
the on board ethernet port or an optional external usb wlan adaptor (e.g. EW-7811Un Realtek RTL8188CUS)\
for internet access.

best wlan channel for hotspot functionality will be determined automatically by least used frequency spectrum.

actions will be logged to /tmp/hotspot and syslog\
pls. see examples in troubleshooting section.

for full installation and setup sequence, pls. see **installation and setup** section at the bottom of this file

## installation

rpi login as root required

~~~bash
root:# cd /usr/local/sbin
root:# wget https://raw.githubusercontent.com/rudiratlos/hotspot/master/hotspot
root:# chmod +x hotspot
root:# apt-get update
root:# apt-get upgrade
~~~

## setup

will install all required packages (e.g. iw tor hostapd dnsmasq),\
setting parameters and create config files:

- /etc/sysctl.conf (activate line net.ipv4.ip_forward=1) 
- /etc/rc.local
- /etc/dhcpcd.conf
- /etc/dnsmasq.conf
- /etc/default/hostapd
- /etc/hostapd/hostapd.conf
- /etc/tor/torrc

Existing files will be backed up with a date extension (YYYYMMDDhhmmss). 

~~~bash
hotspot setup
hotspot try
~~~

above command sequence will create a hotspot with following default parameter:

ssid:    \<HOSTNAME\>wlan-\<MAC3ByteAdr\> (e.g. RPIwlan-abcdef)\
pwd:     hallohallo\
country: DE


next commands will create all config files and adjusts parameter to your environment.

~~~bash
hotspot setup
hotspot modpar hostapd ssid myHotspotID 
hotspot modpar hostapd wpa_passphrase myHotspotPassword
hotspot modpar hostapd country SE

hotspot try
~~~

## start

start all hotspot associated functions:

- terminate connection on wlan0
- create device ap0 and assign IP addr
- start dnsmasq
- start hostapd 

~~~bash
hotspot start
~~~

## try

will start hotspot if following condition is met:

- wlan0 or eth0 not connected
- wlan0 and eth0 IP addresses are on same IP subnet (wlan0 connection will be stopped)

~~~bash
hotspot try
~~~

## stop [nowlan]

stop hotspot functions:

- stop hostapd
- stop dnsmasq
- optional: restart wlan

~~~bash
hotspot stop
~~~

## restart

executes following hotspot sequence:

- hotspot stop nowlan
- sleep 20 seconds (settling time)
- hotspot start

~~~bash
hotspot restart
~~~

## retry

executes following hotspot sequence:

- hotspot stop nowlan
- sleep 20 seconds (settling time)
- hotspot try

~~~bash
hotspot retry
~~~

## modpar

change parameter value in config file

format:
hotspot modpar \<dnsmasq|hostapd\> \<name\> [value]

~~~
file selector:
dnsmasq 		/etc/dnsmasq.conf
hostapd 		/etc/hostapd/hostapd.conf

name			parameter name
value			parameter value
~~~

examples:
~~~bash
hotspot modpar hostapd ssid myHotspotID     # set parameter ssid=myHotspotID
hotspot modpar hostapd country_code DE      # set parameter country_code=DE
~~~

### special hostapd parameter

#### autostart

During boot process /etc/rc.local will look for file content ***#autostart=1*** and will execute **hotspot try** command.

~~~bash
hotspot modpar hostapd autostart 1          # enable  autostart
hotspot modpar hostapd autostart 0          # disable autostart
~~~

#### torstart

start tor service automatically

~~~bash
hotspot modpar hostapd torstart 1           # enable  torstart
hotspot modpar hostapd torstart 0           # disable torstart
~~~

#### useiptables

hotspot script will look for file content ***#useiptables=1*** or ***#useiptables=0*** and will execute **iptables** commands for activation and deactivation.

~~~bash
hotspot modpar hostapd useiptables 1        # executing iptable commands
hotspot modpar hostapd useiptables 0        # no iptable commands
~~~

## syslog [lines]

show hotspot related syslog entries

~~~bash
hotspot syslog
hotspot syslog 5
~~~

## tor

start or stop tor service **experimental**\
pls. see ***torstart*** parameter for automatic starting tor service.

~~~bash
hotspot tor start                           # start tor service
hotspot tor stop                            # stop  tor service
~~~

## version

show hotspot script version

~~~bash
hotspot version
~~~

## installation and setup

rpi login as root required

~~~bash
root:# cd /usr/local/sbin
root:# wget https://raw.githubusercontent.com/rudiratlos/hotspot/master/hotspot
root:# chmod +x hotspot
root:# apt-get update
root:# apt-get upgrade                      # optional

root:# hotspot setup

root:# hotspot modpar hostapd ssid myHotspotID 
root:# hotspot modpar hostapd wpa_passphrase myHotspotPassword
root:# hotspot modpar hostapd country SE
root:# hotspot modpar hostapd autostart 1   # optional autostart enable
root:# hotspot modpar hostapd useiptables 1 # optional

root:# reboot                               # if autostart enable or use hotspot try
~~~

## troubleshooting

log entries will be sent to the file /tmp/hotspot.log and syslog utility

following commands will show you hotspot script activity

~~~bash
hotspot syslog
cat /tmp/hotspot.log
tail -500 /var/log/syslog | grep -a "hotspot:"
cat /var/log/syslog | grep -a "hotspot:"
~~~

these commands will show 5 log entries of involved SW packages caused by hotspot command sequence

~~~bash
hotspot syslog 5
tail -500 /var/log/syslog | grep -a -A 5 "hotspot:"
cat /var/log/syslog | grep -a -A 5 "hotspot:"
~~~
