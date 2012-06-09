---
title: Can't access network resources over VPN connection on Mac OS X?
layout: post
published: true
categories: mac-osx rdp vpn networking
iwpost: https://www.interworks.com/blogs/jpoehls/2012/04/18/cant-access-network-resources-over-vpn-connection-mac-os-x
---

So you have your shiny OS X connected to a VPN, good deal! The problem is, you can't connect to any of the servers and workstations on the VPN. What could be wrong?

It could be that OS X is still trying to find those machines on the internet instead of looking for them on the VPN connection. We can tell OS X to check the VPN connection first by giving it a higher priority than the other network connections on your Mac.

To change the priority of your VPN connection:

1. Choose Apple menu > System Preferences and click Network.
2. Choose Set Service Order from the Action pop-up menu (looks like a gear).
3. Drag your VPN connection to the top of the list.
4. Click OK, and then click Apply to make the new settings active.

This solution saved my day when I couldn't Remote Desktop into my workstation over the VPN.

For more information, refer to [Apple's documentation for this](http://docs.info.apple.com/article.html?path=Mac/10.6/en/8853.html).