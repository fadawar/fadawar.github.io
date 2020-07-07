.. title: How to split traffic on GlobalProtect VPN on Mac OS
.. slug: how-to-split-traffic-on-globalprotect-vpn-on-mac-os
.. date: 2018-02-13 12:57:59 UTC+02:00
.. tags: vpn, privacy, mac-os
.. category: 
.. link: 
.. description: 
.. type: text

Recently the company I work for started to use Palo Alto's GlobalProtect as solution for VPN.
The solution works quite well but have 2 flaws by default that I don't like.

<!-- TEASER_END -->

First is that GlobalProtect agent (client) runs automatically after operating system turns on
and this behaviour can't be changed in the settings. You can find solution for it on [other blogs](http://richddean.com/post/147155656349/stopautostartglobalprotectvpn).

Second flaw is that it automatically send *ALL* of my traffic through my company's VPN.
I don't think this is beneficial for company but most importantly it goes against my privacy.
There is no need for employer to know what goes on in my traffic.

This article describes:

* How to split traffic based on IP addresses
* How to do traffic splitting automatically after GlobalProtect agent connects to VPN

I will only focus on Mac OS but similar steps can be taken also on other operating systems.

Traffic split with GlobalProtect
--------------------------------
When you connect to VPN with GlobalProtect, it creates new network interface
and edits the routing table so all our traffic is sent through this new network interface.

To solve this we need to remove a route created by GlobalProtect and then create
few new routes for only those IP addresses which we want to be directed through our VPN.

We implemented it in Python (based on this [blog post](https://www.shadabahmed.com/blog/2013/08/11/split-tunneling-vpn-routing-table)).
Save the script as split_vpn.py to your home folder.
Edit the lists VPN_NETS and VPN_HOSTS based on your needs. Then you can run it everytime
you want to split traffic.

```python
#!/usr/bin/env python
import os
import subprocess
import sys

WIRELESS_INTERFACE = 'en0'	# could be different on other systems
TUNNEL_INTERFACE = 'gpd0'

VPN_NETS = [
    '172.222',				# subnets which should use VPN
]

VPN_HOSTS = [
    '90.131.25.244',
    '90.131.25.240',        # IP address which should use VPN
]


def main():
    if os.getuid() != 0:
        sys.exit('Please, run this command with sudo.')

    gateway = None
    out = subprocess.check_output(('netstat', '-nrf', 'inet'))
    routes = out.decode('utf-8').split('\n')[3:]

    for route in routes:
        route = route.split()
        interface = route[3]
        if interface == WIRELESS_INTERFACE:
            gateway = route[1]
            break

    if gateway is None:
        sys.exit('Unable to determine VPN default gateway.')

    print('Resetting routes with gateway ' + gateway)
    subprocess.call(('route', '-n', 'delete', 'default', '-ifscope', WIRELESS_INTERFACE))
    subprocess.call(('route', '-n', 'delete', '-net', 'default', '-iface', TUNNEL_INTERFACE))
    subprocess.call(('route', '-n', 'add', '-net', 'default', gateway))

    print('\nAdding routes for addresses which should go through VPN.')
    for addr in VPN_NETS:
        subprocess.call(('route', '-n', 'add', '-net', addr, '-iface', TUNNEL_INTERFACE))
    for addr in VPN_HOSTS:
        subprocess.call(('route', '-n', 'add', '-host', addr, '-iface', TUNNEL_INTERFACE))


if __name__ == '__main__':
    main()
```


Automatic traffic split after connecting to VPN
-----------------------------------------------
Now when we have the script to split our traffic, we want it to run automatically
after we connect to VPN with GlobalProtect.
As it is stated in [documentation](https://www.paloaltonetworks.com/documentation/80/globalprotect/globalprotect-admin-guide/globalprotect-clients/deploy-agent-settings-transparently/deploy-agent-settings-to-mac-clients/deploy-scripts-using-the-mac-plist),
GlobalProtect agent can run commands
before connecting, after connecting and before disconnecting.

Follow these steps to run the script after GlobalProtect agent connects to VPN:

1. Disable and close GlobalProtect
2. Run `killall cfprefsd`
3. Open in editor `/Library/Preferences/com.paloaltonetworks.GlobalProtect.settings.plist`
4. Add to the section `/Palo Alto Networks/GlobalProtect/Settings/` following *(edit path based on your username)*:

```xml
<key>post-vpn-connect</key>
<dict>
        <key>command</key>
        <string>/Users/your_usename/post_vpn_connect.sh</string>
        <key>context</key>
        <string>admin</string>
</dict>
```

5. Add this script to your home folder and save it as `post_vpn_connect.sh`

```bash
#!/bin/bash
osascript -e 'display notification "Start" with title "VPN traffic split"'

DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

${DIR}/split_vpn.py

country=`curl ifconfig.co/country`

osascript -e "display notification \"End. You country is $country\" with title \"VPN traffic split\""
```


Now your traffic should be automatically split each time you connect to VPN with GlobalProtect. Nice!
