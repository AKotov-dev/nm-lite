# nm-lite
The switch `network/net_applet` to `NetworkManager/nm-applet` (after installation) and back (after removal) + disable unnecessary services to speed up system loading.

Note: To apply the changes, logout or reboot.


Переключатель `network/net_applet` на `NetworkManager/nm-applet` (после установки) и обратно (после удаления) + отключение ненужных служб для ускорения загрузки системы.

Примечание: Для применения изменений сделайте логаут или перезагрузитесь.

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
systemctl disable network-up
systemctl mask network.service network-up.service

#Disable net_applet.desktop, Enable all nm-applet's
rename -v \.desktop.bak \.desktop $(find /etc/xdg/autostart/* -name '*nm-applet.desktop.bak')
mv -f /etc/xdg/autostart/net_applet.desktop /etc/xdg/autostart/net_applet.desktop.bak

#Show nm-applet in all DE
sed -i 's/^NotShowIn*/#NotShowIn/' /etc/xdg/autostart/nm-applet.desktop

#Enable NetworkManager
systemctl enable NetworkManager.service
systemctl start NetworkManager.service
#systemctl enable NetworkManager-wait-online.service
#nm-applet & Logout required

#Speeding up system loading
systemctl disable lvm2-monitor.service
systemctl disable avahi-daemon.service
systemctl disable ModemManager.service
systemctl disable NetworkManager-wait-online.service

#Disable program RAID & monitoring
systemctl disable mdadm.service
systemctl disable mdmonitor.service
systemctl disable mdmonitor-takeover.service
```

%postun
--
```
#!/bin/bash

#Если удаление
if [ $1 -eq 0 ]; then
#Disable NetworkManager, Enable network & net_applet
killall -KILL nm-applet net_applet
systemctl stop NetworkManager.service
systemctl disable NetworkManager.service
systemctl disable NetworkManager-wait-online.service

#Disable all nm-applet's, Enable net_applet.desktop (rename)
rename -v \.desktop \.desktop.bak $(find /etc/xdg/autostart/* -name '*nm-applet.desktop')
mv -f /etc/xdg/autostart/net_applet.desktop.bak /etc/xdg/autostart/net_applet.desktop

#Return to network & net_applet
systemctl mask network.service network-up.service
systemctl enable network-up
systemctl enable network.service
systemctl start network.service
#net_applet & Logout required

#Re-disabling unnecessary services
systemctl disable ModemManager.service
systemctl disable lvm2-monitor.service
#systemctl disable avahi-daemon.service

#Disable program RAID & monitoring
systemctl disable mdadm.service
systemctl disable mdmonitor.service
systemctl disable mdmonitor-takeover.service
fi;
```
