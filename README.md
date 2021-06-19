# nm-lite
The switch `network/net_applet` to `NetworkManager/nm-applet` (after installation) and back (during removal) + disable unnecessary services to speed up system loading.

Переключатель `network/net_applet` на `NetworkManager/nm-апплет` (после установки) и обратно (после удаления) + отключение ненужных служб для ускорения загрузки системы.

%post
--
```#!/bin/bash

#Disable network.service & net_applet, Enable NetworkManager light
killall -KILL net_applet nm-applet
systemctl stop network.service
systemctl disable network.service

#Diasable net_applet.desktop, Enable all nm-applet's
rename -v \.desktop.bak \.desktop $(find /etc/xdg/autostart/* -name '*nm-applet.desktop.bak')
mv -f /etc/xdg/autostart/net_applet.desktop /etc/xdg/autostart/net_applet.desktop.bak

#Enable NetworkManager
systemctl enable NetworkManager.service
systemctl start NetworkManager.service
nm-applet &

#Speeding up system loading
systemctl disable ModemManager.service
systemctl disable NetworkManager-wait-online.service
systemctl disable lvm2-monitor
systemctl disable avahi-daemon.service

exit 0;
```

%postun
--
```#!/bin/bash

#Disable NetworkManager, Enable network & net_applet
killall -KILL nm-applet net_applet
systemctl stop NetworkManager.service
systemctl disable NetworkManager.service

#Diasable all nm-applet's, Enable net_applet.desktop (rename)
rename -v \.desktop \.desktop.bak $(find /etc/xdg/autostart/* -name '*nm-applet.desktop')
mv -f /etc/xdg/autostart/net_applet.desktop.bak /etc/xdg/autostart/net_applet.desktop

#Return to network & net_applet
systemctl enable network.service
systemctl start network.service
net_applet &

#Re-disabling unnecessary content
systemctl disable ModemManager.service
systemctl disable NetworkManager-wait-online.service
systemctl disable lvm2-monitor
systemctl disable avahi-daemon.service

exit 0;
```
