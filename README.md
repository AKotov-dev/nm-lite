# nm-lite
The switch `network/net_applet` to `NetworkManager/nm-applet` (after installation) and back (after removal) + disable unnecessary services to speed up system loading.

Note: To display the nm-applet icon in KDE and GNOME, you need to comment out a line in the file /etc/xdg/autostart/nm-applet.desktop: `#NotShowIn=KDE;GNOME;`


Переключатель `network/net_applet` на `NetworkManager/nm-applet` (после установки) и обратно (после удаления) + отключение ненужных служб для ускорения загрузки системы.

Примечание: Для отображения значка nm-applet в KDE и GNOME нужно закомментировать строку в файле /etc/xdg/autostart/nm-applet.desktop: `#NotShowIn=KDE;GNOME;`

Сomparison of system loading speed:
--
network/net_applet
```
> systemd-analyze
Startup finished in 8.257s (firmware) + 4.259s (loader) + 4.122s (kernel) + 33.652s (userspace) = 50.292s 
graphical.target reached after 33.643s in userspace
```
nm-lite
```
> systemd-analyze
Startup finished in 8.262s (firmware) + 3.134s (loader) + 4.577s (kernel) + 33.524s (userspace) = 49.498s 
graphical.target reached after 33.512s in userspace
```
%post
--
```
#!/bin/bash

#Disable network.service & net_applet, Enable NetworkManager lite
killall -KILL net_applet nm-applet
systemctl stop network.service
systemctl disable network.service

#Disable net_applet.desktop, Enable all nm-applet's
rename -v \.desktop.bak \.desktop $(find /etc/xdg/autostart/* -name '*nm-applet.desktop.bak')
mv -f /etc/xdg/autostart/net_applet.desktop /etc/xdg/autostart/net_applet.desktop.bak

#Enable NetworkManager
systemctl enable NetworkManager.service
systemctl start NetworkManager.service
#systemctl enable NetworkManager-wait-online.service
nm-applet &

#Speeding up system loading
systemctl disable ModemManager.service
systemctl disable lvm2-monitor.service
#systemctl disable avahi-daemon.service

#Disable program RAID & monitoring
systemctl disable mdadm.service
systemctl disable mdmonitor.service
systemctl disable mdmonitor-takeover.service

exit 0;
```

%postun
--
```
#!/bin/bash

#Disable NetworkManager, Enable network & net_applet
killall -KILL nm-applet net_applet
systemctl stop NetworkManager.service
systemctl disable NetworkManager.service
#systemctl disable NetworkManager-wait-online.service

#Disable all nm-applet's, Enable net_applet.desktop (rename)
rename -v \.desktop \.desktop.bak $(find /etc/xdg/autostart/* -name '*nm-applet.desktop')
mv -f /etc/xdg/autostart/net_applet.desktop.bak /etc/xdg/autostart/net_applet.desktop

#Return to network & net_applet
systemctl enable network.service
systemctl start network.service
net_applet &

#Re-disabling unnecessary services
systemctl disable ModemManager.service
systemctl disable lvm2-monitor.service
#systemctl disable avahi-daemon.service

#Disable program RAID & monitoring
systemctl disable mdadm.service
systemctl disable mdmonitor.service
systemctl disable mdmonitor-takeover.service

exit 0;
```
