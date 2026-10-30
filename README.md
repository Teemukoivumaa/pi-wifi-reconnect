# Raspberry Pi Wi-Fi reconnect service

A small systemd timer checks the Wi-Fi connection every minute. If NetworkManager
reports the configured Wi-Fi device as disconnected or unavailable, the script
asks it to reconnect using the saved connection profile. The check starts two
minutes after boot.

This can help a Raspberry Pi recover after a router restart, but it cannot prevent
the network outage while the router is restarting. 

It requires Raspberry Pi OS or another system using systemd, NetworkManager, and `nmcli`.

## Configure and install

The script defaults to Wi-Fi device `wlan0` and NetworkManager profile `Network`.

Edit `pi-wifi-reconnect` if your device or saved profile has a different name.

Find the profile name with:

```sh
nmcli connection show
```

From this directory, install and enable the timer:

```sh
sudo install -o root -g root -m 755 pi-wifi-reconnect /usr/local/sbin/pi-wifi-reconnect
sudo install -o root -g root -m 644 pi-wifi-reconnect.service /etc/systemd/system/
sudo install -o root -g root -m 644 pi-wifi-reconnect.timer /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now pi-wifi-reconnect.timer
```

The saved NetworkManager profile must have autoconnect enabled and credentials
available to system services. Confirm the timer is enabled and see its next run:

```sh
systemctl status pi-wifi-reconnect.timer
systemctl list-timers pi-wifi-reconnect.timer
```

## Check logs

Each timer run starts the oneshot service. Inspect the latest result or follow
future runs with:

```sh
systemctl status pi-wifi-reconnect.service
journalctl -u pi-wifi-reconnect.service
```

To run a check immediately:

```sh
sudo systemctl start pi-wifi-reconnect.service
```

## Uninstall

```sh
sudo systemctl disable --now pi-wifi-reconnect.timer
sudo rm /etc/systemd/system/pi-wifi-reconnect.timer
sudo rm /etc/systemd/system/pi-wifi-reconnect.service
sudo rm /usr/local/sbin/pi-wifi-reconnect
sudo systemctl daemon-reload
```
