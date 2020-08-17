# Enable Wifi ADB
1. Connect device to computer with cable.
2. Ensure your device and computer are in save network, like connect to same wifi.
3. Find out your device ip address, such as 172.10.10.5
4. In terminal, command: adb tcpip 5555
5. In terminal, command: adb connect ip:5555, such as 172.10.10.5: 5555
